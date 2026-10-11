---
created: '2026-10-11T12:15:05.505+02:00'
extra:
  entities:
  - Tailscale Auth Keys Raffael
  - Tailscale Auth Keys
  - Alle Tailscale
  - Automation
  - Rotation
  - Tailscale
  - Auth
  - Keys
  - Raffael
  - Services
  - Stacks
  - Secret
  - Namen
  - Stack
  - Umgebung
  - Entrypoint
  - Login
  - Anmeldung
  - Laufzeit
  - Rotations
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
id: 3ebbc4063c634f9a8bc2d638f9115a70
normalized_hash: 8362f612d4181137
tags:
- nanobot
- todo
- tailscale
- secrets
- reminder
- project:nanobot
title: 'TODO: Zuverlässige Automation für die Rotation der Tailscale Auth Keys'
type: note
updated: '2026-10-11T12:15:05.505+02:00'
valid_at: '2026-10-11T12:15:05.505+02:00'
verification_state: unverified
---

Raffael möchte später daran erinnert werden (Stand 2026-10-11): Er braucht eine zuverlässige Automation für die Rotation der Tailscale Auth Keys.

Warum: Abgelaufene oder ungültige Auth-Keys haben schon mehrfach Services ausgeloggt. Bekannt: nanobot_tailscale brauchte am 2026-09-07 einen neuen Key (ts_authkey_v3, der alte ts_authkey war abgelaufen); hostmonitor_tailscale hatte am 2026-08-22 einen ungültigen gemeinsamen TS_AUTHKEY; bei der letzten Rotation waren nur nanobot und open-webui erledigt, weitere Stacks standen noch aus.

Randbedingungen: Docker-Secrets sind unveränderlich. Rotation heißt: neues Secret unter neuem Namen anlegen (z. B. ts_authkey_v4), die secrets:-Referenz im Stack umbiegen und neu deployen; docker secret create auf einen bestehenden Namen schlägt fehl. containerboot liest TS_AUTHKEY nur aus der Umgebung, deshalb ein Entrypoint-Shim, der das Secret einliest. Nach dem ersten erfolgreichen Login hält das persistente tailscale-state-Volume die Anmeldung; der Key wird danach nur bei Re-Login gebraucht (Node-Ablauf, Logout).

Idee zur Lösung: Statt langlebiger Keys ein OAuth-Client oder die Tailscale-API (Keys per API erzeugen, kurze Laufzeit, reusable), ein Rotations-Job (systemd-Timer oder Cron) der pro Stack einen neuen Key erzeugt, ein neues versioniertes Secret anlegt, den Service umstellt und prüft, dass der Node online ist (tailscale status), alte Secrets nach Erfolg entfernt, und bei Fehlern benachrichtigt. Alle Tailscale-Sidecars (nanobot, open-webui, ollama, ai-stack, odoo, opensearch, boinc, hostmonitor) einbeziehen. Sollte mit der Notiz zum Kunden-Rollout zusammenpassen (pro Kunde eigener Tailnet-Key).