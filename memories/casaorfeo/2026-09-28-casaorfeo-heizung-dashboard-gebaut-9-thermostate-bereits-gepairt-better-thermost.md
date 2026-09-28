---
created: '2026-09-28T06:19:37.075+02:00'
extra:
  entities:
  - Neues Dashboard
  - Zwei Funde
  - Der CLAUDE
  - In Zigbee
  - Better Thermostat
  - Im Dashboard
  - Thermostate
  - Casa
  - Dashboard
  - Zigbee
  - Thermostat
  - Raum
  - Funde
  - Stand
  - Klarnamen
  - Schlafzimmer
  - Innenfühler
  - Regel
  - Entity
  - Regler
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
id: 828fc916cd9a4c629442759ad2c15178
normalized_hash: b3fe2770da8acc85
tags:
- casaorfeo
- homeassistant
- zigbee2mqtt
- thermostat
- dashboard
- hardware-inventar
- project:casaorfeo
title: 'CasaOrfeo: Heizung-Dashboard gebaut, 9 Thermostate bereits gepairt, Better-Thermostat-Integration
  entdeckt'
type: fact
updated: '2026-09-28T06:19:37.075+02:00'
valid_at: '2026-09-28T06:19:37.075+02:00'
verification_state: unverified
---

CasaOrfeo, 2026-09-26. Neues Dashboard "Heizung" (dashboards/heizung_dashboard.yaml) für alle 9 Zigbee-Heizkörperthermostate erstellt: Batterieübersicht, 3x3-Grid mit Thermostat-Karten pro Raum, Fenster-/Kindersicherungs-Status.

**Zwei Funde beim Bau, die den bisherigen Stand korrigieren:**
1. Der CLAUDE.md-Fahrplan-Punkt "9 Thermostate pairen und kalibrieren" war veraltet - alle 9 sind bereits gepairt (per core.device_registry/core.entity_registry verifiziert). In Zigbee2MQTT selbst tragen alle 9 aber weiterhin nur ihre rohe IEEE-Adresse als friendly_name - die Klarnamen/Raumzuordnung (Studio, Küche, Esszimmer, Klavierzimmer x2, Badezimmer, Wohnzimmer x2, Schlafzimmer) leben ausschließlich im HA-Device-Registry, nicht in Z2M. Kalibrierungsstand (Ventilstellung/Temperatur-Offset) bleibt weiterhin ungeprüft/offen.
2. Bislang nirgends dokumentiert: Für das Schlafzimmer läuft zusätzlich die HACS-Integration "Better Thermostat" (climate.temperatur_schlafzimmer, config_entry seit 2026-09-21), die den rohen TRV climate.0x7cc6b6fffe0bbb3b wrapped und dessen eigenen Innenfühler durch sensor.wetterstation_indoor_temperature/_humidity (Tuya-Wetterstation) ersetzt, plus sensor.innen_aussen_temperatur und weather.forecast_home_2 für die Regel-Logik. Im Dashboard bewusst NUR die virtuelle Entity gezeigt, nicht zusätzlich der rohe TRV (sonst zwei unabhängige Regler auf demselben Ventil).

Dashboard in configuration.yaml unter lovelace.dashboards registriert (Key "heizung-details", Bindestrich-Pflicht siehe bekannter Lovelace-YAML-Fallstrick), HA-Neustart fehlerfrei. Alles committed, Details in CLAUDE.md.