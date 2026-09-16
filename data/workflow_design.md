# Coordinator/Subagent Workflow Design (v2)
Neurology Residency Cities Dataset — schema finalization pipeline

**Scope:** covers the 24 tasks in the table below, with tasks 3 and 11 each split into two per the notes further down, and task 6a added alongside task 6 to cover a data gap (`financial.cost_of_living_index` null in 10/39 records) that the original task list missed. Per earlier instruction, `commute`, `connectivity`, and `_meta`/tag-formatting are explicitly out of scope for this entire pipeline.

**Coordinator = the main Claude Code session itself**, not a subagent. It holds all reference files, dispatches subagents in the order below, and is the only thing that talks to you directly.

**v2 changes from the original design:** (1) resequenced so a task and its dependent follow-up run back to back instead of being separated by unrelated passes, (2) split two tasks that were bundling a mechanical step and a research step into one blob, (3) added a deterministic per-pass validator (`contract-checker`) so "did this pass work" is a scripted check, not an LLM opinion, (4) added `output_contracts.md` and `golden_record_atlanta.md` as the explicit, checkable definition of "correct" that both `contract-checker` and `schema-validator` check against, (5) simplified the amenities object shape (dropped `nearest_alt`).

---

## 1. Task classification

| # | Field | Bucket | Assigned agent | Pass |
|---|---|---|---|---|
| 1 | `law.state_income_tax` (remove) | SCRIPT | `field-remover` | 1 |
| 2 | `law.alcohol_sales_off_premise` → `alcohol_sales` (rename) | SCRIPT | `field-remover` | 1 |
| 3 | `law.alcohol_sales` placement | *No separate work — task 2's rename accomplishes this. Kept as a numbered item only for lineage tracking.* | `field-remover` | 1 |
| 3a | `law.alcohol_sales` content fill (bar hours, dry county) | RESEARCH | `law-researcher` (mode: alcohol_fill) | 2 — **immediately follows Pass 1**, not bundled with unrelated research |
| 4 | `law.other_notable` | RESEARCH | `law-researcher` (mode: other_notable) | 6 — no dependency, batched with other independent research |
| 5 | `financial.state_income_tax_rate` (remove) | SCRIPT | `field-remover` | 1 |
| 6 | `financial.local_hidden_fees` | RESEARCH | `financial-researcher` | 6 |
| 6a | `financial.cost_of_living_index` (null-fill, 10/39 records) | RESEARCH | `financial-researcher` | 6 — batched with 6/7/8, same agent/pass |
| 7 | `financial.utilities_note` | RESEARCH | `financial-researcher` | 6 |
| 8 | `financial.insurance_exposure` | RESEARCH | `financial-researcher` | 6 |
| 9 | financial accuracy audit | AUDIT (content) | `category-auditor` (param: financial) | 7 |
| 10 | `geography.nearest_airport` (remove) | SCRIPT | `field-remover` | 1 |
| 11 | `geography.seasons` vs `ecosystem_climate.seasons` — compare/merge, write | JUDGMENT | `seasons-reconciler` | 5 |
| 11a | `geography.seasons` — remove after merge | *Mechanical, but kept as `seasons-reconciler`'s own last step rather than a separate dispatch — see rationale below* | `seasons-reconciler` | 5 |
| 12 | `geography.unique_facts` | RESEARCH | `geography-researcher` | 6 |
| 13 | geography accuracy audit | AUDIT (content) | `category-auditor` (param: geography) | 7 |
| 14 | `household_lifestyle` | NONE | — | — |
| 15 | `amenities.*_present` (retype, simplified — no `nearest_alt`) | SCRIPT+REVIEW | `field-reshaper` | 3 |
| 16 | `amenities.note` (remove) | SCRIPT | `field-remover` | 1 |
| 17 | `visitability.nearest_airport.distance_mi`/`.drive_time_min` (retype) | SCRIPT+REVIEW | `field-reshaper` | 3 |
| 18 | `visitability.nearest_airport.airport_name` (trim) | SCRIPT+REVIEW | `field-reshaper` | 3 |
| 19 | `visitability.distance_from_anchors` (retype) | SCRIPT+REVIEW | `field-reshaper` | 3 |
| 19a | `distance_from_anchors[].flight_time_hr`/`drive_time_hr` (fill) | RESEARCH (Tier 0) | `distance-calculator` | 4 — **immediately follows Pass 3** |
| 20 | `visitability.nearby_pois` (retype) | SCRIPT+REVIEW | `field-reshaper` | 3 |
| 21 | `ecosystem_climate` | NONE | — | — |
| 22 | `geography_context.proximity_highlights` (retype) | SCRIPT+REVIEW | `field-reshaper` | 3 |
| 23 | `geography_context.regional_centrality` | NONE | — | — |

### Why 3/3a split but 11/11a didn't split into separate dispatches

Both look like the same pattern (a task plus a trailing cleanup) but the cost profile is opposite:

- **3 → 3a**: task 2's mechanical rename is instant (one script, all 39 records, done in Pass 1). Task 3a's research fill is slow and expensive (live search, batched, canary-tested). Bundling them would mean either delaying the trivial rename behind slow research, or doing the research too early/rushed to fit a "mechanical pass" mold. Splitting them into separate dispatches, sequenced back-to-back, gets the fast part done immediately and lets the slow part run properly.
- **11 → 11a**: the comparison/merge (11) and the deletion (11a) happen on the same record, in the same agent run, with no meaningful time or cost gap between them. There's no efficiency gained by making the coordinator spin up `field-remover` again for a single trivial key deletion right after `seasons-reconciler` already touched that exact record. They're numbered separately because they're logically distinct steps worth tracking, but they stay one dispatch.

### Why task 6a exists

`audit_discrepancy_log.md` (row 19) already flagged `financial.cost_of_living_index` as `null` in 10/39 records — a genuine `DATA-FILL` gap, same category as tasks 6/7/8. It never made it into the original 23-task list; a later pipeline review caught the gap by diffing the log against the task table. It's `financial-researcher`'s field and `financial-researcher` is already dispatched in Pass 6, so 6a batches in alongside 6/7/8 rather than getting its own pass — same reasoning as why 11/11a didn't split into separate dispatches.

## 2. Agent roster

| Agent | Tools | Model | Invocation pattern |
|---|---|---|---|
| `field-remover` | Read, Write, Bash, Grep | haiku | Once — script handles all 39 records in one pass |
| `field-reshaper` | Read, Write, Bash, Grep | sonnet | Once — script + manual fix-up |
| `law-researcher` | Read, Edit, WebSearch, WebFetch | sonnet | Twice — mode: alcohol_fill (Pass 2), mode: other_notable (Pass 6) |
| `distance-calculator` | Read, Edit, WebSearch, WebFetch | sonnet | Batched, ~5–8 cities/invocation, runs right after Pass 3 |
| `seasons-reconciler` | Read, Write, Edit, WebSearch, WebFetch | sonnet | Once — all 39 comparisons in one pass |
| `financial-researcher` | Read, Edit, WebSearch, WebFetch | sonnet | Batched, ~5 cities/invocation |
| `geography-researcher` | Read, Edit, WebSearch, WebFetch | sonnet (haiku viable) | Batched, ~5–8 cities/invocation |
| `category-auditor` | Read, Grep | sonnet | Twice — once per category, read-only, content-accuracy. No WebSearch by design — Tier 2 audits against model knowledge only, never live search (see §3) |
| `contract-checker` | Read, Bash, Grep | haiku | Once after every pass that touches any `data/sections/*.json` file — structural, script-only |
| `schema-validator` | Read, Bash, Grep | sonnet (opus optional) | Once, final gate, rolls up every `contract-checker` result |

## 3. The three-tier audit system

A single "auditor agent" checking everything with LLM judgment has a real failure mode: it can rubber-stamp something because it *reads* plausible, without ever actually verifying it. The fix is splitting audit work by what's actually checkable how:

| Tier | What it checks | How | Agent | Hallucination risk |
|---|---|---|---|---|
| 1 | Structure: keys present, correct types, barred fields actually gone, counts match | Deterministic script, hard assertions from `output_contracts.md` | `contract-checker` | None — it's a script, not a judgment |
| 2 | Content plausibility: does a claim hold up against what the model already knows | Comparison against model knowledge — no live search. Three outcomes per claim: consistent/plausibly time-drifted → trust; contradicts known facts → flag; no relevant knowledge either way → explicitly "unverifiable," not silently folded into "clean" | `category-auditor` | Real and not fully mitigated — this tier trades verification rigor for speed and cost. It catches claims that clearly contradict what the model knows, and it's honest about the cases it can't speak to at all (the "unverifiable" bucket), but it cannot catch a plausible-sounding, specific, wrong fact the model has no independent knowledge of either way. That gap is a deliberate tradeoff, not an oversight — worth remembering when deciding how much weight a "clean" verdict from this tier should carry. |
| 3 | Final gate: does the whole dataset satisfy the full contract, including any schema patches from Pass 8 | Script + rollup of every prior `contract-checker` result | `schema-validator` | None on the structural half; inherits Tier 2's residual risk on the content half |

`output_contracts.md` and `golden_record_atlanta.md` exist so Tier 1 and Tier 3 have something concrete to check against instead of "does this look done." Tier 2 doesn't get that same guarantee — content accuracy genuinely can't be fully scripted, and this design deliberately doesn't spend search budget re-deriving it either. The mitigation that's left is disclosure: every Tier 2 verdict is one of clean / flagged / unverifiable, never collapsed into a single "looks fine," so a "clean, unverified against model knowledge" result downstream doesn't get mistaken for "clean, independently confirmed."

## 4. Pass sequence and checkpoints

| Pass | Agent(s) | Tasks | Checkpoint |
|---|---|---|---|
| 0 — Baseline | Coordinator only | — | Confirm all 8 files under `data/sections/` are present, git-initialized, commit. |
| 1 — Field removal | `field-remover` → `contract-checker` | 1, 2, 5, 10, 16 | **Pause.** First run of a new agent — review diff across the 4 section files it touched, approve. |
| 2 — Alcohol content fill | `law-researcher` (alcohol_fill) → `contract-checker` | 3a | **Pause (canary, then full).** Runs immediately after Pass 1 since it depends on the rename. |
| 3 — Field reshaping | `field-reshaper` → `contract-checker` | 15, 17, 18, 19, 20, 22 | **Pause.** Review diff + manually-fixed records specifically. |
| 4 — Distance/flight time fill | `distance-calculator` → `contract-checker` | 19a | **Pause (canary, then full).** Runs immediately after Pass 3 since it depends on the restructure. |
| 5 — Seasons reconciliation | `seasons-reconciler` → `contract-checker` | 11, 11a | **Pause.** Read the per-city decision table (39 short rows) before it finalizes. |
| 6 — Independent research | `law-researcher` (other_notable), `financial-researcher`, `geography-researcher` — parallel → `contract-checker` | 4, 6, 7, 8, 12 | **Pause twice** — canary batch (2–3 cities), then full batch. |
| 7 — Category audits | `category-auditor` ×2 (financial, geography) | 9, 13 | **Pause.** Route findings back to the relevant researcher for a fix-up batch. |
| 8 — Schema reconciliation | Coordinator only | — | **Pause — every proposed patch needs your yes/no.** Produces `city_schema_final.md`. |
| 9 — Final audit | `schema-validator` | full-dataset check | **Pause — finalization gate.** Rolls up all `contract-checker` results + its own final structural pass against `city_schema_final.md`, cross-checked structurally against `golden_record_atlanta.md`. |

Don't pause within a pass (per-city). Do pause once at the end of every pass, plus the extra canary pause in Passes 2, 4, and 6 (any pass with live research). If a pass comes back clean, don't re-open it later.

### Pass 8 in detail — why it exists

Every data-touching agent can hit a case the schema doesn't address — not missing data, a genuine gap in what the schema *says*. The first one surfaced just from designing `field-reshaper`: `geography_context.proximity_highlights[].distance_mi` is typed `number` but some places genuinely need `null`. See `golden_record_atlanta.md` for the worked Atlanta example of this exact case.

Every data-touching agent (all except `field-remover`) has a "If you hit a schema gap" instruction: report it, don't route around it silently.

**What the coordinator does in Pass 8:**
1. Collect every schema-gap flag reported across Passes 1–7.
2. Draft the specific patch to `city_schema_source_of_truth.md` that resolves each one.
3. Present the full batch of proposed patches for approval — not auto-applied.
4. Write `data/city_schema_final.md` = `city_schema_source_of_truth.md` + approved patches.

If zero gaps were flagged, `city_schema_final.md` is still produced (a copy, with a note that no patches were needed) so Pass 9 always has an explicit file to point at.

## 5. Recommended project structure

```
project-root/
  CLAUDE.md
  .claude/agents/
    field-remover.md
    field-reshaper.md
    law-researcher.md
    distance-calculator.md
    seasons-reconciler.md
    financial-researcher.md
    geography-researcher.md
    category-auditor.md
    contract-checker.md
    schema-validator.md
  data/
    sections/                          # OPERATIONAL — the actual working data, one file per schema section
      law.json
      financial.json
      geography.json
      ecosystem_climate.json
      household_lifestyle.json
      amenities.json
      visitability.json
      geography_context.json
    cities_final.json                  # OPERATIONAL — doesn't exist until Pass 9 assembles it from all 8 section files. The actual deliverable.
    city_schema_source_of_truth.md     # OPERATIONAL
    city_schema_final.md               # OPERATIONAL — produced by Pass 8
    output_contracts.md                # OPERATIONAL — assertions now scoped per section file, not one shared file (needs updating, see §6)
    golden_record_atlanta.md           # OPERATIONAL — needs restructuring into per-section examples (see §6)
    source_tier_list.md                # OPERATIONAL
    audit_discrepancy_log.md           # PROVENANCE ONLY — no agent reads this for instructions
  passes/                              # agent-written pass reports (.md) only — never a copy of section-file data
```

**Each `data/sections/<name>.json` is an array of 39 records, shaped as:**
```json
{
  "city_ref": "atlanta-ga",
  "city": "Atlanta",
  "state_ref": "georgia",
  "residency_context": { /* untouched by this pipeline — carried through as-is */ },
  "<section_name>": { /* the one section this file owns */ }
}
```

No single file gets progressively mutated across the whole pipeline anymore. Each section-scoped agent reads and writes only the section file(s) its job actually touches. There is no working `cities.json` — the closest equivalent, `cities_final.json`, is a terminal *output* produced once, at Pass 9, by joining all 8 section files on `city_ref`. Nothing upstream of Pass 9 depends on it existing.

## 6. Impact of the section-file split — high-level notes

This is a real architecture change, not just a file rename. Noting what it touches without re-implementing every agent yet:

- **`field-remover`** now edits 4 section files per pass (`law.json`, `financial.json`, `geography.json`, `amenities.json`) instead of 1. Same script-based approach — just multiple target files.
- **`field-reshaper`** now edits 3 section files (`amenities.json`, `visitability.json`, `geography_context.json`).
- **`seasons-reconciler`** was already conceptually cross-section (compares `geography.seasons` against `ecosystem_climate.seasons`); now it literally opens two files (`geography.json`, `ecosystem_climate.json`) instead of two keys in one record. No logic change, just where it's reading from.
- **`category-auditor`** gains a real new responsibility: any cross-section consistency check (e.g. financial's `insurance_exposure` against geography's `disaster_risk`) now requires it to open multiple section files and join them by `city_ref` itself — that join used to be free (same record, same file).
- **`contract-checker` / `output_contracts.md`**: every assertion needs to name which section file it applies to. A "grep the output file" instruction from the old design is now ambiguous — needs to become "grep `data/sections/law.json`," etc.
- **`schema-validator` (Pass 9)** gains a new first step: assemble all 8 section files into `data/cities_final.json` by `city_ref`, *then* validate the assembled view. This is where the deliverable actually gets produced — it didn't exist as a discrete step before.
- **`golden_record_atlanta.md`** should become 8 small per-section examples instead of one combined record, matching the new file layout. Not rebuilt yet — flagged as a follow-up.
- **Pass 0** simplifies: no more "copy baseline into a working file," since there's no working file to seed. It just confirms all 8 section files are present under `data/sections/`. The `baseline_cities_data_v1.json` concept becomes redundant once there's no single file to protect from in-place edits — git's initial commit already preserves the untouched starting state on its own.
- **New risk this split introduces, not present before:** the 8 section files can drift out of sync — a city present in `law.json` but missing from `visitability.json`, or a `city_ref` typo that only shows up in one file. This wasn't possible when everything lived in one record. Worth a cheap `contract-checker` assertion, checked after *every* pass, not just at the end: all 8 section files contain the exact same 39 `city_ref` values, no more, no fewer.
