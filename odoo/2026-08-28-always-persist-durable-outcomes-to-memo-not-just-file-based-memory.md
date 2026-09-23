---
created: '2026-08-28T15:17:02.076+02:00'
extra:
  entities:
  - Claude Code
  - User
  - Claude
  - Code
  - PrivateMind
  - Persist durable outcomes with memo_save
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
id: 17365ed9825b4e0fa0e3142c17981706
normalized_hash: 1f7d909be7bc878e
review_after: '2026-11-26T15:17:02.076000+02:00'
tags:
- workflow
- memory
- claude-code
- privatemind-odoo
- project:odoo
title: Always persist durable outcomes to memo, not just file-based memory
type: preference
updated: '2026-08-28T15:17:02.076+02:00'
valid_at: '2026-08-28T15:17:02.076+02:00'
verification_state: unverified
---

User explicit instruction (2026-08-26, PrivateMind/odoo project, im_livechat_llm_bot work): always persist durable session outcomes to memo via memo_save, in addition to (not instead of) the file-based Claude Code memory system (.claude/projects/.../memory/*.md). This was raised because an entire long session's fixes had only been saved to the file-based memory, not to memo, which the memo server's own instructions already call for ("Persist durable outcomes with memo_save"). Going forward: treat memo_save as a standing, default step at the end of any session with durable outcomes (bug fixes, decisions, architecture changes) — not something to be asked for each time.