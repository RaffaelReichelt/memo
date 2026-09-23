---
created: '2026-07-31T08:27:01.911+02:00'
extra:
  entities:
  - Docker Swarm
  - Bekannter Nebeneffekt
  - Update
  - Ollama
  - Swarm
  - Auto
  - Stand
  - Pulls
  - Image
  - Sonntag
  - User
  - Login
  - Nebeneffekt
  - Downtime
  - Service
  - Replica
  - Sonntagnacht
  - Fenster
  - Ordner
  - Container
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
id: d68d364e49f04bedbe991f53f1f9aec5
normalized_hash: d550bd5e0b08c4a7
review_after: '2027-01-27T08:27:01.911000+02:00'
tags:
- ollama
- docker-swarm
- tailscale
- systemd-timer
- infra
- project:ollama
title: 'Ollama-Stack: automatisches Update von ollama + tailscale-sidecar'
type: decision
updated: '2026-07-31T08:27:01.911+02:00'
valid_at: '2026-07-31T08:27:01.911+02:00'
verification_state: unverified
---

Ollama-Stack (Docker Swarm, /home/raffael/Projekte/ollama) hat einen automatischen Update-Mechanismus für die `:latest`-Images bekommen, da `:latest` kein Auto-Update ist (Container bleiben sonst auf dem Stand des letzten Pulls/Builds).

Setup:
- Skript: `/home/raffael/.local/bin/ollama-stack-update.sh`
  1. `docker pull ollama/ollama:latest`
  2. `docker build --pull --no-cache -t ollama-tailscale-sidecar:latest /home/raffael/Projekte/ollama` (Sidecar ist ein lokal gebautes Image aus `FROM tailscale/tailscale:latest`, wird sonst nie aktualisiert)
  3. `docker service update --force --image ollama/ollama:latest ollama_ollama`
  4. `docker service update --force --image ollama-tailscale-sidecar:latest ollama_ollama-tailscale`
  5. `docker image prune -f`
- systemd-User-Units (Muster wie `memo-gx10-sync.timer`): `~/.config/systemd/user/ollama-stack-update.service` + `.timer`
- Timer: wöchentlich Sonntag 04:20 CEST (`OnCalendar=Sun 04:20`, `RandomizedDelaySec=10min`, `Persistent=true`), aktiviert via `systemctl --user enable --now ollama-stack-update.timer`
- Linger für User raffael ist aktiv, Timer läuft auch ohne Login
- Logs: `journalctl --user -u ollama-stack-update.service`
- Bekannter Nebeneffekt: `docker service update --force` verursacht kurze Downtime (Sekunden) pro Service, da nur 1 Replica — deswegen Sonntagnacht als Fenster gewählt

Zusätzlich wurde am 2026-07-30 der verwaiste Ordner `.open-webui` (1,9 GB, alte webui.db + Uploads/Fotos + vector_db, root-owned aus Container, kein Volume-Mount mehr in docker-compose.yml) gelöscht.