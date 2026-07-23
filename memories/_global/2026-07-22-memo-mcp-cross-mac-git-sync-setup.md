---
created: '2026-07-22T15:40:01.692+02:00'
extra:
  entities:
  - Claude Desktop
  - Application Support
  - Claude Code
  - So MEMO
  - Secure Enclave
  - Touch ID
  - GitHub Deploy Key WITH WRITE ACCESS
  - STILL OUTSTANDING
  - Claude
  - Desktop
  - Support
  - Code
  - Drive
  - Session
  - Secretive
  - Enclave
  - Touch
  - Host
  - Deploy
  - Raffael
id: f3e92f47cec34467b743aa065c85eab2
normalized_hash: dc74d0033fdbd5b6
tags:
- memo-sync
- infrastructure
- macbookpro-todo
title: memo-mcp cross-Mac git-sync setup
type: manual
updated: '2026-07-22T15:40:01.692+02:00'
verification_state: unverified
---

memo-mcp (mlx-memo) cross-Mac git-sync setup, done on Mac-mini-von-Harald on 2026-07-22.

Registration: memo-mcp registered as MCP server in Claude Desktop
(~/Library/Application Support/Claude/claude_desktop_config.json, top-level
mcpServers key) and Claude Code (~/.claude.json, top-level mcpServers, user
scope).

Critical data-dir/git layout gotcha: memo's git sync (sync_git.py:
git_root_for()) requires the git repo (.git) to live at the PARENT of
MEMO_DATA_DIR, not at MEMO_DATA_DIR itself. Corpus lives in iCloud Drive at
.../CloudDocs/memo (the git clone, remote git@github.com:RaffaelReichelt/memo.git),
actual memories in a _global subfolder. So MEMO_DATA_DIR must be
".../CloudDocs/memo/_global" (one level deeper than the repo root), not
".../CloudDocs/memo". Set consistently in three places: ~/.zshrc (export
MEMO_DATA_DIR=..., no space around =), Claude Desktop config's
mcpServers.memo.env.MEMO_DATA_DIR, Claude Code config's same key.

Sync automation: `memo sync auto` (debounced, safe every prompt) is NOT
auto-wired by memo's self-heal (self-heal only installs recall-hook and
capture-tick). Wired manually into ~/.claude/settings.json under
SessionStart + Stop hooks: `MEMO_NONINTERACTIVE=1 MEMO_DATA_DIR="..." /opt/homebrew/bin/memo sync auto`.

Unattended git auth: default SSH config routes everything through Secretive
(Secure Enclave agent, needs interactive Touch ID) via a global `Host *`
block — breaks unattended hook-triggered sync. Fix: per-machine dedicated
passphrase-less ed25519 key (ssh-keygen -t ed25519 -f
~/.ssh/id_ed25519_memo_sync -N "" -C "memo-sync@<hostname>"), a Host block
ABOVE `Host *` in ~/.ssh/config (order matters, first match wins):
  Host github-memo-sync
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_memo_sync
    IdentitiesOnly yes
    IdentityAgent none
then `git remote set-url origin git@github-memo-sync:RaffaelReichelt/memo.git`
on the corpus repo. Public key added as a GitHub Deploy Key WITH WRITE ACCESS
on RaffaelReichelt/memo (one deploy key per machine, chosen over a shared
key so a lost device can be revoked individually). gh CLI isn't
installed/authenticated here, so deploy-key registration is a manual step
via github.com/RaffaelReichelt/memo/settings/keys.

Verified end-to-end: ssh -T git@github-memo-sync authenticates
non-interactively; `memo sync once` / `memo sync status` pull+push cleanly.

STILL OUTSTANDING: replicate this entire setup on the MacBookPro — MEMO_DATA_DIR
fix, both mcpServers registrations, SessionStart/Stop hook wiring, and a FRESH
per-machine deploy key (do not reuse this machine's private key). GX10 already
has a working scheduled sync job via a separate (non-memo-hook) mechanism —
unclear if it already solved unattended auth the same way; worth checking
before assuming.