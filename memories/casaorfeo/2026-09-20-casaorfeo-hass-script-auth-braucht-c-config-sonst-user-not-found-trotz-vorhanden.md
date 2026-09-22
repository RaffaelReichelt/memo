---
created: '2026-09-20T13:05:24.705+02:00'
extra:
  entities:
  - Nutzer CasaOrfeo
  - Funktionierender Befehl
  - Nutzer
  - Casa
  - Aussperrung
  - Container
  - Config
  - Parameter
  - Befehl
  - Passwort
  - Pflicht
  - Klartext
  - Kommandozeile
  - Session
  - Shell
  - Format
  - Vorsicht
  - CasaOrfeo
  - JSON
  - CLI
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
id: fa70310d579f4585a4aba7089a68d631
normalized_hash: ec9d7ff7052a85d6
tags:
- casaorfeo
- homeassistant
- docker
- auth
- password-reset
- project:casaorfeo
title: 'CasaOrfeo: hass --script auth braucht -c /config, sonst "User not found" trotz
  vorhandenem Nutzer'
type: fact
updated: '2026-09-20T13:05:24.705+02:00'
valid_at: '2026-09-20T13:05:24.705+02:00'
verification_state: unverified
---

CasaOrfeo, 2026-09-20. Passwort-Reset für lokalen HA-Nutzer nach Aussperrung.

**Fallstrick:** `hass --script auth change_password <user> <pw>` (im `homeassistant`-Container ausgeführt, egal ob via `docker exec` oder direkt als root im Container) meldet **"User not found"**, obwohl der Nutzer nachweislich in `config/.storage/auth_provider.homeassistant` existiert (per direktem JSON-Read verifiziert).

**Ursache:** Der `hass`-CLI-Aufruf braucht das Config-Verzeichnis explizit über `-c /config`. Ohne den Parameter liest er ein anderes (leeres) Default-Verzeichnis — `hass --script auth list` zeigt dann `Total users: 0`, mit `-c /config` korrekt den echten Nutzer.

**Funktionierender Befehl:**
```
hass --script auth -c /config change_password <username> '<neues-passwort>'
```
Kein interaktiver Passwort-Prompt in dieser HA-Version — `new_password` ist Pflicht-Positionsargument, muss also im Klartext auf der Kommandozeile stehen (am besten direkt im Container-Terminal ausführen, nicht über eine Session, die man nicht selbst kontrolliert, wegen Shell-History).

Verifikation danach ohne das Passwort zu sehen: `stat` auf `auth_provider.homeassistant` zeigt neue mtime, Hash-Präfix `$2b$12$...` (bcrypt) bestätigt gültiges Format.

Siehe auch [[db-nicht-im-betrieb-direkt-lesen]] (gleiche Vorsicht bei .storage-Dateien, hier aber unproblematisch da reines JSON ohne WAL).