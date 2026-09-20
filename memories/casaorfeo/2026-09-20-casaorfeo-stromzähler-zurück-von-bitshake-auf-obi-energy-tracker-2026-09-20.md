---
created: '2026-09-20T09:45:58.108+02:00'
extra:
  entities:
  - OBI Energy Tracker
  - Die Stromversorgung
  - OBIs Cloud
  - Die OBI
  - NICHT Teil
  - In CLAUDE
  - Energy
  - Tracker
  - Smart
  - Stromversorgung
  - Details
  - Einsatz
  - Ersatz
  - Cloud
  - Momentanleistung
  - Template
  - Stand
  - Commit
  - Teil
  - Reverts
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
id: 77f5a74dbad147f9b521b26a2431c6af
normalized_hash: 98ac9832ce1a1d51
review_after: '2027-03-19T09:45:58.108000+02:00'
tags:
- home-assistant
- casaorfeo
- energie
- obi
- bitshake
- revert
- project:casaorfeo
title: 'CasaOrfeo: Stromzähler zurück von bitShake auf OBI Energy Tracker (2026-09-20)'
type: decision
updated: '2026-09-20T09:45:58.108+02:00'
valid_at: '2026-09-20T09:45:58.108+02:00'
verification_state: unverified
---

CasaOrfeo: Stromzähler-Integration am 2026-09-20 von bitShake SmartMeterReader zurück auf OBI Energy Tracker umgestellt. Grund (Nutzerangabe): Die Stromversorgung der bitShake-Hardware hat nicht funktioniert (keine weiteren technischen Details genannt). bitShake war seit 2026-08-10 im Einsatz, ursprünglich selbst als Ersatz für OBI eingeführt, weil OBIs Cloud-Backend anhaltend unzuverlässig war (18h-Ausfälle, nie funktionierende Momentanleistung).

Durchgeführt: `mqtt:`-Block mit den fünf bitShake-Sensoren, drei abgeleitete Template-Sensoren (Wh-Umrechnung, Ausreißer-Filter, Näherungsleistung) und zwei `platform: statistics`-Sensoren komplett aus `configuration.yaml` entfernt. `dashboards/energie_details_dashboard.yaml` auf den Stand vor dem bitShake-Umbau zurückgesetzt (per `git show` aus der Commit-Historie rekonstruiert, referenziert wieder `sensor.obi_*`). `.storage/energy` (natives HA-Energie-Dashboard) `stat_energy_from` zurück auf `sensor.obi_energy_bridge_verbrauch_kwh`.

WICHTIG, noch offen: Die OBI-Integration selbst (`custom_components/obi_energy` + Config-Entry) wurde am 2026-08-10 vollständig deinstalliert und ist NICHT Teil dieses Reverts wieder da — muss vom Nutzer manuell über HACS neu installiert und per Cloud-Login in der HA-UI neu eingerichtet werden, sonst bleiben die zurückgestellten Dashboard-Karten leer. Die früheren OBI-Hilfssensoren (sensor.obi_zaehlerstand_wh etc.) waren nie in Git (config/ ist gitignored) und müssten bei Bedarf neu gebaut werden. In CLAUDE.md unter "Stromzähler: zurück auf OBI Energy Tracker (2026-09-20)" dokumentiert, bitShake-Historie bleibt darunter zur Referenz stehen falls die Hardware doch reaktiviert wird.