---
created: '2026-10-11T12:09:24.304+02:00'
extra:
  entities:
  - Modell Kolibri
  - Tailscale
  - Nutzer
  - Branch
  - Server
  - Operator
  - Zugangsdaten
  - Gastes
  - Hand
  - Zugriff
  - Chat
  - Verzeichnis
  - Service
  - Docker
  - Klartext
  - Punkte
  - Skills
  - Pers
  - Dream
  - Kolibri
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
id: 8e2f58ccb8ad4fe7b6a6ccb3752c9649
normalized_hash: dd7ecfc45288b3b1
tags:
- nanobot
- todo
- tailscale
- mcp
- reminder
- project:nanobot
title: 'nanobot TODO: Selbstbedienung für Tailscale-Gäste (MCP-Zugangsdaten)'
type: note
updated: '2026-10-11T12:09:24.304+02:00'
valid_at: '2026-10-11T12:09:24.304+02:00'
verification_state: unverified
---

Raffael möchte später daran erinnert werden (Stand 2026-10-11): Selbstbedienung für Tailscale-Gäste bauen, mit der jeder Nutzer seine eigenen MCP-Zugangsdaten (z. B. iCloud) selbst einträgt.

Ausgangslage (bereits gebaut und deployt, Commits auf Branch `local`): MCP-Server haben `visibility` ("all" | "operators"); `tools.userOverrides.<login>.mcpServers` startet eigene Server pro Nutzer. Live ist `icloud` auf "operators" gestellt. Heute trägt der Operator die Zugangsdaten eines Gastes von Hand in `/var/lib/nanobot/config.json` ein (Klartext). Gäste haben absichtlich keinen Zugriff auf WebUI-Einstellungen.

Offen: Seite oder Chat-Befehl, die Zugangsdaten sicher ins private Verzeichnis des Gastes schreibt (nur er sieht sie), Neustart seiner MCP-Verbindung ohne Service-Neustart, idealerweise Docker-Secrets statt Klartext. Weitere offene Punkte: gemeinsame Skills und SOUL.md auf Persönliches prüfen, history.jsonl ist global (nur Dream filtert), Dream läuft mit dem 65-GB-Modell Kolibri alle 2 Stunden.