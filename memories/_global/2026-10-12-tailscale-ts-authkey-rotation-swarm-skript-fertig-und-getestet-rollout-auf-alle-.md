---
created: '2026-10-12T13:43:36.917+02:00'
extra:
  entities:
  - Tailscale Auth
  - Auth Key
  - Beim Test
  - Getaggte Geraete
  - Ein Auth Key
  - Echter Test
  - Das OAuth
  - Laufender Service
  - Bei Rollout
  - Rotation
  - Stacks
  - Auth
  - Homelab
  - Swarm
  - Compose
  - Service
  - Phasen
  - Zeit
  - Nutzern
  - Projekt
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
id: 0523144adc4f41bc936065b60b963e56
normalized_hash: 50a1ff77907c8e0c
review_after: '2027-04-10T13:43:36.917000+02:00'
tags:
- homelab
- tailscale
- docker-swarm
- ts_authkey
- key-rotation
- gx10-b3ed
title: 'Tailscale ts_authkey Rotation (Swarm): Skript fertig und getestet, Rollout
  auf alle Stacks steht aus'
type: decision
updated: '2026-10-24T08:50:28.985+02:00'
valid_at: '2026-10-12T13:43:36.917+02:00'
verification_state: unverified
---

# Tailscale Auth-Key-Rotation fuer den Homelab-Swarm (Stand 2026-10-12)

Projekt: /home/raffael/Projekte/key-rotation (kein Git). Loest das TODO [3ebbc406].

**Wie es funktioniert**
- `scripts/rotate-ts-authkey.sh` holt per OAuth-Client (Scope `auth_keys`) einen neuen Auth Key und legt ihn als Swarm-Secret `SECRET_NAME` ab. Die `.env` und die Compose-Dateien werden nicht angefasst.
- Docker-Secrets sind unveraenderlich und lassen sich nicht loeschen, solange ein Service sie einbindet. Deshalb zwei Phasen ohne secretlose Zeit: (1) neuer Key als `<name>_rotating`, in allen Nutzern tauschen; (2) `<name>` loeschen, neu anlegen, zurueck tauschen, `_rotating` entfernen. Zielpfad /run/secrets/<name>, uid, gid und mode bleiben erhalten. Jeder betroffene Service startet pro Rotation zweimal neu.
- Konfiguration in rotation.env (OAuth-Credentials, TS_TAILNET, SECRET_NAME, Key-Parameter). Aufruf manuell: `ROTATION_ENV_FILE=... ./scripts/rotate-ts-authkey.sh`. systemd-Units (weekly) liegen im Projekt, sind noch NICHT installiert.
- Integrationstest: `tests/test-rotation.sh` (Wegwerf-Stack `tstest`, curl gestubbt, raeumt auf).

**Erkenntnisse**
- Beim Test-Bau gefunden: `docker service update --secret-add ...,mode=` parst "0440" oktal, "440" dezimal. `File.Mode` aus `docker service inspect` ist dezimal und muss unveraendert uebergeben werden.
- OAuth-erzeugte Keys muessen getaggt sein. Tag `tag:swarm` wurde in der ACL unter `tagOwners` angelegt (Owner `autogroup:admin`), die OAuth-Credential hat dieses Tag. Getaggte Geraete gehoeren dem Tag, ihr Node-Key laeuft nicht ab.
- Ein Auth Key wird nur bei der Erstanmeldung gebraucht. Sidecars mit persistiertem `tailscale-state` nutzen ihn nicht. Echter Test mit leerem State (Basis-Image tailscale/tailscale, Wegwerf-Hostname): Anmeldung klappt, Geraet erscheint als `tag:swarm`, 17 Peers sichtbar. Ein angehaengtes `?ephemeral=true` am Key wurde abgelehnt ("unable to validate API key"), ohne Anhang klappt es.
- Erster echter Lauf am 2026-10-12 mit SECRET_NAME=ts_authkey: betraf nur `ollama_ollama-tailscale` (der einzige Nutzer), erfolgreich.
- Das OAuth-Token-Credential kann keine Geraete loeschen (nur auth_keys). Testgeraete muessen in der Admin-Console von Hand geloescht werden.

**Stand der Secrets im Swarm (2026-10-12)**
- `ts_authkey`: nur ollama (neu rotiert). Laufender Service weicht von ollama/docker-compose.yml ab, die schon `name: ts_authkey_v3` setzt: bei Redeploy springt ollama auf v3.
- `ts_authkey_v3`: ai-stack, boinc-dashboard, nanobot, odoo, open-webui, opensearch (noch nicht rotiert).
- `ts_authkey_hostmonitor`: hostmonitor (eigenes Secret, wird von keinem Lauf erfasst).

**Offen / Entscheidung**
Raffael will vor dem Rollout alle Stacks einzeln ueberarbeiten und denkt noch nach. Noch nichts auf ts_authkey_v3 oder per Timer ausgerollt. Bei Rollout auf v3 werden 6 Stacks doppelt neu gestartet, besser ausserhalb der Nutzungszeit. Offene Frage: ein gemeinsames Tag/Secret fuer alle oder pro Stack getrennt, und monatlich statt woechentlich rotieren (Neustarts).

**Update 2026-10-24 (geprüft per docker service inspect):** Auf `ts_authkey` (rotiert 2026-10-12) laufen jetzt: ollama_ollama-tailscale, ai-stack_tailscale, open-webui_tailscale, registry2_tailscale. Noch auf `ts_authkey_v3`: odoo_tailscale-odoo, opensearch_tailscale; nanobot (und boinc-dashboard) nicht geprüft, laut Stand 10-12 ebenfalls v3. Raffael stellt odoo, opensearch und nanobot als Nächstes um. Der systemd-Timer ist weiterhin NICHT installiert (kein Rotations-Timer in `systemctl list-timers`, nichts in /etc/systemd/system), TODO [3ebbc406] damit noch offen.