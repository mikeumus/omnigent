# Divinci fork of Omnigent

This is our tracked fork of `omnigent-ai/omnigent`, running instead of the
Homebrew install so our patches survive `brew upgrade`.

- **Branch:** `divinci` (our patches) — based on `upstream/main` at `0.6.0.dev0`.
- **Remotes:** `origin` = mikeumus/omnigent (backup), `upstream` = omnigent-ai/omnigent.
- **Runtime:** editable `uv` venv at `.venv/`. The LaunchAgent
  `com.mikeumus.omnigent-stack` runs `.venv/bin/{omni,omnigent}`; `omni`/
  `omnigent` are also symlinked into `~/.local/bin` (first in PATH) so the
  interactive shell uses the fork. Homebrew's omnigent stays installed as a
  fallback (later in PATH).
- **Data dir:** the fork uses the SAME `~/.omnigent` (chat.db / configs / agent
  templates) — full continuity with the prior brew setup.

## Our patches (branch `divinci`)
- `inner/cursor_executor.py` — cwd writable-fallback when cursor is dispatched
  as a sub-agent and relative `os_env.cwd` resolves to `/` (avoids
  `[Errno 30] Read-only file system: '/.cursor'`).

Previously carried, now **upstream**:
- model catalog entries for Fable5 / Sonnet5 / Opus4.8
- cursor `permission_mode` → `HARNESS_CURSOR_PERMISSION_MODE` plumbing
- cursor autonomy (`auto` / `bypassPermissions` skip ApprovalCards)

Upstream issues filed: omnigent#2286 (cursor autonomy), omnigent#1933 (GUI PATH),
earendil-works/pi#6443 (pi-ai timeout).

## Updating from upstream
```sh
cd ~/Documents/omnigent
git fetch upstream
# Cherry-pick only the remaining divinci commits (skip anything already upstream):
git reset --hard upstream/main
git cherry-pick <docs-DIVINCI_FORK> <cwd-writable-fallback>   # drop SHAs already upstream
uv sync                          # re-resolve deps (may bump versions)
# restart the stack to pick it up:
launchctl bootout gui/$(id -u)/com.mikeumus.omnigent-stack
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.mikeumus.omnigent-stack.plist
```
If a cherry-pick conflicts on a patched file, resolve it (our edits are small +
clearly marked `DIVINCI PATCH`), then `git cherry-pick --continue`. Prefer
`git cherry-pick --skip` when upstream already absorbed the change.

## pi-ai patch (separate — npm package, not omnigent)
The pi CLI (`@earendil-works/pi-coding-agent` → `@earendil-works/pi-ai`) is a
bundled npm dist, patched in place (LLM request timeout+retries). An npm
reinstall wipes it. Re-apply with:
```sh
~/.omnigent/patches/apply-piai-patches.sh   # idempotent
```
