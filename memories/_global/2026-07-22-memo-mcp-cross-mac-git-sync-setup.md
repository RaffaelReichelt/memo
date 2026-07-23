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
updated: '2026-07-23T11:36:25.040+02:00'
verification_state: unverified
---

memo-mcp (mlx-memo) cross-Mac git-sync setup, done on Mac-mini-von-Harald on 2026-07-22, CORRECTED 2026-07-23 (see bug below).

Registration: memo-mcp registered as MCP server in Claude Desktop
(~/Library/Application Support/Claude/claude_desktop_config.json, top-level
mcpServers key) and Claude Code (~/.claude.json, top-level mcpServers, user
scope).

CORPUS LAYOUT (corrected 2026-07-23 — the first version of this note had
this wrong, do not use the old value): memo's git sync (sync_git.py:
git_root_for()) requires .git at the PARENT of MEMO_DATA_DIR. Separately,
memo has a reserved internal bucket name GLOBAL_BUCKET = "_global"
(project.py) that it auto-appends under MEMO_DATA_DIR for every untagged
save. Setting MEMO_DATA_DIR directly to a folder literally named "_global"
causes double-nesting: new saves land at ".../_global/_global/<file>.md".
Correct structure: repo-root (.../CloudDocs/memo, remote
git@github-memo-sync:RaffaelReichelt/memo.git) contains a memories/
container (sibling of signal/ and embed_cache/, all three at repo root per
signal_dir_for()/embed_cache_dir_for() = memory_dir.parent/...), and
_global sits INSIDE memories/ as the actual bucket dir. So MEMO_DATA_DIR =
".../CloudDocs/memo/memories" — NOT ".../memo/_global", NOT ".../memo".
Set consistently in ~/.zshrc, both mcpServers.memo.env.MEMO_DATA_DIR
configs (Claude Desktop + Claude Code), and the sync-hook command in
~/.claude/settings.json. Run `memo reindex --rebuild` after ever changing
this path.

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
on RaffaelReichelt/memo (one deploy key per machine). gh CLI isn't
installed/authenticated here, so deploy-key registration is a manual step
via github.com/RaffaelReichelt/memo/settings/keys.

Verified end-to-end: ssh -T git@github-memo-sync authenticates
non-interactively; `memo sync once` / `memo sync status` pull+push cleanly,
including a real rebase-conflict resolution (see below).

BUG THAT HAPPENED (2026-07-22 to 2026-07-23): MEMO_DATA_DIR was first set
to ".../memo/_global" directly. This satisfied git_root_for() but caused
this very memory to be saved double-nested at
"_global/_global/2026-07-22-memo-mcp-cross-mac-git-sync-setup.md" and
pushed to GitHub. Caught by a GX10-side review of the pulled commit.
Fixed by: git mv-ing the whole _global/ tree into memories/_global/
(preserves history), fixing MEMO_DATA_DIR everywhere to
".../memo/memories", memo reindex --rebuild, then a real git rebase
against 2 concurrent GX10 commits that had (in the meantime) added 2 new
files flat at "_global/<file>.md" — git's rename-detection correctly
proposed relocating those into memories/_global/ during the rebase, which
was accepted, and the rebase completed cleanly. Pushed successfully after.

STILL OUTSTANDING / ACTION NEEDED:
- GX10: its own scheduled sync job (NOT memo's hook system) is still
  writing new memories flat as "_global/<file>.md" at the repo root (seen
  in commits 810d7ee / 2d4500f, 2026-07-23 ~02:50). GX10 needs to update
  wherever it sets its data dir / target path to point one level deeper,
  into "memories/_global/", or every future GX10 auto-sync will resurrect
  the flat structure and cause repeat rebase conflicts. Also unclear
  whether GX10 already has its own unattended-git-auth solution (deploy
  key or otherwise).
- MacBookPro: whole setup still needs replicating — MEMO_DATA_DIR =
  ".../memo/memories" (use the CORRECTED value, not the original mistake
  above), both mcpServers registrations, SessionStart/Stop hook wiring,
  and a FRESH per-machine deploy key (do not reuse another machine's
  private key).