# City schema — research criteria (FINAL, post-Pass-8)

> This file is `city_schema_source_of_truth.md` plus the patches approved during Pass 8 schema reconciliation. It is what `schema-validator` (Pass 9) checks the finished dataset against. See "Pass 8 patch log" at the bottom for what changed and why.
>
> **Scope:** everything that genuinely varies city to city. Every city document is **independent and fully resolved** — no inheritance from a "host" city, no partial-override merging. A city links up to its state only for the fields listed in `state_schema.md` (statutory law + income tax) via `state_ref`, and for airport identity via `state_ref` → `major_airports`.
>
> **Not covered here:** residency-program relationships (which program a city hosts, or is a commutable suburb for). Those live in a separate collection keyed by the same city slug — unrelated to this livability research, do not include here.

## Identifiers

- `city` — string. City name (e.g. `"Los Angeles"`)
- `state_ref` — string. Slug of the parent state doc (required — resolves the invariant law + income tax fields)

## 1. amenities

```
in_n_out_present / costco_present / dutch_bros_present: {
  present: boolean,
  detail: string              // sourced narrative — evidence, address, citation tag
}
note: string, optional         // batch-level caveats
_meta: string, optional        // section-level completion/tracking note
```

No nearest-alternative field — presence/absence plus the sourced detail is sufficient; tracking the nearest alternative location adds research cost without adding decision-relevant information.

## 2. commute

- `ez_time_diff` — hours from Eastern business hours. Value must include the rubric descriptor, not just the raw offset.
  - rubric: 0h = ideal | 1–2h = fine | 3h = not ideal | 4–5h = not ideal (note if origin observes DST vs. not — e.g. AZ/HI — since the effective offset shifts seasonally relative to DST-observing zones)

## 3. law (city-variant only)

Flag only genuinely non-standard items; 1–2 sentences each, cite source. The remaining law categories (`gun_carry`, `cannabis`, `malpractice_tort_climate`, `driving`, `licensure`, `reproductive_family_health_law`, `state_income_tax`) are inherited from the parent state doc — **do not duplicate them here, in any city-schema section.**

- `alcohol_sales` — must explicitly address all three of: (1) grocery/off-premise sales, (2) on-premise/bar hours, (3) dry-county status (state even if the answer is simply "not dry")
- `other_notable` — catch-all, only if unusual — no filler entries; default `null` if nothing surfaces

## 4. financial (city-variant only)

`state_income_tax` is inherited from the parent state doc — **do not duplicate it here.**

- `local_hidden_fees` — sales tax, wage tax, local surtax
- `cost_of_living_index` — 2 adults + dog + cat, 100 = national avg
- `utilities_note` — flag if AC/heating load likely exceeds COL index
- `insurance_exposure` — renters/homeowners cost & availability given disaster risk
- `median_rent_1br` — typical 1BR asking rent, cite source and date
- `median_rent_2br` — typical 2BR asking rent, cite source and date
- `property_tax_rate` — effective rate, cite the computing jurisdiction/method (millage, assessment ratio, etc.)

## 5. geography

- `terrain_summary` — mountains/coast/flat/elevation
- `disaster_risk` — hurricane/wildfire/earthquake/tornado exposure
- `unique_facts` — freeform, 1–2 items max

Airport identity is defined in Section 9 (`visitability`); seasonal climate detail is defined in Section 6 (`ecosystem_climate`) — do not duplicate either here.

## 6. ecosystem_climate

- `pests` — common insects + disease vectors (ticks/mosquitoes: Lyme, West Nile, Valley fever, etc.)
- `skin_irritants` — humidity extremes, hard water, pollen, dry air (eczema-relevant)
- `air_quality` — wildfire smoke season, pollen index
- `daylight` — winter/summer avg hours
  - rubric: >10h winter & >14h summer = ideal | moderate swing = fine | <9h winter = not ideal
- `seasons` — variety/severity

## 7. household_lifestyle

- `partner_job_market` — note if applicable, else `"N/A"`
- `pet_logistics` — rental pet deposits/breed restrictions, vet cost/access, dog parks/trails
- `housing_snapshot` — typical rent near hospital, pet-friendly inventory

## 8. geography_context

General spatial orientation: where this city sits relative to things people actually care about.

- `proximity_highlights` — array of `{place: string, distance_mi: number | null, note: string}` (beaches, national parks, ski areas, etc.). Must be a structured array, not prose. `distance_mi` is `null` when a place is named without an explicit distance in the source material — still include the entry rather than dropping it. **(Pass 8 patch — see log)**
- `regional_centrality` — freeform, e.g. `"central to the LA/San Diego/Las Vegas triangle"`

## 9. visitability

- `nearest_airport` — `{airport_code: string, airport_name: string, distance_mi: number, drive_time_min: number}`. `airport_code` references an entry in this city's own state's `major_airports`.
- `distance_from_anchors` — array of `{anchor_label: string, anchor_airport_code: string, distance_mi: number, drive_time_hr: number | null, flight_time_hr: number}`. Must be a structured array, not prose. `anchor_airport_code` resolves through *the anchor's own* state's `major_airports` list — so an anchor can be a specific city (`"SF"` → `SFO`) or just a state (`"VA"` → `IAD`) without needing to pin an exact city. `drive_time_hr` must be populated or explicitly `null` (e.g. transoceanic/impractical routes), never omitted.
- `nearby_pois` — array of strings, short — things to do when family visits. Must be a structured list, not prose.

## 10. connectivity

- `cell_coverage` — carrier coverage quality, flag dead zones
- `broadband_availability` — fiber/cable/DSL/satellite options, typical speed
- `note` — flag if relevant for remote work or a telehealth-dependent lifestyle

## _meta

Same conventions as the state schema:
- `null` (not a guess) for anything unverified
- cite source inline per field
- flag anything time-sensitive as `"verify"`
- tag format: `[tier|T/F|status|date]`
- canonical tier tokens: `Tier1` | `Tier2` | `Tier3` | `computed` — no space between "Tier" and the number; `computed` replaces a tier for internally-derived/calculated facts (time-zone math, great-circle distance) that have no external citation to grade

---

## Pass 8 patch log

Three items approved out of the ten schema-gap flags collected across Passes 1–7 (full list of all ten is in the Pass 8 coordinator report in the session transcript; the other seven remain open/unresolved, not applied here, and can be revisited later if needed):

1. **`geography_context.proximity_highlights[].distance_mi` — `number` → `number | null`.**
   Flagged by `field-reshaper` (Pass 3): the source prose regularly names a place with no explicit distance attached, and forcing a distance to preserve the entry was worse than allowing `null`. Applied to Section 8 above (this was already treated as an accepted deviation during Pass 3's own contract-checker runs and `category-auditor`'s Pass 7 review — this patch formalizes what the data already does in ~20 of 39 records).

2. **Tier-tag spacing (`[Tier1|...]` vs `[Tier 1|...]`) — no schema text change; documented as accepted legacy condition.**
   The `_meta` canonical format (line above) already mandates no-space and always has. What Pass 6/7 found is that a large share of the *pre-existing, pre-pipeline* dataset uses `[Tier N|...]` with a space instead. Reformatting the entire dataset to match the documented canonical form was judged out of scope for this pipeline (would touch hundreds of fields unrelated to any of the 24 tasks in scope). `contract-checker`'s assertions were updated (in `output_contracts.md`) to accept either form so this legacy drift doesn't block finalization. No further action is planned against this inconsistency within this pipeline's scope.

3. **Housekeeping (not a schema-file change): `field-reshaper.md` agent definition corrected.**
   The agent file instructed "never overwrite the working copy in place; write to a new pass-output file," directly contradicting this project's `CLAUDE.md`, which requires in-place edits so the diff-pane/git-commit checkpoint workflow functions. This had to be manually overridden in the delegation message on every Pass 3 dispatch. Fixed directly in `.claude/agents/field-reshaper.md` (see that file's own history) so future runs of this pipeline don't need the override.

**Not approved / left open** (from the full ten-item list presented at Pass 8): `visitability.nearest_airport` optional `airport_selection_note` field; `distance_from_anchors[].flight_time_hr` same-metro-anchor floor-value guidance; `financial.cost_of_living_index` aggregator-cluster-fallback acknowledgment; `financial.utilities_note` formal method definition; `_meta`'s `"null - <reason> [tier]"` string convention; connectivity-framing-facts field question; `source_tier_list.md`'s TWIA-as-Tier1-equivalent addendum.
