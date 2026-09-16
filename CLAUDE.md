# Project: Neurology Residency Cities — Schema Finalization

You are the coordinator for this project. You dispatch the subagents in `.claude/agents/` in the pass order below; you never do the pass work yourself except Pass 0 and Pass 8. Full detail on every decision behind this pipeline lives in `data/workflow_design.md` — read it before your first delegation if you haven't already.

## Scope
This pipeline covers 24 tasks (tasks 3 and 11 each split into a base task and an "a" follow-up; task 6a added alongside task 6 to cover a data gap found during pipeline review — see `workflow_design.md` §1) drawn from `data/audit_discrepancy_log.md`. **`commute`, `connectivity`, and `_meta`/tag-formatting are explicitly out of scope** — do not delegate any task touching those.

## Reference files

**Operational — agents actually read/write these at runtime:**
- `data/sections/*.json` (8 files: `law`, `financial`, `geography`, `ecosystem_climate`, `household_lifestyle`, `amenities`, `visitability`, `geography_context`) — the actual working data. Each entry = `{city_ref, city, state_ref, residency_context, <section_name>}`. Every agent reads/writes only the section file(s) its job touches — there is no single shared working file.
- `data/cities_final.json` — does not exist until Pass 9 assembles it from all 8 section files, joined on `city_ref`. This is the actual deliverable.
- `data/city_schema_source_of_truth.md` — the pre-workflow spec. Starting point, not the final word — see Pass 8.
- `data/city_schema_final.md` — does not exist until Pass 8 produces it. This is what Pass 9 validates against.
- `data/output_contracts.md` — the explicit, scriptable pass/fail checklist `contract-checker` runs after every pass. Assertions are scoped per section file.
- `data/golden_record_atlanta.md` — structural (not factual) worked examples, one per section file.
- `data/source_tier_list.md` — full citation tier list; each research agent has its own curated excerpt baked in. If Pass 8 approves a sourcing-policy gap, patch it here too, same approval flow as a schema patch.

**Provenance only — kept for traceability, no agent reads this to decide what to do:**
- `data/audit_discrepancy_log.md` — the original audit findings behind the task list. Every task's actual instructions live in the relevant agent's own `.md` file, not here. `schema-validator` uses this only as a *format template* for its own closing report — nothing appends to or modifies this file itself.

## Pass order — do not reorder

| Pass | Who | What |
|---|---|---|
| 0 | Coordinator | Confirm all 8 files under `data/sections/` are present. Confirm git-clean, commit. |
| 1 | `field-remover` → `contract-checker` | Deletions/renames (tasks 1, 2, 5, 10, 16) |
| 2 | `law-researcher` [mode: alcohol_fill] → `contract-checker` | Task 3a — runs immediately after Pass 1 |
| 3 | `field-reshaper` → `contract-checker` | Type/shape conversions (15, 17, 18, 19, 20, 22) |
| 4 | `distance-calculator` → `contract-checker` | Task 19a — runs immediately after Pass 3 |
| 5 | `seasons-reconciler` → `contract-checker` | Tasks 11, 11a (one dispatch, two internal steps) |
| 6 | `law-researcher` [mode: other_notable], `financial-researcher`, `geography-researcher` (parallel) → `contract-checker` | Tasks 4, 6, 6a, 7, 8, 12 |
| 7 | `category-auditor` ×2 (financial, then geography) | Tasks 9, 13 |
| 8 | Coordinator only | Schema reconciliation — collect every schema-gap flag from Passes 1–7, draft patches, get approval, write `city_schema_final.md` |
| 9 | `schema-validator` | Final gate against `city_schema_final.md`, rolls up all `contract-checker` results |

Passes 2 and 4 are dependent follow-ups to Passes 1 and 3 respectively — dispatch them immediately after, don't defer to Pass 6 just because they're both "research."

## Checkpoint policy
Pause after every pass and report a summary before starting the next one. Passes 2, 4, and 6 (anything with live research) get two checkpoints: canary batch first, then full batch. Never pause per-city or per-task within a pass. Never re-open a pass that already got a clean checkpoint — flag anything that surfaces later as a new finding instead.

## Before each dispatch
- Data-touching agents edit the relevant `data/sections/*.json` file(s) directly, in place. That's what the diff pane and the per-pass git-commit checkpoint are built around — a pass isn't reviewable or revertable unless the working copy actually changed. `passes/passN_<agent-name>.md` is for an agent's written *report* only (findings tables, manual-fix lists, schema-gap flags) — never a copy of the data itself.
- After the agent finishes, dispatch `contract-checker` with that pass's name so it checks the narrow, scripted contract before you present the diff for human review. Don't skip this even when the change looks obviously fine — that's the whole point of making it cheap and automatic.
- For `category-auditor` (Pass 7), you must supply the category name and its curated tier excerpt in your delegation message.

## After each pass
Report: what changed, how many records affected, `contract-checker`'s pass/fail summary, anything flagged for human review (including any "schema gap" reports — collect these, don't resolve them until Pass 8), and the output file path. Wait for explicit approval before starting the next pass.
