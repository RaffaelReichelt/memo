---
created: '2026-09-28T06:40:30.026+02:00'
extra:
  entities:
  - Casa Orfeu
  - Hub Minis
  - Ein Device
  - Nach Korrektur
  - Entities
  - Casa
  - Lampen
  - Raum
  - Problem
  - Orfeu
  - Vortag
  - Hausinventar
  - Tuya
  - Samsung
  - Sensoren
  - Switch
  - Minis
  - Namenskollisionen
  - Duplikate
  - Hardware
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
id: 8cb821a87a0a40b8a23b8a7fb00d0516
normalized_hash: de3780b6f32a181f
tags:
- casaorfeo
- homeassistant
- smartthings
- entity-registry
- device-registry
- hue
- project:casaorfeo
title: 'CasaOrfeo: SmartThings-Geisterintegration duplizierte 95 Entities/59 Geräte,
  entfernt'
type: bug
updated: '2026-09-28T06:40:30.026+02:00'
valid_at: '2026-09-28T06:40:30.026+02:00'
verification_state: unverified
---

CasaOrfeo, 2026-09-28. Nutzer meldete: alle Lampen tauchen zusätzlich im Raum "Wohnzimmer" auf, ohne aus ihren echten Räumen zu verschwinden.

Diagnose ergab ein deutlich größeres Problem als gemeldet: eine `smartthings`-Integration (Config-Entry-Titel "Casa Orfeu", angelegt erst am Vortag, 2026-09-27 06:40 Uhr) hatte 95 Entities über 59 Geräte importiert - praktisch das komplette Hausinventar ein zweites Mal: alle Hue-Lampen, alle Tuya-Türkontakte inkl. Batterien, beide Samsung-The-Frame-TVs samt Sensoren, alle SwitchBot-Steckdosen, Hub Minis, Thermometer, Ventilatoren, Luftreiniger. HA hatte die Namenskollisionen selbst erkannt und Duplikate mit `_2`-Suffix versehen (z.B. binary_sensor.terrasse_tur_2) - Beweis, dass es dieselben physischen Geräte waren, keine neue Hardware. Vermutlich eine versehentliche/testweise Verknüpfung (z.B. beim Einrichten eines Samsung-Geräts, das automatisch SmartThings + "Works with SmartThings"-Cloud-Links zu Hue/Tuya/SwitchBot vorschlägt).

Ursache der "alles im Wohnzimmer"-Symptomatik: SmartThings hatte beim Import fast jedem Gerät pauschal area_id=wohnzimmer zugewiesen, unabhängig vom echten Standort (daher z.B. light.wohnzimmer_neon_kuche für eine Küchenlampe).

Fix: komplette Integration entfernt (Container gestoppt, core.config_entries/core.device_registry/core.entity_registry direkt editiert, Backups als .bak-smartthings-removal-<timestamp>, gleiches Muster wie bei früheren Registry-Edits in diesem Projekt).

**Wichtiger technischer Stolperstein für künftige Registry-Edits:** Ein Device referenziert seinen Config-Entry über das Feld `config_entry_id` (bzw. `primary_config_entry`), NICHT über ein `config_entries`-Listenfeld - ein erster Anlauf mit dem falschen Feldnamen fand 0 zu entfernende Geräte (devices: 182->182), obwohl Entities (909->814, korrekt -95) und der Config-Entry selbst schon korrekt entfernt waren. Nach Korrektur des Feldnamens: 59 Geräte sauber entfernt (182->123), keine verwaisten device_id-Referenzen in der Entity-Registry zurückgeblieben (verifiziert). HA-Neustart fehlerfrei, light.wohnzimmer_* zeigt danach nur noch die 4 echten Wohnzimmer-Lampen.

Offen: ob der SmartThings-Account noch mit Hue/Tuya/SwitchBot verknüpft ist (Samsung-Account-Ebene, außerhalb von HA) - falls ja, könnte ein erneutes Hinzufügen der HA-Integration das Problem wiederholen.