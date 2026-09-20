---
created: '2026-09-20T08:54:36.437+02:00'
extra:
  entities:
  - YAML HA
  - Home Assistant
  - Mac Mini Docker
  - Die YAML
  - Kein Fehler
  - Bad Request
  - In CLAUDE
  - Home
  - Assistant
  - Mini
  - Docker
  - Store
  - Container
  - Fehler
  - Request
  - Liste
  - Eintrag
  - Proxy
  - Tailscale
  - Migrations
  owner_principal: 4c3e0ce04a3b
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
id: 02b03dcda8164b35ae721622962b832b
normalized_hash: 6510a523d44f2b61
tags:
- home-assistant
- casaorfeo
- docker
- tailscale
- trusted_proxies
- storage-migration
- project:casaorfeo
title: HA http.trusted_proxies seit 2026.9.3 nur noch über .storage/http, nicht mehr
  YAML
type: bug
updated: '2026-09-20T08:54:36.437+02:00'
valid_at: '2026-09-20T08:54:36.437+02:00'
verification_state: unverified
---

HA `http.trusted_proxies` in `configuration.yaml` ist seit Home Assistant `2026.9.3` wirkungslos (CasaOrfeo-Projekt, Mac Mini Docker-Stack). Ursache: HA hat die `http:`-Konfiguration von reinem YAML-Reload auf einen persistenten Store umgestellt (`homeassistant/components/http/config.py`: "YAML config is only migrated once. Subsequent boots will ignore YAML and use the store exclusively."). Die YAML wird nur einmalig übernommen (`.storage/http` → `data.yaml_migration_done: true`); jede spätere Änderung an `configuration.yaml` unter `http:` (z.B. eine neue trusted_proxies-IP) wird stillschweigend ignoriert, egal wie oft der Container neu gestartet/recreated wird. Kein Fehler im Log — HA blockt einfach weiter mit `400 Bad Request` ("Received X-Forwarded-For header from an untrusted proxy <IP>"), obwohl die IP längst in der YAML steht.

Diagnose-Trick: `docker exec homeassistant python3 -c "from homeassistant.util.yaml import loader; print(loader.load_yaml('/config/configuration.yaml')['http'])"` zeigt die korrekte YAML — der Fehler bleibt trotzdem, das beweist die Store-Diskrepanz.

Fix: Die tatsächlich wirksame Liste liegt in `config/.storage/http` unter `data.stable.trusted_proxies` (Format `"<ip>/32"` je Eintrag). Container stoppen, Feld dort direkt ergänzen (Backup als `.bak`), Container wieder starten. Bei künftigen neuen vertrauenswürdigen Proxy-IPs (z.B. wechselnde Docker-Desktop-Gateway-IPs für den Tailscale→HA-Pfad) zuerst `.storage/http` prüfen/ändern, nicht die YAML. In CLAUDE.md unter der Migrations-Sektion dokumentiert (2026-09-20).