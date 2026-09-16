# Source Tier List — full reference

Each research agent in `.claude/agents/` has its own curated excerpt of this list baked into its system prompt. This file is the full source — keep it in `data/` for reference and for updating an agent's curated excerpt if the underlying policy changes.

---

**TIER 0 — computed / API-derived (not a citation — exclude from source-tier labeling)**
- Mapping/routing outputs: drive times, distances, nearby POIs (Google Maps, HERE, etc.)
- Applies to: `residency_context.suburb_for[].distance_mi` / `drive_time_*`, `visitability.distance_from_anchors`, `visitability.nearby_pois`

**TIER 1 — primary (always prefer if it exists)**
- Government: .gov domains, city/county/state agency sites, official statute text
- Regulatory/licensing boards (state bar, state medical board, DMV)
- Primary data: Census.gov/ACS, BLS.gov, IRS.gov, state revenue departments
- Climate/geo: NOAA/NWS (climate normals + sunrise/sunset), USGS (terrain), FEMA National Risk Index
- FAA / airport authority data
- Official company store locators, for point-presence queries only (In-N-Out, Costco, Dutch Bros)
- City/county fee schedules, utility provider sites (local fees, utilities)

**TIER 2 — established secondary (use when Tier 1 doesn't cover the claim)**
- Recognized data aggregators: C2ER/MERIC (COL), Zillow Research / Redfin Data Center (raw housing data — not their blog content), RentCafe/Zumper/Apartments.com (rent), Niche/AreaVibes
- EPA AirNow (air quality), FCC/Ookla (broadband/cell)
- State agricultural extension / health department sites (pests, skin irritants)
- Major news orgs with a correction/editorial policy (not press-release mirrors)
- University or professional-association publications (.edu, bar journals)

**TIER 3 — general web (only if Tier 1/2 are silent, and always flagged)**
- Everything else — must be labeled lower-confidence in the output
- Default bucket for: `unique_facts`, `proximity_highlights`, `regional_centrality`

**EXCLUDE outright, regardless of tier availability:**
- Listicles / "best places to live" content with obvious commercial intent (realtor blogs, moving-company blogs, SEO content farms)
- Forums, Reddit, Quora as a factual source (fine as a lead to verify elsewhere, never as the citation)
- Any site where the byline/date is missing or the content reads AI-generated without sourcing of its own

**UNRESOLVED — do not tier yet:**
- `commute.ez_time_diff`: field definition ambiguous, confirm meaning before sourcing — n/a for this pipeline, `commute` is out of scope
- `household_lifestyle.pet_logistics`: no authoritative source category exists; treat as manual/optional, out of strict-source scope — n/a for this pipeline, no issues flagged for this field

## Known gap (flagged during agent design, not yet resolved)
No explicit Tier 1/2 category exists for state insurance regulators or industry bodies (e.g. a state Department of Insurance, NAIC, III.org) for `financial.insurance_exposure`. `financial-researcher`'s curated excerpt treats a state Dept. of Insurance site as Tier 1 by analogy to other regulatory boards, but this should be formally added here once confirmed.
