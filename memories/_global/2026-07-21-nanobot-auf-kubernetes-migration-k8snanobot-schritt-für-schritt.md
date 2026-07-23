---
created: '2026-07-21T09:13:13.873+00:00'
extra:
  entities:
  - Migration Schritt
  - Erste Migrationsrunde
  - Docker Swarm
  - 'Sealed Secrets

    Controller'
  - Der Versiegelungs
  - Vier Fixes
  - "Auf\n   Pod"
  - Kubernetes
  - Schritt
  - Migrationsrunde
  - Docker
  - Swarm
  - Cutover
  - Ready
  - Anwendungsreihenfolge
  - Namespace
  - Cluster
  - Proxy
  - Tailscale
  - Network
id: f81e6a55add04682b8521fd462770ff6
normalized_hash: fefe42f516cbe927
tags:
- k8s
- nanobot
- kubernetes-migration
- swarm
title: 'nanobot auf Kubernetes: Migration k8s/nanobot Schritt für Schritt'
type: note
updated: '2026-07-21T09:13:13.873+00:00'
verification_state: unverified
---

# nanobot auf Kubernetes — Migration Schritt für Schritt (k8s/nanobot/)

Erste Migrationsrunde von nanobot von Docker Swarm nach Kubernetes, **parallel** zum
laufenden Swarm-Produktivstack (kein Cutover). Cluster: `kind` 3-Node-Cluster `learn`,
Kontext `kind-learn`, Host gx10-b3ed (ARM64/aarch64). Namespace: `nanobot`.
Ergebnis: Deployment 2/2 Ready, erreichbar unter `https://nanobot-k8s.tail5a2ccd.ts.net`
und per `kubectl -n nanobot port-forward service/nanobot 8765:8765`. Committed als `3883c54`
im nanobot-Repo (`k8s/nanobot/`).

## Dateien in Anwendungsreihenfolge

**00-namespace.yaml** — legt den Namespace `nanobot` an, isoliert von anderen
Cluster-Inhalten (z. B. `learn-basics`).

**01-configmap-tailscale-serve.yaml** — entspricht `webui/serve.json` aus dem
Swarm-Stack, aber Proxy-Ziel auf `http://localhost:8765` umgebogen. Grund: nanobot
und der Tailscale-Sidecar laufen im selben Pod (Sidecar-Pattern) und teilen sich
den Network-Namespace — kein Service-DNS-Name nötig wie im Swarm-Overlay-Netz.

**02-pvc.yaml** — zwei PersistentVolumeClaims (beide ReadWriteOnce):
- `nanobot-data` (2Gi): ersetzt den Swarm-Bind-Mount `/var/lib/nanobot` (config.json,
  Sessions, Memory). PVC statt ConfigMap, weil zur Laufzeit beschreibbar.
- `tailscale-state` (100Mi): Tailnet-Login-Zustand, damit der Sidecar nicht bei jedem
  Neustart neu authentifizieren muss.
Die `local-path`-StorageClass in `kind` bindet ein Volume beim ersten Consumer an einen
bestimmten Node (WaitForFirstConsumer) — übernimmt implizit die Rolle von Swarms
expliziter `node.role == manager`-Placement-Constraint.

**03-secrets.md** — keine Manifest-Datei, sondern eine Anleitung (Sealed Secrets
Controller + kubeseal-CLI installieren, dann das echte Secret erzeugen und sofort
versiegeln). Der Versiegelungs-Schritt mit echten Werten wurde bewusst vom User selbst
ausgeführt, nicht von Claude — echte Secret-Werte sollen nicht durch die Konversation
laufen. Ergebnis: `02-secrets.sealed.yaml`, enthält nur `encryptedData`, git-sicher.
Stolperstein: kubeseal-Release-Binary war zunächst amd64 verlinkt, GX10 ist aber
aarch64 — arm64-Binary nötig.

**04-deployment.yaml** — der Kern: ein Pod mit zwei Containern (`nanobot` +
`tailscale`-Sidecar). Vier Fixes waren nötig, um von 0/2 auf 2/2 Ready zu kommen:

1. **`args` statt `command`** beim nanobot-Container. Dockerfile setzt
   `ENTRYPOINT ["entrypoint.sh"]` + `CMD ["gateway"]`. In K8s ersetzt `command:` das
   ENTRYPOINT komplett (anders als docker-compose's `command:`, das nur CMD überschreibt) —
   dadurch fehlte entrypoint.sh (macht die Secret→Env-Bridge) und der Container versuchte
   "gateway" selbst als Programm auszuführen → "executable file not found". `args:` löst das.

2. **`securityContext.runAsUser: 1000` auf Container-Ebene statt Pod-Ebene.** Auf
   Pod-Ebene galt es für beide Container; der tailscale-Sidecar braucht aber Root
   (chmod auf /var/lib/tailscale, Unix-Socket in /var/run) — mit UID 1000 kam
   "operation not permitted"/"permission denied". Fix: runAsUser nur noch im
   nanobot-Container, fsGroup: 1000 bleibt pod-weit (macht beide PVCs für Gruppe 1000
   beschreibbar, unabhängig vom Container-User).

3. **`TS_KUBE_SECRET: ""`** beim Sidecar. containerboot (Tailscale-Image-Entrypoint)
   erkennt automatisch, dass es in einem Pod läuft, und will den Auth-Zustand
   standardmäßig in einem Kubernetes-Secret statt im Dateisystem speichern — fehlende
   RBAC-Rechte dafür. Leerer TS_KUBE_SECRET erzwingt das datei-basierte
   TS_STATE_DIR-Verhalten (wie im Swarm-Stack, PVC tailscale-state).

4. **`gateway.host: 0.0.0.0`** — nicht im Deployment, sondern in der config.json auf
   der nanobot-data-PVC. Der Health-Server bindet standardmäßig auf 127.0.0.1 (passt
   für Docker-Healthchecks im selben Netzwerk-Namespace), aber kubelets Probes
   verbinden sich von außen über die Pod-IP → "connection refused" ohne diesen Fix.
   Pod blieb sonst dauerhaft 1/2 Ready.

Ansonsten 1:1 aus dem Swarm-Compose übertragen: 7 Secret-Keys als Env-Vars
(openai/anthropic/mistral/icloud_email/icloud_app_password/memo_http_api_token/ts_authkey),
nanobot-data gemountet auf /home/nanobot/.nanobot, Tailscale-Hostname bewusst
`nanobot-k8s` statt `nanobot` (Swarm-Service belegt den Namen noch, kein Cutover),
hostPort 41642/UDP für direkte WireGuard-Verbindung ohne DERP-Relay.

**05-service.yaml** — simpler ClusterIP-Service auf Port 8765, nur für lokales
Debugging per `kubectl port-forward`, unabhängig vom Tailscale-Login-Status.

## Was zusätzlich nötig war (nicht in den Dateien)

- `kind load docker-image nanobot:latest --name learn` — kind-Nodes haben keinen
  Zugriff auf den lokalen Docker-Daemon, das Image muss explizit reingeladen werden.
- config.json existierte auf der leeren PVC nicht (frisches PVC = leer). Einmalig über
  einen Wegwerf-busybox-Pod mit demselben PVC-Mount reingeschrieben (kubectl cp).
  Keine Automatisierung dafür — bei PVC- oder Pod-Neuanlage muss das wiederholt werden.
- Empirisch verifiziert: Pods in kind-learn können `*.ts.net`-Tailscale-MagicDNS-Hostnamen
  aus dem Pod-Netzwerk heraus auflösen und erreichen (DNS via CoreDNS-Forwarding,
  TLS-Handshake erfolgreich) — daher funktionieren providers.ollama und
  tools.mcpServers.memo unverändert auch aus dem K8s-Pod.

## Bewusst noch nicht Teil dieser Runde

ComfyUI-Netzwerk, Open-WebUI/RAG-Netzwerk, embedding_cache-Volume, iCloud-MCP-Host-Mount,
Telegram-Channel (würde mit dem Swarm-Bot kollidieren, der denselben Bot-Token pollt),
tools.exec.sandbox/bwrap-Frage. Cutover (Umbenennen auf "nanobot", Swarm-Service
abschalten) ist bewusst nicht terminiert.