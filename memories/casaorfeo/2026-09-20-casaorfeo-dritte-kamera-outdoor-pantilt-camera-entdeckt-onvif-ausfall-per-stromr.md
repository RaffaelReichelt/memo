---
created: '2026-09-20T11:55:31.206+02:00'
extra:
  entities:
  - Outdoor Pan
  - Tilt Camera
  - SwitchBot Pan
  - Tilt Cam
  - Gleiches Muster
  - Kamera Dresden
  - In CLAUDE
  - Kamera
  - Camera
  - Stromreset
  - Casa
  - Debuggen
  - Switch
  - Wannsee
  - Sicherheitsdashboard
  - Entity
  - Neustart
  - Entities
  - Muster
  - Imou
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
id: 1f34e741b7e045a28e30f2cce5e497df
normalized_hash: 59c67b02383d958e
tags:
- home-assistant
- casaorfeo
- kamera
- onvif
- switchbot
- hardware-inventar
- project:casaorfeo
title: 'CasaOrfeo: dritte Kamera "Outdoor Pan/Tilt Camera" entdeckt, ONVIF-Ausfall
  per Stromreset behoben'
type: fact
updated: '2026-09-20T11:55:31.206+02:00'
valid_at: '2026-09-20T11:55:31.206+02:00'
verification_state: unverified
---

CasaOrfeo: Am 2026-09-20 beim Debuggen einer fehlenden Kamera-Vorschau eine dritte, bis dahin komplett undokumentierte Kamera entdeckt - "Outdoor Pan/Tilt Camera" (IP 192.168.2.48, ONVIF-Config-Entry-Unique-ID b0:e9:fe:f5:6b:af, Port 2020 für ONVIF, Port 554 RTSP), eine zweite SwitchBot Pan/Tilt Cam 3K neben der bereits in CLAUDE.md dokumentierten Wannsee-Kamera (192.168.1.53). War weder im CLAUDE.md-Geräteinventar noch sonst dokumentiert, tauchte aber im Sicherheitsdashboard (camera.switchbot_outdoor_pan_tilt_cam_3k_secstream) und in der Entity-Registry auf.

Symptom war: HA-Fehlerdialog "secStream wird nicht mehr von der onvif-Integration bereitgestellt". Ursache: ONVIF-Dienst der Kamera (Port 2020) war unerreichbar (Connection refused), nur RTSP (554) lief noch - HAs onvif-Integration konnte deshalb bei keinem Neustart mehr Entities registrieren, alte Entity galt als verwaist. Gleiches Muster wie bei der Imou-Kamera Dresden 2026-08-15/16 (Firmware bleibt teilweise hängen, RTSP läuft noch, ONVIF/HTTP nicht mehr). Fix: physischer Stromreset der Kamera (nicht nur Software-Reboot über die App) - hat funktioniert, vom Nutzer bestätigt.

In CLAUDE.md ist diese Kamera weiterhin nicht im Geräteinventar ergänzt - sollte bei Gelegenheit nachgeholt werden.