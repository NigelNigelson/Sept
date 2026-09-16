# Output Contracts — per pass

Every assertion here is checkable by a script — no judgment calls. If you find yourself wanting to write "value seems reasonable" as an assertion, it doesn't belong in this file; that's `category-auditor`'s job, not `contract-checker`'s. This file is what makes "the pipeline is done" a checkable claim instead of a vibe.

Applies to all 39 records unless stated otherwise. `commute`, `connectivity`, `_meta` are out of scope for every pass below — no assertion here should ever touch them.

---

## Pass 1 — field-remover
- [ ] `law.state_income_tax` key absent from all 39 records
- [ ] `law.alcohol_sales_off_premise` key absent from all 39 records
- [ ] `law.alcohol_sales` key present in all 39 records, type `string`
- [ ] `financial.state_income_tax_rate` key absent from all 39 records
- [ ] `geography.nearest_airport` key absent from all 39 records
- [ ] `amenities.note` key absent from all 39 records
- [ ] Record count unchanged: 39
- [ ] No keys other than the 5 above changed anywhere in the file (diff against the Pass-0 baseline should show exactly these 5 key-level changes and nothing else)

## Pass 2 — law-researcher (mode: alcohol_fill)
- [ ] `law.alcohol_sales` value, for all 39 records, contains a substring plausibly addressing on-premise/bar hours (keyword check only — e.g. "bar", "on-premise", "on premise"; this is a completeness check, not an accuracy check)
- [ ] `law.alcohol_sales` value, for all 39 records, contains a substring plausibly addressing dry-county status (keyword check only — e.g. "dry", "not dry")
- [ ] `law.alcohol_sales` value, for all 39 records, ends with a `[TierN|...]` tag matching the existing tag-format regex
- [ ] `law.other_notable` key: still absent or unchanged from Pass 1 state (not this pass's job — Pass 6 fills it)

*Keyword-presence checks here confirm the topic was addressed, not that the content is correct — that distinction matters, and `contract-checker`'s report should say so explicitly rather than implying "bar hours: PASS" means "bar hours: verified accurate."*

## Pass 3 — field-reshaper
- [ ] `amenities.in_n_out_present`, `costco_present`, `dutch_bros_present`: for all 39 records, type `object`, keys exactly `{present, detail}` (no more, no fewer — `nearest_alt` must NOT be present), `present` type `boolean`, `detail` type `string`
- [ ] `visitability.nearest_airport.distance_mi`: type `number`, all 39
- [ ] `visitability.nearest_airport.drive_time_min`: type `number`, all 39
- [ ] `visitability.nearest_airport.airport_name`: type `string`, under 80 characters (flag, don't hard-fail, anything longer as a likely un-trimmed commentary blob)
- [ ] `visitability.distance_from_anchors`: type `array`, length 3, all 39
- [ ] Each element of `distance_from_anchors`: keys exactly `{anchor_label, anchor_airport_code, distance_mi, drive_time_hr, flight_time_hr}`; `distance_mi` type `number`; `drive_time_hr` and `flight_time_hr` both `null` at this stage (Pass 4 fills them — a non-null value here means distance-calculator ran out of order)
- [ ] `visitability.nearby_pois`: type `array` of `string`, length ≥ 1, all 39
- [ ] `geography_context.proximity_highlights`: type `array` of objects with keys exactly `{place, distance_mi, note}`, all 39. `distance_mi` may be `number` or `null` — this is a known, approved schema exception (see Pass 8), not a failure

## Pass 4 — distance-calculator
- [ ] `distance_from_anchors[].drive_time_hr`: for all 39 × 3 anchors, type `number` or explicit `null` — never the JSON-missing/undefined state
- [ ] `distance_from_anchors[].flight_time_hr`: for all 39 × 3 anchors, type `number`. If any record has `null` here, that's a new schema-gap flag, not a silent pass — report it, don't auto-fail or auto-pass
- [ ] Neither sub-field carries a `[TierN|...]` tag anywhere (Tier 0 — citation-free by design; a tag here is itself a failure)

## Pass 5 — seasons-reconciler
- [ ] `geography.seasons` key absent from all 39 records
- [ ] `ecosystem_climate.seasons` key present in all 39 records, type `string`, non-empty
- [ ] `ecosystem_climate.seasons` value ends with a `[TierN|...]` tag, all 39

## Pass 6 — independent research batch (law-researcher/other_notable, financial-researcher, geography-researcher)
- [ ] `law.other_notable`: key present in all 39 records; value is `null` OR a non-empty `string` — never absent
- [ ] `financial.local_hidden_fees`: present, non-empty `string`, all 39, tagged
- [ ] `financial.utilities_note`: present, non-empty `string`, all 39, tagged
- [ ] `financial.insurance_exposure`: present, non-empty `string`, all 39, tagged
- [ ] `financial.cost_of_living_index` (task 6a): non-null in all 39 records (was null in exactly 10 going in — Columbia, Farmington, Folsom, Germantown, Greenville, Palo Alto, Salisbury, Scottsdale, Tacoma, Woodbridge; confirm those 10 specifically got filled, and confirm the other 29 are byte-for-byte unchanged from their Pass-0 baseline value)
- [ ] `geography.unique_facts`: present, non-empty `string`, all 39, tagged `[Tier3|...]`

---

## Not covered by contract-checker (belongs to category-auditor or schema-validator instead)
- Whether a cited source is actually the correct tier by name (a lookup `category-auditor` does against `source_tier_list.md` — not scriptable here since it needs to read the citation's prose, not just check a tag format)
- Whether a fact is plausible against model knowledge (requires judgment, not scriptable — `category-auditor`'s job, done without live search; see its file for the clean/flagged/unverifiable framework)
- Cross-field factual consistency, e.g. does `insurance_exposure` actually agree with `disaster_risk` in substance (requires reading comprehension, not scriptable)
- Whether the dataset as a whole is ready to finalize (that's the Pass 9 rollup — see `schema-validator.md`)
