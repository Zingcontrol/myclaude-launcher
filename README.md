# myclaude-launcher

**Deployment vector for the myclaude launcher script + test harness.** Not the source-of-truth for docs or config — just the git home for the binaries.

## Canonical references (READ THESE before modifying anything in this repo)

| Topic | Canonical doc |
|---|---|
| Architecture (Three Doors, per-door MCP, op-env wrap, entry modes, known deficiencies) | `~/myclaude/README.md` on M4 |
| The Three Doors spec | `myclaude_spec_v2.md` (Personal handoff folder) |
| The original build package | `myclaude_unified_build_package.md` (Personal handoff folder) |
| The unification execution plan | `~/.claude/plans/breezy-snuggling-mitten.md` |
| The M1 reorg runbook | `~/Documents/M1_SESSION2_REORG_20260407.md` |
| The M1 deployment runbook | `~/Documents/M1_MYCLAUDE_DEPLOYMENT.md` |
| Per-door MCP architecture (memory) | `project_three_doors_architecture.md` |
| Per-machine projects.conf, op-env, MCP configs | `~/.myclaude/` on each machine (machine-local, NOT in this repo) |

## What lives here

| File | Purpose |
|------|---------|
| `myclaude` | The launcher script. Door → project → entry-mode menu. Deployed to `~/bin/myclaude` on M4 and M1. |
| `myclaude-test` | Non-destructive single-key launch simulator. Mirrors `launch_claude` exactly but execs `node --version` instead of `claude`. Logs PATH/OP_WRAP state at every checkpoint. |
| `myclaude-test-all` | Runs `myclaude-test` for every menu key in parallel. Regression suite. |

## What does NOT live here (and why)

- **`projects.conf`** — machine-local at `~/.myclaude/projects.conf`. Each machine has its own. The unification spec is in `myclaude_unified_build_package.md` (canonical doc above).
- **`mcp/*.json`** — machine-local at `~/.myclaude/mcp/`. Per-door, references `${VAR}` from op-env.
- **`op-env`** — machine-local at `~/.myclaude/op-env`. References `op://Private/...` 1Password items. Never source-controlled.
- **README beyond this file** — the canonical README is `~/myclaude/README.md`.
- **An `m1-setup.sh`** — the canonical M1 setup is `~/Documents/M1_SESSION2_REORG_20260407.md`, designed to be executed by a Claude Code session running ON M1.

## Deploying

Either machine, public raw URL works without auth:

```sh
cd ~/bin && \
  curl -fsSLO https://raw.githubusercontent.com/Zingcontrol/myclaude-launcher/main/myclaude && \
  curl -fsSLO https://raw.githubusercontent.com/Zingcontrol/myclaude-launcher/main/myclaude-test && \
  curl -fsSLO https://raw.githubusercontent.com/Zingcontrol/myclaude-launcher/main/myclaude-test-all && \
  chmod +x myclaude myclaude-test myclaude-test-all
```

Then run the regression suite to verify:

```sh
for e in fresh status handoff resume; do ~/bin/myclaude-test-all $e; done
```

Expected: `0 failed` on each run.

## Bug history

### 2026-04-07 (b) — node/cli.js detection M1 mismatch

M1 has Homebrew node installed but `@anthropic-ai/claude-code` is under nvm, not Homebrew. Old detection branched on `[[ -x /opt/homebrew/bin/node ]]` first and never checked whether cli.js was actually present at the Homebrew path. Launcher exited at startup with `Error: claude cli.js not found at /opt/homebrew/lib/.../cli.js`.

**Fix:** Detection now checks both Homebrew and nvm and picks whichever has BOTH `node` AND `cli.js`. Homebrew wins ties. Clear diagnostic error if neither is usable.

### 2026-04-07 (a) — `path↔PATH` clobber (Phase 1.7b regression)

In zsh, the lowercase scalar `path` is bidirectionally tied to the uppercase `PATH` array. The launcher used `path` as both a `while read` loop variable and a `launch_claude` function parameter — every assignment silently clobbered `PATH`. After `cd "$target"`, `PATH` was the literal project directory string. Phase 1.7b's new `date` (in `log_launch`) and `op` (in `OP_WRAP`) calls hit a PATH with no `/bin` and died with exit 127.

**Fix:** Renamed `path` → `proj_path` everywhere. Verified 32/32 cells pass on M4 + 20/20 on M1.

**Lesson encoded in memory:** `feedback_zsh_path_tied_params.md` — never use `path`/`cdpath`/`fpath`/`manpath`/`mailpath` as variables in zsh.

## History note

This repo was created on 2026-04-07 during a session that initially built parallel infrastructure (a draft `projects.conf`, `m1-setup.sh`, and a different README) instead of reading the existing canonical docs listed above. The parallel files were removed in the same session once the canonical infrastructure was discovered. The repo's current scope is intentionally narrow: it holds only the launcher binary + test harness as a deployment vector. Documentation and config remain in their canonical locations.

The lesson — **Look → Inventory → Plan → Approve → Move → Reconcile** — is encoded in memory at `feedback_six_step_governance.md`.
