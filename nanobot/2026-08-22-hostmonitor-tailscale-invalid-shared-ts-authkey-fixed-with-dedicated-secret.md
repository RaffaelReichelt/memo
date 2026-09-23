---
created: '2026-08-22T13:53:57.740+02:00'
extra:
  entities:
  - Docker Swarm
  - Swarm
  - Tailscale
  - Docker
  - BackendState
  - API
  - hostmonitor_tailscale
  - /home/raffael/Projekte/monitor/gpu_monitor/docker-compose.yml
  - 'restart_policy.max_attempts: 5'
  - ts_authkey
  - tailscale-state
  - 'invalid key: API key kqkT2wBhPn11CNTRL not valid'
  - docker service logs hostmonitor_tailscale
  - ts_authkey_hostmonitor
  - printf '%s' "tskey-auth-..." | docker secret create ts_authkey_hostmonitor -
  - docker-compose.yml
  - cat /run/secrets/...
  - 'secrets:'
  - docker stack deploy -c docker-compose.yml hostmonitor
  - docker exec <container> tailscale status --json
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
id: d1cdc4d78d124b189cf9150d459b4615
normalized_hash: 86661a9bc53b0b3d
tags:
- tailscale
- docker-swarm
- hostmonitor
- gx10-b3ed
- ts_authkey
- infra
- project:nanobot
title: 'hostmonitor_tailscale: invalid shared TS_AUTHKEY, fixed with dedicated secret'
type: bug
updated: '2026-08-22T13:53:57.740+02:00'
valid_at: '2026-08-22T13:53:57.740+02:00'
verification_state: unverified
---

On 2026-08-22, `hostmonitor_tailscale` (Docker Swarm global service on host gx10-b3ed, stack file `/home/raffael/Projekte/monitor/gpu_monitor/docker-compose.yml`) was found stuck at 0/0 replicas — had been failing since ~2026-08-01, once per host reboot, until Swarm's `restart_policy.max_attempts: 5` was exhausted around 10 days prior.

**Root cause:** all 7 Tailscale sidecar services on this host (nanobot, ai-stack, odoo, ollama, open-webui, opensearch, hostmonitor) shared one Docker secret `ts_authkey` (created 2026-07-04). The other 6 run fine because they already completed initial auth and now run on their own persisted node key (in their `tailscale-state` volume) — the auth key is only needed for the *first* login. `hostmonitor_tailscale`'s node had lost its server-side authorization and, on every restart, tried to re-auth with the shared `ts_authkey`, which the Tailscale control plane rejected outright: `invalid key: API key kqkT2wBhPn11CNTRL not valid`. Confirmed live via `docker service logs hostmonitor_tailscale`.

**Fix:** rather than rotate the shared `ts_authkey` (immutable Docker secret, in active use by 6 other services — deleting/recreating it would have required touching all of them), created a new *dedicated* reusable auth key in the Tailscale admin console and stored it as a new secret `ts_authkey_hostmonitor` (`printf '%s' "tskey-auth-..." | docker secret create ts_authkey_hostmonitor -`). Updated `docker-compose.yml`: entrypoint's `cat /run/secrets/...` path, the service-level `secrets:` list, and the top-level `secrets:` block, all switched from `ts_authkey` → `ts_authkey_hostmonitor`; also updated the in-file setup comment. Redeployed with `docker stack deploy -c docker-compose.yml hostmonitor`. Verified via `docker exec <container> tailscale status --json`: `BackendState: Running`, `Online: true`, reachable as `hostmonitor-gx10-b3ed.tail5a2ccd.ts.net`.

**General lesson for this host:** when checking "is any TS_AUTHKEY expired", `Docker secret ts_authkey` itself not being past a date isn't sufficient — the meaningful check is per-container: `docker exec <tailscale-container> tailscale status --json` → `BackendState`/`Self.Expired`, and if a node is failing to (re)join, check `docker service logs <service>` for `invalid key` / `You are logged out` before assuming it's a docker/network issue. Sharing one auth-key secret across many first-time-joining sidecars is fine as long as each only needs it once; a node that loses tailnet authorization later (revoked from admin console, etc.) needs a *fresh* key, and the old shared secret cannot simply be overwritten (Docker secrets are immutable, and can't be removed while any service still references it) — cheapest fix is a new dedicated secret for just that one service.

Related file-based memory: `project_swarm_tailscale_authkey` (the earlier 2026-07-16 nanobot_tailscale fix that established the shared `ts_authkey` secret + entrypoint-shim pattern in the first place).