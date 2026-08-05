---
created: '2026-07-29T13:25:31.512+02:00'
extra:
  entities:
  - Apple Silicon
  - Apples GPU
  - ASUS Ascent GX
  - Hugging Face
  - NVIDIA CUDA
  - Apple Metal
  - Unified Memory
  - 'PrivateMind


    Für'
  - NVIDIA GB
  - Ein Modell
  - Apple
  - Array
  - Lernen
  - Grund
  - Silicon
  - Metal
  - Unified
  - Speicher
  - Prinzip
  - Ascent
id: 7fd66b05dc6843c3902a7fff47c1ee18
normalized_hash: 60471c53d142a5f2
tags:
- mlx
- llm
- hardware
- project:privatemind
title: 'MLX: Apple-Framework, was es ist und warum irrelevant für die GX10-Appliance'
type: note
updated: '2026-07-29T13:25:31.512+02:00'
verification_state: unverified
---

# Memo: MLX

**Stand:** Juli 2026

## Was ist MLX?

MLX ist ein von Apple entwickeltes, quelloffenes Array-/Tensor-Framework für maschinelles Lernen — konzeptionell vergleichbar mit NumPy/PyTorch, aber von Grund auf für Apple Silicon (M1–M4) gebaut. Es spricht direkt Metal (Apples GPU-API) an und ist auf die Unified-Memory-Architektur der M-Chips zugeschnitten, bei der sich CPU und GPU denselben Speicher teilen — ähnlich dem Prinzip, das auch die NVIDIA-GB10-Chips im ASUS Ascent GX10 verwenden, nur eben Apple-eigen und nicht kompatibel dazu.

## MLX vs. GGUF (llama.cpp)

Wenn ein Modell auf Ollama oder Hugging Face sowohl als "normale" (GGUF) und als "MLX"-Variante angeboten wird, handelt es sich um zwei unterschiedliche Compile-/Quantisierungs-Pfade für dieselben Gewichte:

| | GGUF (llama.cpp) | MLX |
|---|---|---|
| Zielplattform | Plattformneutral (CPU, NVIDIA CUDA, Apple Metal, Vulkan …) | Ausschließlich Apple Silicon |
| Optimierung | Allgemein, breite Kompatibilität | Direkt auf Metal/Unified Memory zugeschnitten |
| Geschwindigkeit auf Mac | Gut | Meist spürbar schneller, da nativ statt über eine Kompatibilitätsschicht |
| Geschwindigkeit auf GX10/NVIDIA | Läuft (via llama.cpp/CUDA) | **Läuft nicht** — MLX gibt es nur für Apple Silicon |

## Relevanz für PrivateMind

Für die Appliance (NVIDIA GB10) ist MLX **nicht relevant** — dort kommt GGUF (llama.cpp) oder ein natives NVIDIA-Format (TensorRT-LLM) zum Einsatz. MLX betrifft ausschließlich lokales Testen/Entwickeln auf einem Mac. Ein Modell, das man auf dem Mac per MLX ausprobiert, muss für den Appliance-Betrieb separat als GGUF- oder TensorRT-Build vorliegen.

## Bezugsquellen

- Hugging Face: Community-Sammlung unter `mlx-community/…`
- Ollama: eigene Tags, z. B. `qwen3.6:35b-mlx` (Namensgebung je nach Modell-Repo leicht unterschiedlich)