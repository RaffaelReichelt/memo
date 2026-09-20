---
created: '2026-09-20T11:55:56.567+02:00'
extra:
  entities:
  - OBI Energy Tracker Backend
  - GitHub Issue
  - Integration Karo
  - OBI Energy Tracker
  - Custom Component
  - Kein Maintainer
  - Energy
  - Tracker
  - Backend
  - Issue
  - Cloud
  - Karo
  - Custom
  - Component
  - Response
  - Auth
  - Melder
  - Integration
  - Issues
  - Passwort
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
id: c0c1a45e4e0b4edca98d36c8a6583814
normalized_hash: c77e08ced02b3489
tags:
- home-assistant
- casaorfeo
- obi
- energie
- hacs
- offene-entscheidung
- project:casaorfeo
title: 'CasaOrfeo: OBI Energy Tracker Backend-API vermutlich geändert (GitHub Issue
  #28, 404 von CloudFront)'
type: fact
updated: '2026-09-20T11:55:56.567+02:00'
valid_at: '2026-09-20T11:55:56.567+02:00'
verification_state: unverified
---

CasaOrfeo: Am 2026-09-19/20 im GitHub-Repo der HACS-Integration Karo-X/obi_energy (die für den OBI Energy Tracker verwendete Custom Component) Issue #28 gefunden: Login schlägt mit HTTP 404 von CloudFront fehl statt mit dem erwarteten 401 (Log: "ObiConnectionError: Login request failed with HTTP 404", "X-Cache: Error from cloudfront", leerer Response-Body). Deutet auf einen entfernten/verschobenen Auth-Endpunkt beim OBI/heyOBI-Backend hin. Wichtig: die offizielle heyOBI-App selbst funktioniert laut Melder weiterhin normal - betrifft nur den inoffiziellen/reverse-engineerten API-Zugriff der Integration. Zwei weitere frische Issues (#25, #26, 17./18.09.) zu erzwungenen Passwort-Resets ohne 2FA, möglicherweise verwandt. Kein Maintainer-Fix zum Zeitpunkt der Prüfung, auch der aktivere Fork urfin78/obi_energy hatte in seinen Commits vom 18.09. keinen Fix dafür.

Kontext: Nutzer wollte am 2026-09-20 den Stromzähler von bitShake (Stromversorgungsproblem der Hardware) zurück auf OBI Energy Tracker umstellen. Dashboards/Energy-Storage wurden bereits zurückgestellt, die eigentliche OBI-Integration (custom_components/obi_energy + Config-Entry, am 2026-08-10 komplett deinstalliert) aber noch NICHT neu installiert/eingeloggt, weil ein Reinstall aktuell vermutlich genau in diesen Login-404 laufen würde. Nutzer wollte darüber nochmal nachdenken, Entscheidung offen zum Zeitpunkt dieser Notiz - vor einem Reinstall-Versuch prüfen, ob Issue #28 inzwischen gefixt ist.