---
created: '2026-09-20T12:42:25.707+02:00'
extra:
  entities:
  - Docker Desktop
  - Mac Mini
  - Wichtige Gegenprüfung
  - Die Runtime
  - Ein LaunchDaemon
  - Umstieg Docker Desktop
  - File
  - Desktop
  - Casa
  - Mini
  - Boot
  - Anmeldung
  - Neustart
  - Passworteingabe
  - Auto
  - Definition
  - Benutzeranmeldung
  - Plist
  - Gegenprüfung
  - Problem
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
id: 759511c3daf84e13897db36bc94bb2d6
normalized_hash: 59b32bb2ae57cbe1
tags:
- casaorfeo
- homeassistant
- colima
- macos
- autostart
- docker
- project:casaorfeo
title: 'CasaOrfeo: Colima-Autostart braucht FileVault-aus + Auto-Login; Docker Desktop
  lief parallel weiter'
type: fact
updated: '2026-09-20T12:42:25.707+02:00'
valid_at: '2026-09-20T12:42:25.707+02:00'
verification_state: unverified
---

CasaOrfeo / Mac Mini, 2026-09-20. Ausgangsfrage: Colima startet nicht beim Boot, erst nach manueller Anmeldung.

**Ursache war zweistufig, beides musste gelöst werden:**
1. FileVault war aktiv → der Mac bleibt nach einem Neustart am Pre-Boot-Unlock-Screen stehen und bootet ohne physische Passworteingabe gar nicht ins OS. macOS sperrt bei aktivem FileVault zusätzlich die Auto-Login-Option (ausgegraut).
2. `brew services start colima` legt einen **LaunchAgent** (`~/Library/LaunchAgents/sh.brew.colima.plist`) an, der per Definition erst bei der Benutzeranmeldung lädt. Die `LimitLoadToSessionType`-Liste im Plist (Background/System/LoginWindow) ändert daran nichts.

**Wichtige Gegenprüfung:** Docker Desktop hätte exakt dasselbe Problem — dessen VM startet ausschließlich über die GUI-App als *Login*-Item; die `/Library/LaunchDaemons/com.docker.*` sind nur privilegierte Helper (vmnetd, socket) und starten keine VM. Die Runtime-Wahl ändert am Autostart also nichts. Ein LaunchDaemon ist für Colima ebenfalls kein Workaround (vz braucht Entitlements + Benutzer-Session, VM-State liegt unter /Users/raffael/.colima).

**Nebenbefund, potenziell datenzerstörend:** Docker Desktop und Colima liefen gleichzeitig mit je einem vollständigen Container-Satz aus derselben docker-compose.yml — zwei HA-Instanzen auf demselben Bind-Mount (`config/`, dieselbe .storage/ und home-assistant_v2.db). `docker ps` zeigt nur den aktiven Kontext, deshalb monatelang unbemerkt. Nach einem Runtime-Wechsel immer `docker --context <anderer> ps -a` gegenprüfen.

**Entscheidung des Nutzers:** Colima behalten, Docker Desktop stillgelegt (Autostart aus, App bleibt als Rückfalloption installiert).

Details in CLAUDE.md unter „Umstieg Docker Desktop → Colima". Siehe auch [[anstehende-hardware-aenderungen]].