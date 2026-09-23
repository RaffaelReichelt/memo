---
created: '2026-08-28T15:17:33.632+02:00'
extra:
  entities:
  - Postgres NOTIFY
  - AT COMMIT
  - Postgres LISTEN
  - Python
  - Postgres
  - Odoo
  - PrivateMind
  - ORM
  - ORIGINAL
  - HTTP
  - NOTIFY
  - COMMIT
  - LLM
  - EXPLICIT
  - LISTEN
  - addons/18.0/im_livechat_llm_bot/models/discuss_channel.py
  - Store
  - typing_message.write_date
  - datetime
  - _to_store()
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
id: 4587aeef18744b0ea2c36cbef6f47eb8
normalized_hash: 6916de71668f8321
tags:
- privatemind-odoo
- im_livechat_llm_bot
- odoo-bus
- websocket
- postgres-notify
- project:odoo
title: Odoo bus/websocket updates need explicit commit during long-running requests
type: bug
updated: '2026-08-28T15:17:33.632+02:00'
valid_at: '2026-08-28T15:17:33.632+02:00'
verification_state: unverified
---

In `addons/18.0/im_livechat_llm_bot/models/discuss_channel.py` (PrivateMind/odoo), the livechat bot's reply was only appearing to the visitor after a manual browser refresh — the same symptom recurred twice from two distinct causes.

Cause 1 (fixed first): the bus notification that replaces the "typing…" placeholder with the final answer built its `Store` from raw ORM attribute values (`typing_message.write_date` as a Python `datetime` object, not through the model's own `_to_store()` serializer). `mail.message._to_store()` is what the initial placeholder creation already uses correctly, which is why the placeholder itself always appeared live instantly. Fix: call `typing_message._bus_send_store(typing_message)` with NO explicit values dict, letting the standard `_to_store()` serializer run (same as any normal message fetch).

Cause 2 (found after the same symptom recurred with a much slower model, mistral-small3.2:24b at 12-90s vs. gemma4:12b's 3-15s): everything after the early "typing" placeholder's explicit commit (generation + safety-net guards + the final write) shares one never-explicitly-committed transaction that only auto-commits when the ORIGINAL visitor HTTP request finally returns — and Postgres NOTIFY (which the Odoo bus depends on to push live updates) only fires AT COMMIT, not at write time. For a long-enough generation this pushes past whatever proxy/browser tolerance exists: the final answer lands correctly in the DB (confirmed via write_date) but is never delivered live to the open page. Fix: add an explicit `self.env.cr.commit()` right after the final `_bus_send_store()` call — same technique already used for the placeholder's own early commit.

Lesson: any Odoo flow that does a long-running synchronous operation (LLM generation, etc.) inside a single request handler and expects live bus/websocket updates mid-flight needs an EXPLICIT commit after each update it wants delivered live, not just at the very end — Postgres LISTEN/NOTIFY only fires on commit.