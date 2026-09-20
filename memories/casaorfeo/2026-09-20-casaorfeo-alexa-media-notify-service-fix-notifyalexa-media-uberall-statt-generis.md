---
created: '2026-09-20T11:55:35.148+02:00'
extra:
  entities:
  - Alexa Drop
  - Alexas Multiroom
  - Generelles Muster
  - Drop
  - Türkontakt
  - Aktion
  - Quellcode
  - Service
  - Multiroom
  - Entity
  - Config
  - Muster
  - Projekt
  - CasaOrfeo
  - JEDES
  - 'action: notify.alexa_media'
  - 'target: entity_id: media_player.uberall'
  - Alexa Drop-In bei Türkontakt
  - 'unbekannte Aktion: notify.alexa_media'
  - Überall
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
id: a9ba67d866254460acf0e4a97f7696c1
normalized_hash: 32909c53a259f4bd
tags:
- home-assistant
- casaorfeo
- alexa_media
- automation
- notify
- project:casaorfeo
title: 'CasaOrfeo: alexa_media notify-Service-Fix (notify.alexa_media_uberall statt
  generisches notify.alexa_media+target)'
type: bug
updated: '2026-09-20T11:55:35.148+02:00'
valid_at: '2026-09-20T11:55:35.148+02:00'
verification_state: unverified
---

CasaOrfeo: Automation "Alexa Drop-In bei Türkontakt" (automation.alexa_drop_in_bei_turkontakt) rief `action: notify.alexa_media` mit `target: entity_id: media_player.uberall` auf - HA meldete "unbekannte Aktion: notify.alexa_media". Ursache (per Quellcode-Analyse von custom_components/alexa_media/notify.py, targets-Property verifiziert): die alexa_media_player-Integration unterstützt kein generisches notify.alexa_media mit target:/entity_id-Targeting. Sie registriert stattdessen für JEDES ihrer media_player-Entities einen eigenen Service notify.alexa_media_<entity_name> (Legacy-Notify-Pattern, keine eigenen notify-Entities). media_player.uberall (Alexas Multiroom-Gruppe "Überall") gehört laut Entity-Registry selbst zur alexa_media-Plattform, der korrekte Service ist daher notify.alexa_media_uberall (kein target: nötig). Fix am 2026-09-20 in automations.yaml angewendet, HA-Neustart ohne Config-Fehler bestätigt.

Generelles Muster für dieses Projekt: jeder alexa_media-Drop-In/Notify-Aufruf muss den geräte-/gruppenspezifischen Service notify.alexa_media_<slug> nutzen, nie einen generischen notify.alexa_media mit target.