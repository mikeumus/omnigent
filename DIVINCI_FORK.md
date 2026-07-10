# Divinci fork of Omnigent

This is our tracked fork of `omnigent-ai/omnigent`, running instead of the
Homebrew install so our patches survive `brew upgrade`.

- **Branch:** `divinci` (our patches) — based on `upstream/main` at `0.5.0.dev0`.
- **Remotes:** `origin` = mikeumus/omnigent (backup), `upstream` = omnigent-ai/omnigent.
- **Runtime:** editable `uv` venv at `.venv/`. The LaunchAgent
  `com.mikeumus.omnigent-stack` runs `.venv/bin/{omni,omnigent}`; `omni`/
  `omnigent` are also symlinked into `~/.local/bin` (first in PATH) so the
  interactive shell uses the fork. Homebrew's omnigent stays installed as a
  fallback (later in PATH).
- **Data dir:** the fork uses the SAME `~/.omnigent` (chat.db / configs / agent
  templates) — full continuity with the prior brew setup.

## Our patches (branch `divinci`)
- `model_catalog.py` — add `claude-fable-5`, `claude-sonnet-5` to subscription models.
- `runtime/workflow.py` — plumb `permission_mode` → `HARNESS_CURSOR_PERMISSION_MODE`.
- `inner/cursor_executor.py` — auto-approve cursor native tools when
  `permission_mode=bypassPermissions` (keeps Stage-1 catastrophic DENY).

Upstream issues filed: omnigent#2286 (cursor autonomy), omnigent#1933 (GUI PATH),
earendil-works/pi#6443 (pi-ai timeout).

## Updating from upstream
```sh
cd ~/Documents/omnigent
git fetch upstream
# Cherry-pick only the divinci commits (skip release commits):
git reset --hard upstream/main
git cherry-pick 00cc475f 653ee837 95e81565   # resolve conflicts; keep upstream where it already has our change
uv sync                          # re-resolve deps (may bump versions)
# restart the stack to pick it up:
launchctl bootout gui/$(id -u)/com.mikeumus.omnigent-stack
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.mikeumus.omnigent-stack.plist
```
If the rebase conflicts on a patched file, resolve it (our edits are small +
clearly marked `DIVINCI PATCH`), then `git rebase --continue`.

## pi-ai patch (separate — npm package, not omnigent)
The pi CLI (`@earendil-works/pi-coding-agent` → `@earendil-works/pi-ai`) is a
bundled npm dist, patched in place (LLM request timeout+retries). An npm
reinstall wipes it. Re-apply with:
```sh
~/.omnigent/patches/apply-piai-patches.sh   # idempotent
```
