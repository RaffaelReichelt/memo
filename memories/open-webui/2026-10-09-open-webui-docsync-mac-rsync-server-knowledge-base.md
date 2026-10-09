---
created: '2026-10-09T12:24:56.718+02:00'
extra:
  entities:
  - Open WebUI
  - Knowledge Base Serverseitiger Ersatz
  - Eigener Key
  - Erzeugt Manifest
  - Fehlgeschlagene Dateien
  - Base
  - Serverseitiger
  - Ersatz
  - Browser
  - Hintergrund
  - Server
  - Manifest
  - Pfade
  - Dateien
  - Docker
  - Container
  - Tage
  - Loader
  - Bilder
  - Stubs
  owner_principal: f0bd05cb3225
  trust_tier: agent_inferred
  visibility: owner
  write_policy:
    actor_id: memo
    allowed: true
    conflicts: []
    override: false
    policy_version: memo.write_policy.v1
    reason: allowed
    trust_tier: agent_inferred
    visibility: owner
id: 6f101e852f7a4a09896903bbc4d0acb3
normalized_hash: 14e76621c6f2880b
review_after: '2027-04-07T12:24:56.718000+02:00'
tags:
- open-webui
- docsync
- rsync
- icloud
- project:open-webui
title: 'Open WebUI docsync: Mac -> rsync -> Server -> Knowledge Base'
type: decision
updated: '2026-10-09T12:24:56.718+02:00'
valid_at: '2026-10-09T12:24:56.718+02:00'
verification_state: unverified
---

Serverseitiger Ersatz fuer den Browser-Sync (der nur bei offenem Tab laeuft, Tab im Hintergrund pausiert -> Upload stoppt). Aufbau (2026-10-09):
- Mac: ~/docsync/docsync-push.sh (Repo: open-webui/docsync/mac/) per launchd alle 30 min. Nutzt /opt/homebrew/bin/rsync 3.5 (macOS /usr/bin/rsync ist openrsync, inkompatibel mit rrsync). Eigener Key ~/.ssh/docsync_ed25519, auf dem Server in authorized_keys mit command="rrsync -wo /home/raffael/docsync/documents",restrict. Erzeugt Manifest docsync-manifest.txt (Zeile 1 '#epoch', dann alle Pfade inkl. iCloud-Stubs) und schiebt es nach den Dateien. rsync ohne --delete.
- Server: Swarm-Service open-webui_docsync (Image open-webui-custom, Skript open-webui/docsync/docsync.py, bind-mount ro). Nutzt /api/v1/knowledge/{id}/sync/diff, dirs/create, files upload+process/status, sync/cleanup mit API-Key aus Docker-Secret docsync_api_key. Ziel-KB b547a4c9-3eee-44ca-8e2c-e678b0deb092. Healthcheck muss disabled sein (Image-Healthcheck killt sonst den Container).
- Loeschen: DOCSYNC_ENABLE_DELETE (Default false). Nur mit frischem Manifest; Stubs gelten als vorhanden; Abbruch wenn > min(20, 3 %) Loeschungen oder Manifest < 50 % des letzten; Mirror-Dateien gehen 30 Tage in /home/raffael/docsync/state/trash. DOCSYNC_DRY_RUN schaltet alles auf Nur-Log.
- State liegt root-owned in /home/raffael/docsync/state (Aenderungen via docker exec). Fehlgeschlagene Dateien: Retry nach 24 h, max 3; 'Duplicate content' permanent.
- Caption-Fixes im Loader: Platzhalter '<No text content found>' zaehlt als leer, HEIC via pillow-heif, Timeout 1200 s, keep_alive 30m.
Umfang: ~/Documents = 25684 Dateien, 1763 Bilder, 0 Stubs (Stand 2026-10-09).