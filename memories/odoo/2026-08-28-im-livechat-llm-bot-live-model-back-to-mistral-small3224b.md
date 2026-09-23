---
created: '2026-08-28T15:17:39.003+02:00'
extra:
  entities:
  - Kundenservice Bot
  - Private
  - Ollama
  - PrivateMind
  - GX10
  - FAIL
  - PREIS
  - mistral-small3.2:24b
  - gemma4:12b
  - model_benchmark.py
  - what's on your pricing page
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
id: 94a08a4a1684411ba61b466e883e82be
normalized_hash: 7812fdf8a2c74f3b
review_after: '2027-02-24T15:17:39.003000+02:00'
tags:
- privatemind-odoo
- im_livechat_llm_bot
- model-selection
- ollama
- project:odoo
title: 'im_livechat_llm_bot live model: back to mistral-small3.2:24b'
type: decision
updated: '2026-08-28T15:17:39.003+02:00'
valid_at: '2026-08-28T15:17:39.003+02:00'
verification_state: unverified
---

PrivateMind/odoo im_livechat_llm_bot "Kundenservice Bot" assistant: live production model as of 2026-08-26 is back to `mistral-small3.2:24b` (GX10 Ollama provider). It had been switched to `gemma4:12b` earlier the same session; a live full 46-question `model_benchmark.py` run actually scored gemma4:12b numerically better than any historical mistral-small3.2:24b LoRA round (OK=23/FAIL=22, Ø7.2s/question), but real user testing found gemma4:12b's non-price answers frequently off-topic/evasive, and the user asked to switch back to mistral-small3.2:24b. Lesson: the automated model_benchmark.py heuristic-check scores don't fully capture real conversational quality/relevance — don't treat a higher benchmark OK-count alone as sufficient justification to keep a model swap.

Also: known remaining gap (not yet fixed) — the very generic "what's on your pricing page" question (PREIS-1 in model_benchmark.py) still sometimes routes to knowledge_retriever instead of faq_price_lookup and comes back without any price. Safe (no wrong number shown) but unhelpful; deliberately not covered by the new faq_price_lookup blind-override path since that intent (price_overview) is the one most prone to false-positive triggering.