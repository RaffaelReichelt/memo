---
created: '2026-09-20T11:55:40.907+02:00'
extra:
  entities:
  - Reload Home Assistant
  - Reload
  - Home
  - Assistant
  - Frontend
  - Casa
  - Sprach
  - Einstellungen
  - Onboarding
  - Nutzer
  - Profil
  - Browser
  - Instanz
  - Fallstrick
  - Server
  - Seitenleiste
  - Daten
  - Panel
  - Profilseite
  - Übersetzungs
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
id: 86589b1c06a8401b82fac1d212f56fba
normalized_hash: cbe1f85e9e8629e3
tags:
- home-assistant
- frontend
- sprache
- casaorfeo
- project:casaorfeo
title: 'HA: Instanz-Sprache vs. Profil-Sprache getrennt + Übersetzungs-Fragment-Cache
  braucht vollen Reload'
type: fact
updated: '2026-09-20T11:55:40.907+02:00'
valid_at: '2026-09-20T11:55:40.907+02:00'
verification_state: unverified
---

Home Assistant (aktuelle Frontend-Version, beobachtet in CasaOrfeo-Projekt, HA 2026.9.3 / home-assistant-frontend 20260826.7): Es gibt zwei getrennte Sprach-Einstellungen, die leicht verwechselt werden. 1) Instanz-Sprache unter Einstellungen > System > Allgemein (gespeichert in .storage/core.config, Feld "language") - betrifft nur Onboarding-Texte, System-Benachrichtigungen, Standard für neue Nutzer. 2) Pro-Benutzer-Sprache im eigenen Profil (Avatar unten links > Allgemein > Sprache) - steuert die tatsächlich angezeigte Frontend-UI-Sprache für den eingeloggten Nutzer, Standard ist "Automatisch erkennen" (folgt der Browser-Sprache), unabhängig von der Instanz-Einstellung.

Zusätzlicher Fallstrick, verifiziert im Frontend-Package (hass_frontend/static/translations/profile/de-*.json war vollständig und korrekt übersetzt, also kein Server-/Übersetzungslücken-Problem): Nach Ändern der Profil-Sprache aktualisiert sich die Seitenleiste sofort reaktiv (aus bereits geladenen Daten), aber Panel-Inhalte wie die Profilseite selbst laden ihre Übersetzungs-Fragmente offenbar nur einmal pro Sitzung und cachen das - bleiben dadurch auf der alten Sprache, bis ein kompletter Seiten-Reload (F5/Cmd+R, nicht nur zurücknavigieren) die Fragmente neu lädt. Nutzer bestätigt: Reload hat das Problem vollständig gelöst.