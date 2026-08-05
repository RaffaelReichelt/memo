---
created: '2026-07-29T13:25:40.856+02:00'
extra:
  entities:
  - Der Preis
  - Die Stufen
  - Guter Mittelweg
  - Beispiel Qwen
  - Wann Qualit
  - Tool Calling
  - Lange Reasoning
  - Exakte Zahlen
  - 'PrivateMind


    Auf'
  - GB Unified Memory
  - Mistral Small
  - GB Speicher
  - Private
  - Quantisierung
  - Byte
  - Parameter
  - Anzahl
  - Bits
  - Gewicht
  - Speicherbedarf
id: 0c2d6d58d1574aa38e519b0da2e642b0
normalized_hash: 2e23f7bf9870ed24
tags:
- quantisierung
- llm
- gguf
- project:privatemind
title: 'Quantisierung: Q8 vs. Q4/Q5, GGUF-K-Quants und PrivateMind-Empfehlung'
type: note
updated: '2026-07-29T13:25:40.856+02:00'
verification_state: unverified
---

# Memo: Quantisierung (Q8, Q4/Q5)

**Stand:** Juli 2026

## Was ist Quantisierung?

LLM-Gewichte werden ursprünglich in 16-Bit-Gleitkommazahlen trainiert und gespeichert (FP16/BF16 = 2 Byte pro Parameter). Quantisierung reduziert die Anzahl der Bits pro Gewicht — kleinerer Speicherbedarf, weniger Daten, die durch den Speicherbus müssen. Das ist bei LLM-Inferenz der entscheidende Hebel, weil meist die Speicherbandbreite der Flaschenhals ist, nicht die reine Rechenleistung. Der Preis: Präzisionsverlust, der sich je nach Stärke der Quantisierung mehr oder weniger stark auf die Modellqualität auswirkt.

## Die Stufen

| Stufe | Byte/Parameter (ca.) | Speicherbedarf ggü. FP16 | Qualität |
|---|---|---|---|
| BF16/FP16 (Original) | 2 | 100 % | Referenz, volle Qualität |
| **Q8** | ~1 | ~50 % | Praktisch verlustfrei |
| **Q5** | ~0,6 | ~30 % | Guter Mittelweg |
| **Q4** | ~0,5 | ~25 % | Spürbarer, meist akzeptabler Verlust |

**Faustregel für Speicherbedarf:** Parameter × Byte/Parameter aus der Tabelle. Beispiel Qwen3.6-27B: 27 Mrd. × 1 Byte ≈ 27 GB bei Q8.

## GGUF-Feinheiten (llama.cpp)

Innerhalb von "Q4"/"Q5" gibt es je nach Runtime nochmal Unterstufen:

- **`Q4_0` / `Q5_0` / `Q8_0`:** einfaches Schema, ein Skalierungsfaktor pro Gewichtsblock.
- **`Q4_K_M` / `Q5_K_M` (K-Quants):** neueres, intelligenteres Schema — wichtigere Gewichte im Block bekommen etwas mehr Bits, unwichtigere weniger. Bei gleicher Nominalgröße spürbar bessere Qualität als die `_0`-Variante. `Q4_K_M` ist in der Praxis meist die Standardempfehlung, wenn man auf 4-Bit gehen muss.
- Suffixe `_S` / `_M` / `_L` (Small/Medium/Large) sind Feinabstufungen innerhalb der K-Quants, die etwas mehr Größe gegen etwas mehr Qualität tauschen.

## Wann Qualitätsverlust besonders wehtut

- Tool Calling / strukturierte Ausgaben (JSON-Aufrufe für Agenten wie nanobot) — ein einzelnes falsches Zeichen kann den ganzen Aufruf brechen.
- Lange Reasoning-Ketten — Fehler akkumulieren über mehrere Schritte.
- Exakte Zahlen-/Codegenauigkeit.
- Seltenes, spezifisches Faktenwissen.

## Empfehlung für PrivateMind

Auf dem GX10 (128–256 GB Unified Memory) ist meist genug Speicher vorhanden, um nicht auf Q4 herunter zu müssen. Deshalb ist **Q8 der Standard** für die Kernmodelle in der Appliance (Mistral Small/Large, Qwen3.6-27B, Llama 4 Scout) — Tool-Calling-Zuverlässigkeit ist wichtiger als der letzte GB Speicher gespart. Q4/Q5 bleibt Fallback für Speicherengpässe (z. B. Parallelbetrieb mehrerer Modelle) oder kleinere Hardware-Tiers, wo die Qualitätseinbuße bewusst gegen Machbarkeit getauscht wird.