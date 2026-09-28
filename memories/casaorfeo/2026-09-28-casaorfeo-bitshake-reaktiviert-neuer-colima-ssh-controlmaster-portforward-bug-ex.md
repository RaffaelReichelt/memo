---
created: '2026-09-28T06:19:23.063+02:00'
extra:
  entities:
  - Colima SSH
  - Beim Verifizieren
  - Das Ger
  - UND Host
  - Colima
  - Tasmota
  - Betrieb
  - Sensor
  - Setup
  - Verifizieren
  - Mqtt
  - Fund
  - Mosquitto
  - Container
  - Host
  - Port
  - Ebene
  - Fehlschlag
  - Stack
  - Launch
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
id: 9513bd86530c49cd85d5643ee111222b
normalized_hash: 853677afeb833907
tags:
- casaorfeo
- homeassistant
- colima
- docker
- mqtt
- bitshake
- stromzaehler
- project:casaorfeo
title: 'CasaOrfeo: bitShake reaktiviert + neuer Colima SSH-ControlMaster-Portforward-Bug
  (exit 255)'
type: bug
updated: '2026-09-28T06:19:23.063+02:00'
valid_at: '2026-09-28T06:19:23.063+02:00'
verification_state: unverified
---

CasaOrfeo, 2026-09-26. bitShake Tasmota-Stromzähler wieder in Betrieb genommen (IP 192.168.2.225, jetzt dauerhaft/reserviert). Die fünf rohen mqtt: sensor:-Definitionen (Gesamt/Hochtarif/Niedertarif/Leistung/Einspeisung) wurden in configuration.yaml ergänzt, Feldpfade live vom Gerät bestätigt (`curl http://192.168.2.225/cm?cmnd=Status%2010` -> MT174: {ImportActive, ExportActive, Power, powerOut, E_inHT, E_inNT, Meter_number} - powerOut war bisher unbekannt/undokumentiert). unique_id je Sensor identisch zu den verwaisten core.entity_registry-Einträgen aus dem ursprünglichen Setup, dockt automatisch wieder an sensor.stromzaehler_* an.

Beim Verifizieren gefunden: Das Gerät hatte MqttHost nie gesetzt (StatusMQT.MqttHost: "") - deshalb sah es wochenlang "offline" aus. Per `cmnd=MqttHost 192.168.2.54` gesetzt + Geräte-Neustart.

**Größerer, bisher undokumentierter Fund:** Danach war Mosquitto-Port 1883 trotz laufendem Container und korrektem docker-compose-Mapping von außen (Host-localhost UND Host-LAN-IP) nicht erreichbar, während Port 8123 parallel einwandfrei lief. Weder docker restart noch docker stop/start noch --force-recreate des mosquitto-Containers halfen. Ursache laut ~/.colima/_lima/colima/ha.stderr.log: jeder neue SSH-ControlMaster-Forward-Versuch für Port 1883 (und 8099) scheiterte mit exit status 255 - eine Ebene tiefer als das bereits bekannte "Forward nach Fehlschlag nicht wiederholt"-Muster, das sich mit einem Container-Neustart beheben lässt. Nur ein `colima restart` behob es endgültig - dabei brach der komplette Stack (HA/Z2M/Mosquitto/Tailscale) kurz zusammen (Colima selbst konnte wegen der kaputten SSH-Session nicht sauber stoppen/starten), wurde aber vom LaunchAgent-Watchdog (com.casaorfeo.colima.plist / colima-watchdog.sh) automatisch abgefangen und neu gestartet, ohne manuellen Eingriff.

Dashboard energie_details_dashboard.yaml um eine "Zählerstände"-Karte (Gesamt/Hochtarif/Niedertarif/Einspeisung) ergänzt. Alles committed, Details in CLAUDE.md.