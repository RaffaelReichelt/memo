---
created: '2026-09-20T11:55:29.383+02:00'
extra:
  entities:
  - Nachfrage Nutzer
  - Hast Du
  - Am Ende
  - Nachfrage
  - Nutzer
  - Memory
  - Casa
  - Ende
  - Arbeitssessions
  - Projekterkenntnisse
  - CasaOrfeo
  - Hast Du das auch mit memo gesichert?
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
id: cf07cd5ca07443f7b4d239dad9a33ff8
normalized_hash: 37402f2fd16393f2
tags:
- feedback
- memo-usage
- workflow
title: 'Feedback: immer proaktiv memo_save nutzen, nicht nur auf Nachfrage'
type: feedback
updated: '2026-09-20T11:55:29.383+02:00'
valid_at: '2026-09-20T11:55:29.383+02:00'
verification_state: unverified
---

Nutzer (Raffael, CasaOrfeo-Projekt) hat am 2026-09-20 explizit gebeten, Session-Erkenntnisse künftig standardmäßig sowohl in der lokalen dateibasierten Memory als auch über memo_save zu sichern - nicht nur auf Nachfrage wie zuvor ("Hast Du das auch mit memo gesichert?"). Gilt projektübergreifend, nicht nur für CasaOrfeo. Am Ende relevanter Arbeitssessions (Bugfixes, Entscheidungen, neue Projekterkenntnisse) proaktiv memo_save mit passendem type aufrufen, ohne dass danach gefragt werden muss.