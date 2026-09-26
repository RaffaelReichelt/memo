---
created: '2026-09-26T15:47:48.655+02:00'
extra:
  entities:
  - Dead HTTP
  - Did NOT
  - GLOBAL DNS
  - Same DNS
  - Memo
  - Swarm
  - Tailscale
  - Core
  - ImagePullBackOff
  - DNS
  - HTTP
  - PVC
  - README
  - NOT
  - GLOBAL
  - MEMO_HTTP_API_TOKEN
  - memo_http_api_token
  - memo-swarm
  - docker-compose.yml
  - entrypoint.sh
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
id: ad7a8aab836b425c894e53a477c7f8b5
normalized_hash: 9a527f411b5fa3f0
tags:
- nanobot
- memo
- docker-swarm
- kubernetes
- kind-learn
- gx10-b3ed
- dns
- tailscale
- project:nanobot
title: 'nanobot: outdated Memo config (Swarm+k8s) + host/cluster public-DNS-via-Tailscale
  bug, fixed 2026-09-26'
type: bug
updated: '2026-09-26T15:47:48.655+02:00'
valid_at: '2026-09-26T15:47:48.655+02:00'
verification_state: unverified
---

nanobot (Swarm + k8s), 2026-09-26. User reported nanobot "seems to access an outdated Memo configuration" - root cause was two-fold, found and fixed on both deployments:

**1. Dead HTTP-token wiring (Swarm + k8s):** `MEMO_HTTP_API_TOKEN`/`memo_http_api_token` was leftover from the pre-2026-08-19 remote `memo-swarm` HTTP integration, superseded by per-user stdio memo-mcp. Removed from `docker-compose.yml`, `entrypoint.sh`, and the k8s sealed secret/deployment/docs (commits `0410c31`, `a45fcbe` in nanobot repo). Sealed-secret keys are independently encrypted per bitnami sealed-secrets, so removing one key is a safe plain-text edit - no reseal/plaintext-secret needed.

**2. The actual outdated config (k8s only):** the k8s PVC's live `config.json` (`/var/local-path-provisioner/pvc-2b716567-8650-4531-b519-2208459eb73c_nanobot_nanobot-data/config.json` on kind node `learn-worker`) still had the fully dead `tools.mcpServers.memo` entry pointing at `https://memo-swarm.tail5a2ccd.ts.net/mcp` (streamableHttp + bearer token) - a service that has not been deployed since the 2026-08-19 stdio migration. Swarm's `/var/lib/nanobot/config.json` had already been migrated back then; k8s's PVC config never was, even though `k8s/nanobot/README.md` already documented the correct target config. Rewrote it in place to match Swarm's stdio config (`uvx --from mlx-memo memo-mcp`, `MEMO_DATA_DIR={{private_dir}}/.memo-data/memories`). Did NOT flip the websocket auth model (token vs. `webuiBootstrapTrustedHeader`+`tailscaleUserIsolation`) even though the README recommends it for per-user isolation to fully work - left conservative since the tailscale sidecar can't reach Tailscale-serve anyway right now (see below) and flipping to header-trust auth without a working Tailscale-serve proxy in front would make it spoofable.

**3. Root infra bug found along the way, fixed for Swarm only:** the host (gx10-b3ed) uses Tailscale as the systemd-resolved GLOBAL DNS override (100.100.100.100), with no fallback - so every container's embedded DNS resolver can answer `*.ts.net` but fails outright on public hostnames (confirmed: `pypi.org` unresolvable). This silently broke `uvx --from mlx-memo memo-mcp`'s ability to ever fetch a newer version (it only worked because a stale wheel happened to already be cached in the persistent `embedding_cache` volume). Fixed for the nanobot Swarm service by adding explicit `dns: [100.100.100.100, 1.1.1.1]` + `dns_search: tail5a2ccd.ts.net` to its `docker-compose.yml` service block - confirmed both `*.ts.net` and public DNS resolve afterward.

**Same DNS bug also breaks the kind-learn k8s cluster** (confirmed: `tailscale/tailscale:latest` ImagePullBackOff on the node with "dns error", and memo-mcp's own `uvx`->pypi.org lookup failing inside the pod with the identical error) - NOT fixed, out of scope (cluster-wide CoreDNS/node-resolv.conf fix, bigger blast radius than one Swarm service's dns: override, would need explicit go-ahead).

**Side discovery:** the k8s nanobot Deployment had no `strategy: {type: Recreate}` - with the tailscale sidecar's exclusive hostPort (41642) and local-path-provisioner PVC node-affinity, RollingUpdate can never complete (new pod stuck Pending, old pod never torn down). This silently meant the Deployment had been un-upgradeable for its whole 67-day life (pod was 1/2 Ready, ImagePullBackOff, 334 restarts, entirely unnoticed). Hit this live while applying the config fix, had to manually delete the stuck old pod + scale down/delete stale ReplicaSets. Added `strategy: {type: Recreate}` to `04-deployment.yaml` to prevent recurrence (commit `a45fcbe`).

**Still open / needs a decision:** kind-learn cluster-wide public DNS fix (would also fix the tailscale sidecar image pull); whether to flip k8s config.json's websocket auth to `tailscaleUserIsolation` (blocked on the DNS fix anyway since the sidecar can't come up to provide Tailscale-serve headers).