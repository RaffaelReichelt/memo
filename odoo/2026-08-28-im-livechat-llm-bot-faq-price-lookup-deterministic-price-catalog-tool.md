---
created: '2026-08-28T15:17:25.064+02:00'
extra:
  entities:
  - Kundenservice Prompt
  - In Bundle Standard
  - Odoo
  - Prompt
  - Bundle
  - Standard
  - Monate
  - PrivateMind
  - LLM
  - EUR
  - VAT
  - NOT
  - WRONG
  - FIRST
  - PREISUEBERSICHT
  - EXACT
  - ANY
  - addons/18.0/im_livechat_llm_bot/
  - faq_price_lookup
  - data/faq_catalog.json
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
id: 8be72e480fa344e99e1486fbca0d80b9
normalized_hash: 2e5757321ebf3d86
tags:
- privatemind-odoo
- im_livechat_llm_bot
- llm-tool-calling
- price-mangling
- ollama
- project:odoo
title: 'im_livechat_llm_bot: faq_price_lookup deterministic price-catalog tool'
type: fact
updated: '2026-08-28T15:17:25.064+02:00'
valid_at: '2026-08-28T15:17:25.064+02:00'
verification_state: unverified
---

In `addons/18.0/im_livechat_llm_bot/` (PrivateMind/odoo repo), a new deterministic `faq_price_lookup` tool was built to fix chronic LLM price-transcription errors (e.g. "3800 EUR" → "3.80 EUR"), which a prior LoRA fine-tuning investigation had already concluded was an inherent model quirk, not fixable via prompting/training data.

Architecture:
- `data/faq_catalog.json`: curated price Q&A entries (hardware/service-plan/bundle tiers, VAT, discounts, Starter-Paket combo, ambiguous "Enterprise") with pre-written canonical `answer` text per intent.
- `models/llm_tool.py`: `faq_price_lookup` tool (auto-registers via `@llm_tool` decorator on Odoo restart). `_classify_faq_query()` does keyword-based classification (product tier × category), deliberately NOT embeddings/fuzzy-matching — a fuzzy-similarity prototype scored 80% but its 20% misses were often confidently WRONG (dangerous); the keyword approach trades recall for near-zero dangerous-wrong-match rate (~84% correct / ~9% safe "no match" / ~1% wrong on a 233-question validation set). Also exports `faq_catalog_lookup(query, exclude_intent_ids=...)` for direct reuse outside the tool-call path.
- `models/llm_thread.py`: `_llm_bot_faq_override_answer()` — if `faq_price_lookup` returned matched=True in the current turn, the tool's answer text is substituted verbatim for the model's own reply, because merely instructing the model (even via a `hinweis` field baked directly into the tool result, not just the docstring) to "repeat this verbatim" was NOT sufficient — it still occasionally mangled digits.
- `discuss_channel.py`: `faq_catalog_lookup(body_text, exclude_intent_ids={'price_overview'})` is tried FIRST, independent of whether the model called any tool at all in the current turn — needed because on a same-topic follow-up question the model often skips re-calling the tool and answers from memory, mangling the number again. `price_overview` is excluded from this always-on path because it's the one intent that fires on a bare price-word match with no product/category term (too high false-positive risk if applied blindly to every "kostet"-containing message).
- The live `llm.prompt` "Kundenservice Prompt" (DB record, not just the `scripts/kundenservice_prompt_export.md` snapshot which now needs re-export) was updated to route general price questions to `faq_price_lookup` before the old embedded PREISUEBERSICHT table (now an explicit fallback only).

Two structural bugs found and fixed along the way:
1. Price-guard product-name matching (`_get_known_prices_by_product` in discuss_channel.py) only recognized a product if the EXACT full product.template name appeared in the text. Real names carry qualifiers often omitted in natural sentences (e.g. real name "All-In Bundle Standard (36 Monate)" vs. spoken "All-In Bundle Standard") — when unmatched, the guard fell back to allowing ANY price in the whole catalog, letting an unrelated product's price (29 EUR, real price of "PrivateMind Backup Basic - Managed") through for the bundle (should have been 299 EUR). Fixed by registering an alias per product with any trailing "(...)" qualifier stripped (deliberately NOT stripping " - Managed", which denotes a genuinely different-priced product).
2. The keyword classifier's `_faq_has_word()` used `\bword\b` (both-side boundary), which fails on German inflections ("Rabatt" doesn't match inside "Rabatte" - no boundary between "t" and "e"). Triggered live when the model summarized a customer's discount-pressure message to `query="Rabatte"` when calling the tool. Fixed to left-anchored-only (`\bword`, no trailing `\b`).