# Pass 9 — Final Discrepancy Log
Checked against: `data/city_schema_final.md` (Pass 8 reconciled schema, as amended by the Pass 9 patch log entry for `proximity_highlights[].note`).
Scope: all 39 assembled records in `data/cities_final.json`, every section except `commute`, `connectivity`, and `_meta`.
Log format matches `data/audit_discrepancy_log.md` (columns/structure only) — this is a new file, the original is untouched.

**Status key** (same as source template)
- `DECIDED` — resolved this session
- `NEEDS CONFIRMATION` — recommendation, not yet approved
- `DATA-FILL` — schema/structure fine, content missing
- `DATA-RESTRUCTURE` — schema fine, content needs re-shaping, no new research

---

## A. Section 8 — geography_context (prior finding, now resolved)

| # | Field | Issue Type | Current State | Expected / Decision | Rule Violated | Status |
|---|-------|-----------|----------------|----------------------|----------------|--------|
| 1 | `geography_context.proximity_highlights[].note` | ~~Wrong type (null vs string)~~ — RESOLVED | 9 of 85 total `proximity_highlights` entries across 7 records (`charlottesville-va`, `durham-nc`, `palo-alto-ca` x2, `salisbury-md`, `san-francisco-ca`, `winston-salem-nc` x2, `woodbridge-ct`) have `note: null` | `city_schema_final.md` §8 now defines `note: string \| null` (Pass 9 patch log item 4, added after this finding was first surfaced) — same nullability pattern already approved for the sibling field `distance_mi` (Pass 8 patch log item 1). No data changes were made or needed; the 9 `null` values are schema-compliant as written | None — schema was patched to match legitimate existing data, per the same reasoning already accepted for `distance_mi` | DECIDED — resolved via schema patch, re-verified 0/85 violations on re-check |

---

## B. All sections/fields — clean

No discrepancies found for `law`, `financial`, `geography`, `ecosystem_climate`, `household_lifestyle`, `amenities`, `visitability`, or `geography_context` (`proximity_highlights[].place`, `proximity_highlights[].distance_mi`, `proximity_highlights[].note`, `regional_centrality`), across all 39 records.

- Barred/duplicate fields (`law.state_income_tax`, `financial.state_income_tax_rate`, `geography.nearest_airport`, `geography.seasons`, `amenities.note`, `amenities.*.nearest_alt`) are verifiably absent in all 39 records — 0 occurrences found by script.
- `proximity_highlights[].note`: re-checked against the patched schema — 9/85 entries `null` (unchanged data, now schema-compliant), 76/85 populated strings, 0 wrong-typed entries, 0 missing `place`.
- `proximity_highlights[].distance_mi`: 64/85 `null` (schema-compliant per Pass 8 patch log item 1), rest populated numbers.
- Section key-shape (field presence, no extras/missing) verified 39/39 against `city_schema_final.md` for every in-scope section, and structurally cross-checked against `data/golden_record_atlanta.md`'s shape for `amenities.*` (`present`/`detail`), `visitability.nearest_airport`, and `visitability.distance_from_anchors[]`.
- Top-level record shape (`amenities, city, city_ref, ecosystem_climate, financial, geography, geography_context, household_lifestyle, law, residency_context, state_ref, visitability`) verified 39/39.
- 0 remaining `<PENDING RESEARCH>` markers anywhere in `data/cities_final.json`.

Tier-tag spacing (`[Tier1|...]` vs `[Tier 1|...]`) was observed throughout the dataset but is not logged here — per Pass 8 patch log item 2, this is a documented accepted legacy condition, not a defect, and is explicitly out of scope for this log.

**Net result: this log is empty of open findings.** The single discrepancy found on the prior Pass 9 run was closed by a schema patch (not a data change), and re-validation against the patched schema confirms 0 remaining violations anywhere in scope.
