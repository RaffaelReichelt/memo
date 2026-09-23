---
created: '2026-09-07T06:38:39.968+02:00'
extra:
  entities:
  - Homelab Swarm
  - Beide Tailscale
  - Separater Fund
  - Rotation
  - Stacks
  - Docker
  - Swarm
  - Stack
  - Problem
  - Tailscale
  - Zeitbombe
  - State
  - Secret
  - Einsatz
  - Tailnet
  - Fund
  - Rest
  - Vorsitzung
  - Image
  - Repo
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
id: ca6a91121ff440a3ab2e674417e73bab
normalized_hash: 5e5c21eac368f8db
tags:
- homelab
- tailscale
- docker-swarm
- nanobot
- open-webui
- project:open-webui
title: 'Tailscale ts_authkey Rotation: nanobot + open-webui gefixt, weitere Stacks
  noch offen'
type: fact
updated: '2026-09-07T06:42:23.444+02:00'
valid_at: '2026-09-07T06:38:39.968+02:00'
verification_state: unverified
---

Der geteilte Docker-Secret `ts_authkey` (Homelab Swarm-Host gx10-b3ed, erstellt 2026-07-04) ist abgelaufen/ungültig geworden ("invalid key: API key does not exist" beim tailscale-Sidecar-Login). Betroffen waren alle Stacks, die ihn noch referenzierten: `nanobot`, `open-webui` (kompletter Stack war zudem gar nicht deployed — separates Problem, siehe unten), sowie potenziell `ai-stack`/ComfyUI, `opensearch`, `embedding`, `schmerzprotokoll` (laufen weiter, weil sie eine noch gültige persistierte Tailscale-Session haben und den Key aktuell nicht neu brauchen — tickende Zeitbombe bei nächstem State-Verlust).

Fix (2026-09-07): beide betroffenen `docker-compose.yml` (`/home/raffael/Projekte/nanobot/docker-compose.yml` und `/home/raffael/Projekte/open-webui/docker-compose.yml`) im `secrets:`-Block auf den bereits vorhandenen, funktionierenden Secret `ts_authkey_v3` (erstellt 2026-08-23, schon produktiv bei `odoo_tailscale-odoo` und `hostmonitor` im Einsatz) umgebogen:
```yaml
secrets:
  ts_authkey:
    external: true
    name: ts_authkey_v3
```
Danach `docker stack deploy` je Stack. Beide Tailscale-Sidecars authentifizieren jetzt sauber, Geräte `nanobot` und `open-webui` sind wieder im Tailnet sichtbar.

Separater Fund unterwegs: der `open-webui`-Stack war komplett undeployed (kein `docker stack ls`-Eintrag) — vermutlich Rest einer unterbrochenen Vorsitzung (Rebuild mit neuem Image geplant, nie redeployed). Daten/Volumes waren unangetastet. Neu gebaut (`docker build -t open-webui-custom:local .` aus dem Repo-Root, kein Dockerfile.custom mehr, nur `Dockerfile` mit bereits eingebranntem `NODE_OPTIONS=--max-old-space-size=8192`) ergab exakt dasselbe Image (nur Cache-Hits, Code unverändert seit gestern) und wurde dann deployed.

Offen/noch nicht angegangen: `ai-stack` (ComfyUI), `opensearch`, `embedding`, `schmerzprotokoll` hängen noch am toten `ts_authkey`-Secret und sollten bei Gelegenheit ebenfalls auf `ts_authkey_v3` (oder je einen dedizierten Key) umgestellt werden, bevor sie beim nächsten Neustart/State-Verlust denselben Fehler zeigen.

**Update (2026-09-07, abgeschlossen):** Auch `ai-stack`/ComfyUI (`/home/raffael/Projekte/ComfyUI/docker-compose.yml`, deployed als Stack "ai-stack") und `opensearch` (`/home/raffael/Projekte/opensearch/docker-compose.yml`) wurden auf `ts_authkey_v3` umgestellt und redeployed — beide liefen sauber durch, da ihre Tailscale-Session noch gültig war (kein Re-Auth nötig). `embedding` und `schmerzprotokoll` referenzierten den toten Secret ebenfalls, sind aber inzwischen gar nicht mehr deployed (nicht mehr gebraucht) — kein Handlungsbedarf dort. Damit hängen jetzt alle 6 aktiven Stacks (`nanobot`, `open-webui`, `ai-stack`, `opensearch`, `odoo`, `hostmonitor`) einheitlich an `ts_authkey_v3`. Der tote `ts_authkey`-Secret hat keine Abhängigen mehr und kann bei Gelegenheit per `docker secret rm ts_authkey` aufgeräumt werden. Kein offener Punkt mehr aus diesem Vorfall.