# myclaude-launcher

Unified Claude Code launcher with Three Doors (Business / Personal / Bench), per-door MCP loading, and 1Password env wrapping.

Source-of-truth for the scripts deployed at `~/bin/myclaude*` on Matt's M4 and M1 minis.

## Files

| File | Purpose |
|------|---------|
| `myclaude` | The launcher itself. Door → project → entry-mode menu. |
| `myclaude-test` | Non-destructive single-key launch simulator. Mirrors `launch_claude` exactly but execs `node --version` instead of `claude`. Logs PATH/OP_WRAP state at every checkpoint to `/tmp/myclaude-test-<key>-<entry>.log`. |
| `myclaude-test-all` | Runs `myclaude-test` for every menu key in parallel. Used as the regression suite. |

## Test protocol

```sh
~/bin/myclaude-test-all fresh
~/bin/myclaude-test-all status
~/bin/myclaude-test-all handoff
~/bin/myclaude-test-all resume
```

Expected: `0 failed` on each. Per-key logs land in `/tmp/myclaude-test-*.log`.

## Bug history

### 2026-04-07 (b) — node/cli.js detection M1 mismatch

**Symptom on M1 Mini (after the path-clobber fix):** myclaude exits at startup with `Error: claude cli.js not found at /opt/homebrew/lib/node_modules/@anthropic-ai/claude-code/cli.js`. M1 has homebrew node installed (v24.2.0) but `@anthropic-ai/claude-code` is installed under nvm (v22.17.1), not homebrew. The old detection branched on `[[ -x /opt/homebrew/bin/node ]]` first and never checked whether cli.js actually existed there.

**Fix:** Detection now checks both Homebrew and nvm and picks whichever has BOTH `node` AND `cli.js`. Homebrew wins ties. Clear error surfaces the install state of both branches if neither is usable.

### 2026-04-07 (a) — `path↔PATH` clobber (Phase 1.7b regression)

**Symptom on M4 Mini:** Every menu choice exited 127 with:

```
log_launch:2: command not found: date
launch_claude:35: command not found: op
claude exited with code 127
```

**Root cause:** zsh ties the lowercase scalar `path` bidirectionally to the uppercase array `PATH`. The launcher used `path` as both a `while read` loop variable and a `launch_claude` function parameter:

```zsh
while IFS='|' read -r key door label path; do   # clobbers PATH
typeset door="$1" label="$2" path="$3" entry="$4"  # clobbers PATH again
```

After `cd "$target"`, `PATH` was the project directory string, so `date`, `op`, and every other external command died with command-not-found.

The bug was latent for months — the launcher never needed `PATH` after the loop until Phase 1.7b added (a) a `date -u` call inside `log_launch` and (b) an `op run --env-file=… --` wrap around the `claude` exec. Both surfaced the same underlying corruption.

**Fix:** Renamed `path` → `proj_path` in all 6 affected sites in `myclaude` and the same shadowing in `myclaude-test-all`.

**Verification:** `myclaude-test-all` covers all 8 keys × 4 entry modes = 32 cells. All 32 pass post-fix.

**Lesson:** In zsh, never use `path`, `cdpath`, `fpath`, `manpath`, or `mailpath` as ordinary variable names. They are always tied to their uppercase counterparts.
