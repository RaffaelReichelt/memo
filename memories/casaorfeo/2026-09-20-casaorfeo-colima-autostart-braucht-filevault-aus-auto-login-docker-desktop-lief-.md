# CasaOrfeo: Colima-Autostart braucht FileVault-aus + Auto-Login; Docker Desktop lief parallel weiter

CasaOrfeo / Mac Mini, 2026-09-20. Ausgangsfrage: Colima startet nicht beim Boot, erst nach manueller Anmeldung.

**Ursache war zweistufig, beides musste gelöst werden:**
1. FileVault war aktiv → der Mac bleibt nach einem Neustart am Pre-Boot-Unlock-Screen stehen und bootet ohne physische Passworteingabe gar nicht ins OS. macOS sperrt bei aktivem FileVault zusätzlich die Auto-Login-Option (ausgegraut).
2. `brew services start colima` legt einen **LaunchAgent** (`~/Library/LaunchAgents/sh.brew.colima.plist`) an, der per Definition erst bei der Benutzeranmeldung lädt. Die `LimitLoadToSessionType`-Liste im Plist (Background/System/LoginWindow) ändert daran nichts.

**Wichtige Gegenprüfung:** Docker Desktop hätte exakt dasselbe Problem — dessen VM startet ausschließlich über die GUI-App als *Login*-Item; die `/Library/LaunchDaemons/com.docker.*` sind nur privilegierte Helper (vmnetd, socket) und starten keine VM. Die Runtime-Wahl ändert am Autostart also nichts. Ein LaunchDaemon ist für Colima ebenfalls kein Workaround (vz braucht Entitlements + Benutzer-Session, VM-State liegt unter /Users/raffael/.colima).

**Nebenbefund, potenziell datenzerstörend:** Docker Desktop und Colima liefen gleichzeitig mit je einem vollständigen Container-Satz aus derselben docker-compose.yml — zwei HA-Instanzen auf demselben Bind-Mount (`config/`, dieselbe .storage/ und home-assistant_v2.db). `docker ps` zeigt nur den aktiven Kontext, deshalb monatelang unbemerkt. Nach einem Runtime-Wechsel immer `docker --context <anderer> ps -a` gegenprüfen.

**Entscheidung des Nutzers:** Colima behalten, Docker Desktop stillgelegt (Autostart aus, App bleibt als Rückfalloption installiert).

Details in CLAUDE.md unter „Umstieg Docker Desktop → Colima". Siehe auch [[anstehende-hardware-aenderungen]].
