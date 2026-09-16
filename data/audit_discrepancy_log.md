# Audit Discrepancy Log — Neurology Residency Cities Dataset
Baseline diffed against: `baseline_city_schema_v1.md` + `baseline_cities_data_v1.json` (checksums verified, untouched)
Log locked as of this pass — do not re-derive from raw file re-reads; work from this table per Process step 2.

**Status key**
- `DECIDED` — resolved this session, schema updated, ready to apply to data
- `NEEDS CONFIRMATION` — my recommendation, not yet explicitly approved
- `DATA-FILL` — schema/structure is fine, content is simply missing → future research pass
- `DATA-RESTRUCTURE` — schema is fine, existing content just needs re-shaping (no new research)

---

## A. Cross-cutting

| # | Field | Issue Type | Current State | Expected / Decision | Rule Violated | Status |
|---|-------|-----------|----------------|----------------------|----------------|--------|
| 1 | `residency_context` (top-level) | Out-of-scope field | Present 39/39 | Stays in data file; formally out of *this* schema's scope | Schema's "Not covered here" note | DECIDED — no action, confirmed fine |

## B. Section 1 — amenities

| # | Field | Issue Type | Current State | Expected / Decision | Rule Violated | Status |
|---|-------|-----------|----------------|----------------------|----------------|--------|
| 2 | `in_n_out_present`, `costco_present`, `dutch_bros_present` | Wrong type | String sentence w/ inline citation, 39/39 | Convert to object: `{present: bool, detail: string, nearest_alt: string\|null}` | Schema: "Boolean + nearest-alternative if absent" | DECIDED — schema updated |
| 3 | `note` | Undocumented field | Present 39/39 (batch-caveat text) | Keep; formalize as optional section field | n/a (extension) | DECIDED — added to schema |
| 4 | `_meta` | Undocumented field | Present 39/39 (batch-completion tracking) | Keep; formalize as optional section field | n/a (extension) | DECIDED — added to schema |

## C. Section 2 — commute

| # | Field | Issue Type | Current State | Expected / Decision | Rule Violated | Status |
|---|-------|-----------|----------------|----------------------|----------------|--------|
| 5 | `ez_time_diff` | Missing rubric label | 39/39 give hour offset only (e.g. `"0h (Eastern)"`), no ideal/fine/not-ideal descriptor | Append rubric descriptor to every value | Schema rubric requires ideal/fine/not-ideal classification | DECIDED — schema now requires the label explicitly |
| 6 | `ez_time_diff` rubric itself | Incomplete rubric | Honolulu = 5h; Phoenix/Scottsdale = 2h w/ no-DST caveat — both outside the defined 0/1–2/3h buckets | Extend rubric to cover 4–5h+ and note DST-observance edge cases | Rubric silent above 3h | DECIDED — schema rubric extended |

## D. Section 3 — law

| # | Field | Issue Type | Current State | Expected / Decision | Rule Violated | Status |
|---|-------|-----------|----------------|----------------------|----------------|--------|
| 7 | `state_income_tax` | Field shouldn't exist here at all | Present 39/39 | Remove entirely | Inherited-from-state field, barred in both `law` and `financial` | DECIDED (per your instruction) — remove |
| 8 | `alcohol_sales_off_premise` | Wrong field name | Present 39/39 | Rename → `alcohol_sales` | Schema field name is `alcohol_sales` | DECIDED (per your instruction) — rename |
| 9 | `alcohol_sales` (post-rename) | Incomplete content — bar hours | Only 6/39 mention on-premise/bar hours | Re-research to cover on-premise hours for all 39 | "bar hours" is a required sub-topic | DATA-FILL |
| 10 | `alcohol_sales` (post-rename) | Incomplete content — dry counties | 0/39 address dry-county status | Re-research to explicitly confirm dry/not-dry | "dry counties?" is a required sub-topic | DATA-FILL |
| 11 | `other_notable` | Missing required field | 0/39 present | Add field, default `null`, fill only if something unusual surfaces | Required key per schema | DATA-FILL (mechanical default can be applied immediately, content fill is separate) |

## E. Section 4 — financial

| # | Field | Issue Type | Current State | Expected / Decision | Rule Violated | Status |
|---|-------|-----------|----------------|----------------------|----------------|--------|
| 12 | `state_income_tax_rate` | Duplicate + prohibited | Present 39/39, duplicates removed `law.state_income_tax` | Remove | Inherited-field duplication rule | DECIDED (per your instruction) — remove |
| 13 | `median_rent_1br` | Undocumented field | Present 39/39 | Keep; formalize in schema | n/a (extension) | DECIDED (per your instruction) — added to schema |
| 14 | `median_rent_2br` | Undocumented field | Present 39/39 | Keep; formalize in schema | n/a (extension) | DECIDED — added to schema |
| 15 | `property_tax_rate` | Undocumented field | Present 39/39 | Keep; formalize in schema | n/a (extension) | DECIDED — added to schema |
| 16 | `local_hidden_fees` | Missing required field | 0/39 | Research fill | Required key | DATA-FILL |
| 17 | `utilities_note` | Missing required field | 0/39 | Research fill | Required key | DATA-FILL |
| 18 | `insurance_exposure` | Missing required field | 0/39 | Research fill | Required key | DATA-FILL |
| 19 | `cost_of_living_index` | Incomplete data | `null` in 10/39 (Columbia, Farmington, Folsom, Germantown, Greenville, Palo Alto, Salisbury, Scottsdale, Tacoma, Woodbridge) | Research fill | Required key should be populated | DATA-FILL |

## F. Section 5 — geography

| # | Field | Issue Type | Current State | Expected / Decision | Rule Violated | Status |
|---|-------|-----------|----------------|----------------------|----------------|--------|
| 20 | `unique_facts` | Missing required field | 0/39 | Research fill | Required key | DATA-FILL |
| 21 | `nearest_airport` | Misplaced duplicate | Present 39/39, verified identical (no conflicts) to `visitability.nearest_airport` | Remove from `geography`, keep single copy in `visitability` | Not a `geography` field; belongs to `visitability` | DECIDED (confirmed) — schema updated |
| 22 | `seasons` | Misplaced duplicate | Present 39/39, duplicates `ecosystem_climate.seasons` | Remove from `geography` | Belongs to `ecosystem_climate`, not `geography` | DECIDED (confirmed) — schema updated |

## G. Section 8 — geography_context

| # | Field | Issue Type | Current State | Expected / Decision | Rule Violated | Status |
|---|-------|-----------|----------------|----------------------|----------------|--------|
| 23 | `proximity_highlights` | Wrong type | Flat prose string, 39/39 | Restructure into array of `{place, distance_mi, note}` — same underlying facts, no new research | Schema already specifies array shape | DATA-RESTRUCTURE (schema unchanged, already correct) |

## H. Section 9 — visitability

| # | Field | Issue Type | Current State | Expected / Decision | Rule Violated | Status |
|---|-------|-----------|----------------|----------------------|----------------|--------|
| 24 | `distance_from_anchors` | Wrong type | Flat `\|`-joined string, 39/39; anchor set (SF/VA/Phoenix) is at least consistent city-to-city | Restructure into array of 5-key objects per schema | Schema already specifies array shape | DATA-RESTRUCTURE |
| 25 | `distance_from_anchors[].drive_time_hr` | Missing sub-value | Never populated, 0/39 (only flight time + great-circle miles given) | Fill where meaningful; `null` where a drive is genuinely impractical (e.g. Hawaii) | Required sub-field per schema | DATA-FILL |
| 26 | `nearby_pois` | Wrong type | Flat string, 39/39 | Restructure into list | Schema says "freeform list" | DATA-RESTRUCTURE |
| 27 | `nearest_airport.airport_name` | Undocumented sub-field | Present 39/39 | Keep; formalize in schema | n/a (extension) | DECIDED — added to schema (low-risk, flag if you'd rather drop it) |

## I. Section 10 — connectivity

| # | Field | Issue Type | Current State | Expected / Decision | Rule Violated | Status |
|---|-------|-----------|----------------|----------------------|----------------|--------|
| 28 | `cell_coverage`, `broadband_availability` | Missing content (not a schema mismatch) | `null` in 39/39 | Research fill (opportunistic, per existing in-data notes) | Required keys | DATA-FILL — schema is correct as-is |

## J. Section 11 — _meta / tagging

| # | Field | Issue Type | Current State | Expected / Decision | Rule Violated | Status |
|---|-------|-----------|----------------|----------------------|----------------|--------|
| 29 | Tier tags | Formatting inconsistency | `Tier1`/`Tier 1`, `Tier2`/`Tier 2`, `Tier3`/`Tier 3` all used, roughly 40–50% split each | Normalize to single canonical token: `Tier1`/`Tier2`/`Tier3` (no space), global deterministic find-replace | Tag format should be consistent per `[tier\|T/F\|status\|date]` spec | DECIDED — mechanical, zero accuracy risk |
| 30 | `computed` tag | Undocumented tag value | Present 156x, not in tier vocabulary | Add as officially recognized tag alongside Tier1/2/3, for internally-derived (non-cited) facts | n/a (extension) | DECIDED (per your instruction) — added to schema |

---

## Summary counts
- **Decided this session** (ready to apply): rows 1–8, 12–15, 21–22, 27, 29–30 → **16 items**
- **Needs your confirmation before applying**: none remaining
- **Data-restructure** (mechanical, no research needed): rows 23–24, 26 → **3 items**
- **Data-fill** (genuine research gaps for a future pass): rows 9–11, 16–20, 25, 28 → **10 items**

All open questions resolved as of this update. Next step per your Process: batch-apply the `DECIDED` rows first (Pass 1), since they're pure structural edits with no ambiguity, then re-validate before touching `DATA-RESTRUCTURE` rows, then `DATA-FILL` last.
