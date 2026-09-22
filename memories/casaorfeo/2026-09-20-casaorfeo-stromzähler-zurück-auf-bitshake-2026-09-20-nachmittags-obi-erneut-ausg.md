---
created: '2026-09-20T14:44:38.095+02:00'
extra:
  entities:
  - OBI Energy Tracker
  - Abgeleitete Sensoren
  - Casa
  - Vormittags
  - Energy
  - Tracker
  - Nachmittag
  - Stromversorgung
  - Entities
  - Registries
  - Eintr
  - Sensoren
  - Dashboard
  - Fund
  - Umbenennen
  - Switch
  - Blockade
  - Umbau
  - Ping
  - Scan
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
id: a5d8d108e2af43599ee7f5a2630a0fc5
normalized_hash: 6ff399c5cbda9b44
review_after: '2027-03-19T14:44:38.095000+02:00'
tags:
- casaorfeo
- homeassistant
- stromzaehler
- bitshake
- obi
- mqtt
- project:casaorfeo
title: 'CasaOrfeo: Stromzähler zurück auf bitShake (2026-09-20 nachmittags), OBI erneut
  ausgebaut'
type: decision
updated: '2026-09-20T14:44:38.095+02:00'
valid_at: '2026-09-20T14:44:38.095+02:00'
verification_state: unverified
---

CasaOrfeo, 2026-09-20 nachmittags. Kurswechsel am selben Tag: der Vormittags-Revert auf OBI Energy Tracker [77f5a74d] wurde noch am selben Nachmittag rückgängig gemacht — Nutzerentscheidung, die wiederkehrenden OBI-Backend-Ausfälle [c0c1a45e] wiegen schwerer als die umständliche Stromversorgung der bitShake-Hardware.

**Durchgeführt:**
- OBI erneut komplett entfernt: `custom_components/obi_energy` gelöscht, verwaiste Entities (`sensor.obi_zahlerstand_wh`, `sensor.obi_verbrauch_wh_bereinigt`, `sensor.obi_energy_bridge_verbrauch_kwh_cost`/`_cost_2`, `update.obi_energy_update`, `switch.obi_energy_pre_release`) + HACS-Tracking-Device aus den Registries entfernt. `.storage/hacs.repositories` (kompletter HACS-Store-Katalog, tausende Einträge) bewusst nicht angefasst.
- `.storage/energy` `stat_energy_from` zurück auf `sensor.stromzaehler_gesamt`.
- Abgeleitete Sensoren neu angelegt (Template-Sensor für Wh-Umrechnung, 2× `platform: statistics`, Filter- und Näherungsleistungs-Template) — rekonstruiert aus CLAUDE.md-Doku + Git-Historie der Dashboard-Datei (Commit `bee4128`), da die eigentlichen YAML-Sensordefinitionen nie in Git standen (`config/` gitignored).
- **Wichtiger technischer Fund:** Die ursprünglichen bitShake-Entity-Registry-Einträge (`sensor.stromzaehler_gesamt/_hochtarif/_niedertarif/_leistung/_einspeisung`, korrekte "ae"-Schreibweise) wurden beim Vormittags-Rückbau nie gelöscht, nur verwaist. Sobald die rohen `mqtt: sensor:`-Definitionen mit denselben `unique_id`-Werten neu angelegt werden, docken sie automatisch wieder an — kein manuelles Umbenennen nötig.
- Dashboard [dashboards/energie_details_dashboard.yaml] zurück auf die bitShake-Karten (Stand `bee4128`), die zwischenzeitlich ergänzten SwitchBot-Steckdosen-Karten blieben erhalten.

**Noch offen (bewusst zurückgestellt, keine Blockade):** Die fünf rohen MQTT-Sensordefinitionen fehlen noch — das bitShake-Gerät war beim Umbau offline (LWT=Offline, weder Ping/HTTP unter der alten IP `192.168.2.225` noch per Scan nach seiner MAC `ac:27:6e:36:8e:54` auffindbar), kein Live-Payload zur Verifikation der `MT174`-JSON-Feldpfade möglich. Nutzer hat sich für "später ergänzen, sobald das Gerät wieder läuft" entschieden. Nächster Schritt sobald das Gerät wieder Strom hat: `mosquitto_sub -t tele/tasmota_368E54/SENSOR` live mitschneiden, daraus die exakten `value_template`-Pfade bauen, in `configuration.yaml` unter dem markierten Kommentar ergänzen.

Details in CLAUDE.md unter „Stromzähler: zurück auf bitShake (2026-09-20, nachmittags)".