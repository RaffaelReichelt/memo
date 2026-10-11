---
created: '2026-10-11T12:15:01.668+02:00'
extra:
  entities:
  - Benutzerverzeichnis Raffael
  - Ein Image
  - Docker Swarm
  - Keine Abh
  - Konkrete Stellen
  - Quellen
  - Benutzerverzeichnis
  - Raffael
  - Installation
  - Services
  - Daten
  - Rollout
  - Kunden
  - Workspace
  - Stück
  - Richtung
  - Image
  - Verzeichnis
  - Server
  - System
  owner_principal: f0bd05cb3225
  trust_tier: agent_inferred
  visibility: owner
  write_policy:
    actor_id: memo-update
    allowed: true
    conflicts: []
    override: false
    policy_version: memo.write_policy.v1
    reason: allowed
    trust_tier: agent_inferred
    visibility: owner
id: 835bc103f1954ede88c9ce5824248e51
normalized_hash: 247243dd2c7fad96
tags:
- nanobot
- todo
- rollout
- deployment
- reminder
- project:nanobot
title: 'TODO: Kunden-Rollout aller Services – Image + docker-compose statt Quellen
  im Benutzerverzeichnis'
type: note
updated: '2026-10-11T12:16:03.065+02:00'
valid_at: '2026-10-11T12:15:01.668+02:00'
verification_state: unverified
---

Raffael möchte später daran erinnert werden (Stand 2026-10-11): Die aktuelle Installation aller seiner Services (Quellen, Konfigurationen und Daten im Benutzerverzeichnis /home/raffael/Projekte/...) ist für einen Rollout beim Kunden ungeeignet. Auch der Workspace unter /var/lib/nanobot geht nur ein Stück in diese Richtung.

Ziel: Ein Image plus eine docker-compose.yml in ein geeignetes Verzeichnis im Server-Dateisystem installieren und dort vom System (z. B. systemd / Docker Swarm) starten lassen. Keine Abhängigkeit mehr von Quellcode, Bind-Mounts oder Pfaden im Benutzerverzeichnis.

Konkrete Stellen im Ist-Zustand, die dafür angefasst werden müssen: docker-compose.yml bindet /home/raffael/Projekte/icloud-mcp (ICLOUD_MCP_DIR) und /home/raffael/Projekte/nanobot/webui/serve.json (NANOBOT_SERVE_JSON) ein; das Image wird lokal gebaut (docker build, kein Registry); Config und Workspace liegen in /var/lib/nanobot (config.json mit Klartext-API-Keys, z. B. rag-Keys unter tools.userOverrides); der Tailscale-Auth-Key ist ein externes Docker-Secret (ts_authkey_v3); weitere Services (open-webui, ollama, comfyui, odoo, opensearch, hostmonitor) liegen in eigenen Stacks.

Zu klären: Image-Registry oder docker save/load, Verzeichnislayout auf dem Server (z. B. /opt/<produkt>/), Secrets-Handling statt Klartext in config.json, Config-Vorlage pro Kunde, Update- und Rollback-Verfahren. Siehe auch die Notiz zur Tailscale-Key-Rotation.

GELTUNGSBEREICH (Präzisierung von Raffael, 2026-10-11): Das gilt für ALLE laufenden Services und Stacks, nicht nur für nanobot. Auf dem Swarm-Node gx10-b3ed laufen derzeit: nanobot, open-webui (inkl. tika, docsync), ollama, ai-stack (comfyui), odoo, postgres-odoo, opensearch (node1, dashboards, logstash, journald-/stats-shipper), boinc-dashboard, hostmonitor, jeweils mit eigenem Tailscale-Sidecar. Für jeden Stack gilt dasselbe Ziel: fertiges Image plus docker-compose.yml in einem Verzeichnis im Server-Dateisystem, keine Quellen, Konfigurationen oder Daten im Benutzerverzeichnis, vom System gestartet. Das Rollout-Konzept (Verzeichnislayout, Registry bzw. save/load, Secrets, Config-Vorlage pro Kunde, Update/Rollback) sollte einheitlich für alle Stacks entworfen werden, statt pro Service einzeln.