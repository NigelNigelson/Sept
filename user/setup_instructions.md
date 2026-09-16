# Running this workflow in Claude Desktop — step by step

This assumes Claude Desktop is installed and you have access to the **Code tab** (Claude Code running inside the desktop app).

## 1. Set up the project folder on disk
Create a folder anywhere on your machine (e.g. `neuro-cities-finalization/`) and lay it out as in `workflow_design.md`'s "Recommended project structure": a `data/sections/` folder holding your 8 per-section files (`law.json`, `financial.json`, `geography.json`, `ecosystem_climate.json`, `household_lifestyle.json`, `amenities.json`, `visitability.json`, `geography_context.json`), and a `data/` folder above it holding `city_schema_source_of_truth.md`, `audit_discrepancy_log.md`, and `source_tier_list.md` (save the tier list you pasted into this file). Also create empty `.claude/agents/` and `passes/` folders, and put `CLAUDE.md` at the project root.

Open a terminal in that folder and run `git init`, then `git add . && git commit -m "baseline"`. This isn't optional — the diff pane and the checkpoint policy both depend on git being present so every pass is reviewable and revertable. With the section-file layout, this initial commit *is* the baseline — there's no separate baseline copy to maintain.

## 2. Open the project in the Code tab
In Claude Desktop, open the **Code** tab. Start a new session pointed at your project folder — the app will prompt you to pick a folder if one isn't already open. You'll see the session's **chat pane** open by default.

## 3. Add the subagent files
Drop the 10 files from `agents/` into your project's `.claude/agents/` folder using your regular file system (Finder/Explorer), or open the **file pane** from the **Views** menu in the session toolbar and create them there directly.

If `.claude/agents/` didn't exist before this session started, Claude Code won't detect it until you restart the session — close and reopen this Code tab session once after adding the first agent file.

## 4. Confirm the agents loaded
In the chat pane, type:
```
List the subagents available in this project
```
You should see all 10 names (`field-remover`, `field-reshaper`, `law-researcher`, `distance-calculator`, `seasons-reconciler`, `financial-researcher`, `geography-researcher`, `category-auditor`, `contract-checker`, `schema-validator`) alongside any built-in ones. If any are missing, double-check the file landed in `.claude/agents/` and that its YAML frontmatter starts on the file's first line.

## 5. Run Pass 0 (baseline)
In the chat pane:
```
Run Pass 0: confirm all 8 files under data/sections/ are present, confirm the project is git-initialized and the working tree is clean, then confirm you've read CLAUDE.md and workflow_design.md before we start.
```
This is the coordinator (this main session) acting directly — no subagent dispatch yet.

## 6. Dispatch each pass from the chat pane
For each pass, type a natural-language instruction naming the agent — Claude Code delegates automatically when you name a subagent by its file name. Example for Pass 1:
```
Use the field-remover subagent to run Pass 1 as described in CLAUDE.md.
```
If you want to force a specific subagent rather than let Claude choose, type `@` in the chat pane and pick it from the typeahead list instead of typing the name in prose.

## 7. Monitor while it runs
Open the **tasks pane** from the **Views** menu — it shows any subagent currently running in the background, along with background shell commands. Click a row to open that subagent's live output in the **subagent pane**. You can drag panes to rearrange your layout, or press **Cmd+\\** (macOS) / **Ctrl+\\** (Windows) to close a pane you don't need open.

Pass 6 runs several research subagents in parallel (`law-researcher`, `financial-researcher`, `geography-researcher`) — you'll see multiple rows in the tasks pane simultaneously. That's expected. (Pass 5 is a single agent, `seasons-reconciler` — nothing parallel there.)

## 8. Review the diff before approving
When a pass finishes, open the **diff pane** (Views menu) to see exactly what changed in the section file(s) that pass touched (e.g. `data/sections/law.json`) compared to the previous commit. Read the coordinator's summary in the chat pane alongside the diff. If it looks right, tell the coordinator to proceed:
```
Approved — commit this and start the next pass.
```
If something looks wrong, say so specifically — the coordinator will route the fix to the right agent rather than you editing the JSON by hand.

## 9. Repeat through the pass sequence
Passes 2 through 7, and Pass 9, follow the same pattern: dispatch → monitor via tasks pane → review via diff pane → approve → next pass. (Pass 8 is the odd one out — coordinator-only schema reconciliation, no subagent to watch; see step 10.) Passes 2, 4, and 6 — anything with live research — each get two checkpoints (canary batch, then full batch) — don't skip the canary review even though it's tempting once the pipeline is running smoothly.

## 10. Final sign-off
When `schema-validator` (Pass 9) reports back, that's your finalization gate. If it comes back clean across all 39 cities, the dataset is done. If it flags anything, the coordinator will tell you which pass to re-run — you won't need to start over.

---

## Quick reference: pane names you'll use
| Pane | What it's for |
|---|---|
| **chat** | Where you talk to the coordinator and dispatch subagents |
| **tasks** | Live list of running subagents and background commands |
| **subagent** | Opens when you click a task row — that subagent's own transcript |
| **diff** | Shows file changes since the last commit |
| **file** | Browse/edit project files directly |
| **terminal** | Direct shell access, if you need it |

All are opened via the **Views** menu in the session toolbar and can be dragged into whatever layout you want.
