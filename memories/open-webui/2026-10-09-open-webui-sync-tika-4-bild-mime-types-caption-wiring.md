---
created: '2026-10-09T10:43:40.093+02:00'
extra:
  entities:
  - Open WebUI Sync
  - Wiring KB
  - Sync Documents
  - Bei Sync
  - Sync
  - Documents
  - Klartext
  - Bilder
  - Engine
  - Medien
  - Upstream
  - Loader
  - Vision
  - Chat
  - Datenverzeichnis
  - HEIC
  - JPEG
  - Expecting value
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
id: 82438b6714d942148b0eb8a0d72bc6c6
normalized_hash: 4c248cc72ece5f8d
tags:
- open-webui
- tika
- sync
- project:open-webui
title: 'Open WebUI Sync: Tika 4, Bild-Mime-Types, Caption-Wiring'
type: bug
updated: '2026-10-09T10:43:40.093+02:00'
valid_at: '2026-10-09T10:43:40.093+02:00'
verification_state: unverified
---

KB-Sync Documents/2026 (2026-10-09): Dateien hochgeladen, aber nicht indexiert (file.data.status=failed in webui.db, 670 von ~672). Ursachen:
1. Tika-Image lief auf 4.1.0 (latest-full), DB-Config rag.tika_server_version stand auf "3" -> tika/text liefert Klartext, r.json() scheitert ("Expecting value"). Fix: rag.tika_server_version="4" in webui.db (Compose-Env wirkt nicht, DB gewinnt).
2. Upstream v0.11.x lehnt Bilder ab, solange rag.content_extraction.supported_media_mime_types null ist (nur Engine 'external' erlaubt Medien). Gesetzt auf image/jpeg,png,heic,heif.
3. RAG_IMAGE_CAPTION_* Env-Vars kamen nach Upstream-Umbau (Loader-Config nur noch aus DB) nicht mehr im Loader an -> Fork-Fix in loaders/main.py liest os.environ. Modell qwen3-vl:30b existiert nicht, korrekt qwen3-vl:32b. HEIC wird via pillow-heif nach JPEG gewandelt.
Zusatz: ui.default_models auf robit/kolibri-1:q4_k_m gesetzt (kein Vision, nur Chat). DB-Backup: webui.db.bak-20261009 im Datenverzeichnis. Bei Sync-Problemen zuerst file.data.status/error in webui.db pruefen.