---
created: '2026-10-14T09:56:23.203+02:00'
extra:
  entities:
  - Docker Model Runner
  - Docker Hub
  - Model
  - Runner
  - Ollama
  - Raffael
  - Alternative
  - Gespr
  - Registry
  - Host
  - Swarm
  - Compose
  - Plugin
  - Gewinn
  - Rollout
  - Experiment
  - Umzug
  - Modelle
  - Gelegenheit
  - TODO
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
id: 3dceca91b7a5493e9075fc1712f52700
normalized_hash: f9ba849241c11d56
tags:
- todo
- ollama
- docker-model-runner
- rollout
- project:ollama
title: 'TODO: Docker Model Runner (DMR) als Ollama-Alternative ausprobieren'
type: note
updated: '2026-10-14T09:56:23.203+02:00'
valid_at: '2026-10-14T09:56:23.203+02:00'
verification_state: unverified
---

Raffael möchte gelegentlich daran erinnert werden (Stand 2026-10-14), Docker Model Runner (DMR) als mögliche Alternative zu Ollama auszuprobieren.

Hintergrund/Einschätzung aus dem Gespräch:
- Modelle sind OCI-Artefakte (Docker Hub ai/ oder eigene Registry), Format überwiegend GGUF (llama.cpp), später vLLM/Safetensors, MLX auf dem Mac.
- Läuft als Host-Prozess/Plugin (Port 12434, OpenAI-kompatibel), NICHT als Swarm-Service; `models:` in Compose funktioniert mit `docker compose`, nicht mit `docker stack deploy`.
- Auf dem Host (gx10, aarch64, Docker 29.6.2) ist das Plugin nicht installiert (`docker model` unbekannt).
- Stack würde NICHT schlanker (Sidecar, Modellverwaltung bleiben); möglicher Gewinn: Modelle in eigener Registry (registry-Stack) für den Rollout.
- Erst als Experiment prüfen, NACH dem Umzug der Ollama-Datenpfade nach /var/lib/ollama und /opt/ollama/serve.json.
- Zu testen: llama.cpp auf GB10/aarch64 (Ollama meldet "compute capability not in compiled architectures"), Verfügbarkeit der Modelle (kolibri, mistral-small-4, qwen3-vl), Ersatz für keep_alive/MAX_LOADED_MODELS.

Erinnerung: bei passender Gelegenheit in Ollama-/Rollout-Sessions einmal darauf hinweisen.