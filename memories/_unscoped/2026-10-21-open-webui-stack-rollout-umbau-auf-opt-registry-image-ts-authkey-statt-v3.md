---
created: '2026-10-21T12:01:31.841+02:00'
extra:
  entities:
  - Open WebUI Stack
  - Open WebUI
  - Stack
  - Services
  - Swarm
  - Rotation
  - Compose
  - Image
  - Stacks
  - Verzeichnis
  - DOCSYNC
  - SHA
  - NICHT
  - sudo ./install-opt.sh
  - ./deploy.sh
  - DOCSYNC=1
  - ts_authkey
  - ts_authkey_v3
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
id: c3274366efc54475aed67486a657dc24
normalized_hash: 5c520c4ecb105872
review_after: '2027-04-19T12:01:31.841000+02:00'
tags:
- open-webui
- rollout
- tailscale
- ts_authkey
- registry
- docsync
title: 'Open WebUI Stack: Rollout-Umbau auf /opt, Registry-Image, ts_authkey statt
  v3'
type: decision
updated: '2026-10-21T12:01:31.841+02:00'
valid_at: '2026-10-21T12:01:31.841+02:00'
verification_state: unverified
---

# Open WebUI rolloutfaehig (Stand 2026-10-21)

Projekt: /home/raffael/Projekte/open-webui (Branch local). Teil von [835bc103] (Kunden-Rollout aller Services), betrifft [0523144a] (ts_authkey-Rotation).

- Konfiguration kommt nach /opt/open-webui (root, 755/644, .env 600) per `sudo ./install-opt.sh`; Deploy per `./deploy.sh` (`DOCSYNC=1` fuegt docker-compose.docsync.yml hinzu).
- Image: registry2.tail5a2ccd.ts.net/open-webui-custom, Tag = Git-SHA (a138d19ad gepusht). build-push.sh baut und pusht. OPENWEBUI_IMAGE in .env auf den SHA-Tag pinnen, nicht :latest.
- Tailscale-Sidecar nutzt das Swarm-Secret `ts_authkey` (von der Rotation bedient), NICHT mehr `ts_authkey_v3`.
- docsync ist optional (eigene Compose-Datei, Skript im Image unter /opt/docsync/docsync.py, Default-Datenpfad /srv/docsync, DRY_RUN=true). Raffaels privater iCloud-Sync (~/Documents -> /home/raffael/docsync) bleibt ueber seine .env (DOCSYNC_DATA_DIR).
- Offen: sudo install-opt.sh, Redeploy (open-webui startet kurz neu; docsync steht auf 0/0). Die anderen Stacks folgen einzeln im jeweiligen Verzeichnis.