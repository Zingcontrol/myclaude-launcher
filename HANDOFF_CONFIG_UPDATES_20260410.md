# myclaude-launcher — Config Updates Handoff
**Generated:** 2026-04-10  
**For:** Next myclaude-launcher session  
**Namespace:** Personal (launcher infrastructure is cross-cutting; canonical home is Business, this copy is the execution record)

---

## TL;DR

Six items. **Three resolved** (Items 0, 1, and deco-api confirmation). Three remain (version-control configs, M1 parity, structural debt).

**Resolved 2026-04-10 by knowledge-dev session:**
- Item 0: knowledge-dev moved from Business → Personal in projects.conf ✓
- Item 1: 00-personal-ops added as hotkey `n` under Personal door ✓
- Item 3: deco-api confirmed Personal (current assignment is correct) ✓

---

## Item 0 — Move knowledge-dev from Business to Personal door
**Priority: Execute immediately — Chairman directive 2026-04-10**

`knowledge-dev` is currently registered as Business:
```
k|Business|Knowledge Development|~/claude-projects/knowledge-dev
```

Matt confirmed on 2026-04-10 that this should be Personal. It's a personal knowledge development environment, not a Zingcontrol business project. The CLAUDE.md in the project explicitly says "This is not a business project."

**Change to:**
```
k|Personal|Knowledge Development|~/claude-projects/knowledge-dev
```

This also means the knowledge-dev git repo (once initialized) should go to `StillMatt/knowledge-dev`, not `Zingcontrol/knowledge-dev`.

**Action steps:**
1. Edit `~/.myclaude/projects.conf` — change `k|Business|` to `k|Personal|`
2. Verify the change: `myclaude` → pick `k` → confirm it launches under Personal door with personal MCP config

---

## Item 1 — Add 00-personal-ops to projects.conf
**Priority: Execute immediately**

`~/claude-projects/00-personal-ops/` exists and is in active use but has no entry in `~/.myclaude/projects.conf`. Sessions launched there have no door assignment, which means no namespace enforcement and no MCP config loading.

**Available hotkeys** (current assignments: z, k, o, f, h, d, p, b):  
All single-letter keys are spoken for. Options:
- `n` — "persoNal ops" (stretch, but workable)
- `s` — "perSonal ops"
- `a` — "personAl ops"

Recommended: `n` — it's unused, Personal-adjacent in feel, and easy to type.

**Line to add to `~/.myclaude/projects.conf`**, under the Personal block, after the `p` entry:

```
n|Personal|Personal Ops|~/claude-projects/00-personal-ops
```

Full insertion point — replace the current Personal block footer:

```
p|Personal|Personal Projects|~/personal/projects
```

with:

```
p|Personal|Personal Projects|~/personal/projects
n|Personal|Personal Ops|~/claude-projects/00-personal-ops
```

**Action steps:**
1. Open `~/.myclaude/projects.conf`
2. Add the `n` line under the Personal block
3. Run `myclaude` and verify `n` appears in the menu and launches correctly from `~/claude-projects/00-personal-ops/`

---

## Item 2 — Version-Control projects.conf and m1-setup.sh
**Priority: Execute — low risk, high value**

`~/.myclaude/projects.conf` is the canonical project registry. It is not tracked in any git repo. Same for `m1-setup.sh` if it exists. These are functional config files that should survive machine rebuilds and be diffable over time.

**Current state:**
- `~/claude-projects/myclaude-launcher/` is a git repo (branch: main, clean working tree)
- `projects.conf` lives at `~/.myclaude/projects.conf` — outside the repo
- The launcher repo contains only: `myclaude`, `myclaude-test`, `myclaude-test-all`, `README.md`

**Options:**
1. Move `projects.conf` into the launcher repo and symlink it to `~/.myclaude/projects.conf`
2. Copy it in as a tracked file and manually sync when changed (fragile)
3. Add `~/.myclaude/` as a separate tracked repo

Option 1 is cleanest. The launcher repo becomes the canonical source.

**Action steps:**
1. Copy `~/.myclaude/projects.conf` into `~/claude-projects/myclaude-launcher/projects.conf`
2. Delete `~/.myclaude/projects.conf`
3. Symlink: `ln -s ~/claude-projects/myclaude-launcher/projects.conf ~/.myclaude/projects.conf`
4. Add `.gitignore` in the launcher repo to exclude `*.bak-*` files (covers `projects.conf.bak-20260406` and `projects.conf.bak-20260407`)
5. If `m1-setup.sh` exists anywhere, copy it into the launcher repo as well
6. `git add projects.conf .gitignore` (and `m1-setup.sh` if found), commit

**Note:** The `.bak-20260406` and `.bak-20260407` backup files in `~/.myclaude/` should be excluded from tracking. Add this to `.gitignore`:
```
*.bak-*
```

---

## Item 3 — deco-api Namespace Classification
**Priority: Chairman Decision Required — see section below**

`deco-api` is currently listed under Personal door in `projects.conf`:
```
d|Personal|Deco Dashboard|~/claude-projects/deco-api
```

The label says "Deco Dashboard" and the door is Personal. This is the Firewalla/network monitoring API project at `~/claude-projects/deco-api/`.

The question is whether this is:
- **Personal** (home network hobby project, no Zingcontrol dependency) → current assignment is correct, no action needed
- **Business** (infrastructure tooling that supports Zingcontrol operations) → move to Business door, update the entry

The distinction matters because Personal door loads personal MCP configs (`~/.myclaude/mcp/personal.json`). If deco-api work involves Zingcontrol infrastructure, Business door is the correct namespace.

**No action until Matt decides.** See Chairman Actions section.

---

## Item 4 — M1 Configuration Parity
**Priority: Field op — execute on M1, status unknown**

A deployment doc was generated at `~/Documents/M1_MYCLAUDE_DEPLOYMENT.md` (on the M4). No results or confirmation doc was found, so M1 state is unknown.

**What M1 needs to be fully configured:**

| Component | Path | Status |
|---|---|---|
| myclaude script | `~/bin/myclaude` | Unknown |
| projects.conf | `~/.myclaude/projects.conf` | Unknown |
| MCP configs | `~/.myclaude/mcp/{business,personal,bench}.json` | Unknown |
| op-env | `~/.myclaude/op-env` | Unknown |
| 1Password CLI (`op`) | system | Unknown |

**Notes:**
- `projects.conf` is designed to silently skip directories that don't exist on a given machine, so the same file can be deployed to both machines without modification
- If symlink approach from Item 2 is adopted, M1 would need the launcher repo cloned and the symlink created there too
- Until M1 is confirmed configured, any sessions there have no Three Doors enforcement

**Action steps (on M1):**
1. Check if `~/bin/myclaude` exists and matches current launcher version
2. Check if `~/.myclaude/projects.conf` exists
3. If either is missing, re-run the deployment procedure from `~/Documents/M1_MYCLAUDE_DEPLOYMENT.md`
4. After deployment, create a results doc at `~/Documents/M1_MYCLAUDE_DEPLOYMENT_RESULTS_20260410.md` confirming what was done

---

## Item 5 — Launcher Structural Debt (Backlog)
**Priority: Track, don't block on**

These are known gaps that don't require immediate action but should be on the radar:

**Personal door write isolation is convention-only.**  
MCP configs loaded under Personal door can technically write to Business surfaces. There is no structural enforcement. This is a known gap in the Three Doors model. Mitigation would require MCP-level ACL enforcement or per-door credential scoping — non-trivial.

**Handoff mode picks the most recent .md file.**  
If multiple unrelated handoff docs accumulate in a project directory, the wrong one can be picked. Convention fix: use a `HANDOFF_` prefix and a cleanup step after handoffs are executed.

**No auto-update mechanism.**  
When the myclaude script or MCP configs change on one machine, the other machine doesn't know. Manual sync required. If the launcher repo is properly tracked (Item 2), a `git pull` on both machines is the sync mechanism — but it's still manual.

**figma token is raw in `~/.claude.json`.**  
Should be stored in 1Password and referenced via `op run` or equivalent. This is a credential hygiene issue. Fix: add figma token to 1Password, update `~/.myclaude/op-env` to inject it, remove it from `~/.claude.json`.

---

## Chairman Actions Required

These items cannot be resolved without Matt's explicit decision:

### Decision: deco-api namespace
**Question:** Is `deco-api` (Firewalla/network monitoring) Business or Personal?

- Current assignment: Personal (`d|Personal|Deco Dashboard|~/claude-projects/deco-api`)
- If this is home network tooling with no Zingcontrol dependency → Personal is correct, no change
- If this is infrastructure that supports Zingcontrol operations → change to Business door

**To resolve:** Tell the next session "deco-api is Personal" or "deco-api is Business." That's the full decision. The session will update projects.conf accordingly.

### Decision: hotkey for 00-personal-ops
**Recommended:** `n` (see Item 1). If that's wrong, name the preferred letter and the session will use it.

---

## Execution Order

If executing all items in one session:

1. Get Matt's decisions on deco-api namespace and hotkey (or proceed with `n` as default)
2. Item 1: Add `00-personal-ops` to projects.conf
3. Item 2: Symlink projects.conf into launcher repo, add .gitignore, commit
4. Item 3: Apply deco-api namespace update if decision is made
5. Item 4: Flag M1 as needing field verification — cannot be done remotely
6. Item 5: No action, backlog only
