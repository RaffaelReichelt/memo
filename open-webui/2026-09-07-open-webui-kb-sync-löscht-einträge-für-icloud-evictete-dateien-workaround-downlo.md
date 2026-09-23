---
created: '2026-09-07T08:26:37.941+02:00'
extra:
  entities:
  - Open WebUI KB Sync
  - Download Now
  - Open WebUI Knowledge Base
  - Die Browser
  - File System Access API
  - Sync
  - Eintr
  - Dateien
  - Knowledge
  - Base
  - Drag
  - Lauf
  - Verzeichnis
  - Drive
  - Cloud
  - Browser
  - System
  - Access
  - Ordner
  - Platzhalter
  owner_principal: f0bd05cb3225
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
id: 6931d7d9aee6400e988c8fdce1c68a33
normalized_hash: 18e1f7d01632f676
tags:
- open-webui
- knowledge-base
- icloud
- sync
- data-loss
- project:open-webui
title: 'Open WebUI KB Sync löscht Einträge für iCloud-evictete Dateien (Workaround:
  Download Now vor Sync)'
type: bug
updated: '2026-09-07T08:26:37.941+02:00'
valid_at: '2026-09-07T08:26:37.941+02:00'
verification_state: unverified
---

Open WebUI Knowledge Base "Sync"-Feature (Ordner-Update via Add-Content-Menü → "Sync", nicht das reine Drag&Drop — letzteres löscht nie was) löscht bei jedem Lauf alle KB-Einträge, deren (path, filename) im frisch eingelesenen lokalen Verzeichnis-Snapshot fehlt (`sync_knowledge_diff` in [backend/open_webui/routers/knowledge.py:1889-1995](../../../home/raffael/Projekte/open-webui/backend/open_webui/routers/knowledge.py), Frontend-Traversal in `src/lib/components/workspace/Knowledge/KnowledgeBase.svelte` via `collectDirectoryFiles`/`showDirectoryPicker`).

Symptom (2026-09-07, Beispiel `/Users/raffael/Documents/2026/Batz`): wiederholt gehen KB-Einträge für Dateien verloren, die lokal nachweislich noch existieren. Ursache vermutlich **iCloud Drive "on-demand"-Eviction**: Documents liegt in iCloud Drive, iCloud evictet lokal ungenutzte Dateien zu reinen Cloud-Platzhaltern. Die Browser-Directory-APIs (File System Access API `showDirectoryPicker` bzw. der `webkitdirectory`-Fallback), mit denen der Sync den Ordner einliest, listen oder lesen solche Platzhalter teils nicht zuverlässig — die Datei fehlt dann im Snapshot, obwohl sie in Finder ganz normal sichtbar ist, und der Sync-Diff interpretiert das fälschlich als "lokal gelöscht" → entfernt den KB-Eintrag. Erklärt auch, warum es wiederkehrend auftritt (iCloud evictet laufend im Hintergrund neu).

**Workaround (aktuell gewählt, kein Code-Fix):** vor jedem Sync-Lauf den Ordner in Finder markieren → Rechtsklick → "Jetzt laden" ("Download Now"), damit alle Dateien lokal materialisiert sind, bevor der Browser sie einliest.

**Noch nicht umgesetzt:** ein Sicherheitsnetz im Sync-Code (abbrechen/warnen statt still löschen, wenn auffällig viele vorher indexierte Dateien plötzlich fehlen) wurde vorgeschlagen, aber vom User explizit erstmal abgelehnt ("Nein, erstmal nur der Workaround") — bei erneutem Auftreten trotz Workaround dieses Sicherheitsnetz erneut anbieten.