---
created: '2026-09-20T13:36:36.132+02:00'
extra:
  entities:
  - Ein Live
  - HA Core
  - KEINEM Purge
  - Live
  - Casa
  - Fehleinsch
  - Session
  - Fehler
  - Leseartefakt
  - Minuten
  - Neustart
  - Korruption
  - Recorder
  - Start
  - Selbstheilungs
  - Core
  - Historie
  - Zeilen
  - Stand
  - Energie
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
id: 552683830d0a4bd0b6999dbd6414fdbe
normalized_hash: 42a8291998fd861e
tags:
- casaorfeo
- homeassistant
- sqlite
- recorder
- database-corruption
- virtiofs
- project:casaorfeo
title: 'CasaOrfeo: home-assistant_v2.db real korrumpiert nach Live-Lesezugriff, per
  sqlite3 .recover fast vollständig gerettet'
type: bug
updated: '2026-09-20T13:36:36.132+02:00'
valid_at: '2026-09-20T13:36:36.132+02:00'
verification_state: unverified
---

CasaOrfeo, 2026-09-20. ⚠️ Korrigiert eine frühere Fehleinschätzung in derselben Session.

**Ablauf:** Ein Live-Lesezugriff (`sqlite3` im 5s-Takt gegen die laufende `home-assistant_v2.db` über den virtiofs-Bind-Mount) endete mit `database disk image is malformed`. Direkt danach wirkte HA unauffällig (lief weiter, keine Fehler im Log, keine neue `.corrupt.*`-Datei) — deshalb wurde das ursprünglich als reines Leseartefakt abgetan.

**~55 Minuten später, bei einem Neustart (für einen unabhängigen Fix), zeigte sich: die Korruption war real und massiv.** HAs eigener Recorder-Prozess scheiterte wiederholt beim Start mit `sqlite3.DatabaseError: database disk image is malformed` → `Recorder._run threw unexpected exception, recorder shutting down`. Der sonst übliche Selbstheilungs-Mechanismus (Auto-Move nach `.corrupt.<timestamp>` + frische DB, wie am 5. August dokumentiert) sprang diesmal NICHT an — Recorder blieb tot, HA Core lief aber weiter (HTTP 200), nur ohne Historie/Statistiken.

**Diagnose:** Container gestoppt, Kopie gezogen, `PRAGMA integrity_check` zeigte massenhaft `btreeInitPage() returns error code 11` über mehrere B-Tree-Tabellen (states/events/state_attributes-Bereich).

**Recovery, hat sehr gut funktioniert:**
```
sqlite3 kopie.db ".recover" > recovered.sql   # sqlite3 >=3.29, hier 3.54.0
sqlite3 recovered.db < recovered.sql
sqlite3 recovered.db "PRAGMA integrity_check;"  # -> ok
```
Ergebnis: `states` 174.973 Zeilen (sogar leicht über dem letzten bekannt-guten Stand, 10-Tage-Purge normal), `statistics` (Langzeitwerte fürs Energie-Dashboard etc., unterliegt KEINEM Purge) durchgängig 05.07.–20.09. intakt, `integrity_check` → `ok`. `lost_and_found`-Tabelle mit 69k Zeilen entstand, aber nur mit 2 generischen Spalten (`c0`,`c1`) — vermutlich Reste kaputter Index-Bäume, nicht echte verlorene Nutzdaten, da die typisierten Tabellen bereits vollständig wirkten.

Deutlich besseres Ergebnis als der 5.-August-Präzedenzfall (dort kompletter Datenverlust vor dem Vorfall, weil HA einfach neu angefangen hat statt zu reparieren).

**Deployment:** kaputte Original-DB als `.corrupt.<timestamp>` beiseite gelegt (nie gelöscht), `recovered.db` als neue `home-assistant_v2.db` eingesetzt (Owner/Rechte von der alten Datei übernommen), Container gestartet — sauber, keine Fehler mehr, WAL wächst normal.

**Lehre:** Der zeitliche Zusammenhang mit dem Live-Zugriff ist naheliegend, aber nicht zweifelsfrei bewiesen (dazwischen lagen auch mehrere reguläre Container-Neustarts). Wichtiger als die Kausalität: "HA zeigt gerade keinen Fehler nach einem Live-DB-Zugriff" ist KEIN verlässliches Signal für Unversehrtheit — echte Korruption zeigt sich oft erst beim nächsten Neustart, wenn der Recorder die Datei komplett neu einliest. Siehe [[db-nicht-im-betrieb-direkt-lesen]] (lokale Memory-Datei, dort ebenfalls korrigiert) und [[759511c3]] (Colima/virtiofs-Kontext).