# Golden Record — Atlanta, structural reference

**What this is:** a worked example of what Atlanta's record should look like after every pass through Pass 5 (the mechanical/reshaping/reconciliation work), built from the actual Atlanta content already provided earlier in this project — not invented. **What this is not:** a claim about what the still-missing research fields (Pass 6) actually contain. Those are marked `<PENDING RESEARCH — see [agent]>` rather than filled with a plausible-sounding guess, because a fabricated placeholder that looks real is worse than an honest gap — it's exactly the kind of thing that could get mistaken for verified content later.

Use this for **structural** cross-checking (does a record have the right shape) — `contract-checker` and `schema-validator` can diff a real record's structure against this one's. Don't use it as a content-accuracy reference; that's not what it's for.

```json
{
  "city": "Atlanta",
  "state_ref": "georgia",

  "law": {
    "alcohol_sales": "Local-option under GA Code 3-3-7; where authorized, standard Sunday window ~11 AM-12 AM, varies by county/city referendum; weekday sales effectively unrestricted statewide [Tier1|cross_verified:true|verified|2026-09-13]. <PENDING RESEARCH — see law-researcher, Pass 2: on-premise/bar hours, dry-county status for Fulton/DeKalb County>",
    "other_notable": "<PENDING RESEARCH — see law-researcher, Pass 6 — default null if nothing unusual surfaces>"
  },

  "financial": {
    "cost_of_living_index": "C2ER-sourced composite 95.7 (Atlanta metro, ARC Regional Snapshot Oct 2024 update, Q2 2024 C2ER data) — VERIFY: metro-level not city-specific [Tier2|cross_verified:true|single-source|2026-09-13]",
    "median_rent_1br": "$1,610 (RentCafe, Aug 2026); cross-verified against HUD FY2026 FMR ($1,660) [Tier2|cross_verified:true|verified|2026-09-13]",
    "median_rent_2br": "$1,885 (RentCafe, Aug 2026); cross-verified against HUD FY2026 FMR ($1,820) [Tier2|cross_verified:true|verified|2026-09-13]",
    "property_tax_rate": "Effective ~1.6% of fair market value for City of Atlanta/Fulton County residents (2025 combined millage 40.909 mills), computed from Fulton County Tax Commissioner's published millage table [Tier1|cross_verified:true|verified|2026-09-13]",
    "local_hidden_fees": "<PENDING RESEARCH — see financial-researcher, Pass 6>",
    "utilities_note": "<PENDING RESEARCH — see financial-researcher, Pass 6>",
    "insurance_exposure": "<PENDING RESEARCH — see financial-researcher, Pass 6>"
  },

  "geography": {
    "terrain_summary": "Piedmont plateau, rolling hills, ~320 m (1,050 ft) avg elevation — highest major city in the SE US east of the Rockies [Tier2|cross_verified:true|verified|2026-09-13]",
    "disaster_risk": "FEMA NRI v1.20 composite 95.8, county rating 'Relatively High' (Fulton County; #2 of 159 GA counties) — top hazards flood 97.7, tornado 97.0, earthquake 96.0 [Tier2|cross_verified:true|verified|2026-09-13]",
    "unique_facts": "<PENDING RESEARCH — see geography-researcher, Pass 6 — Tier3, flagged lower-confidence by design>"
  },

  "household_lifestyle": {
    "housing_snapshot": "Metro ZHVI (Atlanta-Sandy Springs-Roswell) $377,813, down ~1.5% YoY (Zillow Research, Aug 2026) [Tier2|cross_verified:true|verified|2026-09-13]",
    "partner_job_market": "Atlanta-Sandy Springs-Roswell MSA unemployment 2.8-3.2% (BLS, 2026) [Tier1|cross_verified:true|verified|2026-09-13]",
    "pet_logistics": "No pit-bull/breed ban identified for this specific city this pass [Tier2|cross_verified:true|single-source|2026-09-13]"
  },

  "amenities": {
    "in_n_out_present": {
      "present": false,
      "detail": "Georgia has zero In-N-Out locations statewide, cross-verified across 5 independent sources [Tier2|cross_verified:true|verified|2026-09-13]"
    },
    "costco_present": {
      "present": true,
      "detail": "Multiple official warehouses within city limits, e.g. Perimeter (6350 Peachtree Dunwoody Rd.) and Brookhaven (500 Brookhaven Ave. NE) [Tier1|cross_verified:true|verified|2026-09-13]"
    },
    "dutch_bros_present": {
      "present": false,
      "detail": "GA's Dutch Bros locations are all metro-Atlanta suburbs outside the city itself: Johns Creek, Tucker, Lawrenceville, Roswell — none inside Atlanta's own boundary [Tier2|cross_verified:true|verified|2026-09-13]"
    },
    "_meta": "amenities complete: in_n_out_present, dutch_bros_present, costco_present all resolved 39/39 cities [batch:2026-09-13]"
  },

  "visitability": {
    "nearest_airport": {
      "airport_code": "ATL",
      "airport_name": "Hartsfield-Jackson Atlanta International Airport",
      "distance_mi": 10,
      "drive_time_min": 15
    },
    "distance_from_anchors": [
      {
        "anchor_label": "SF",
        "anchor_airport_code": "SFO",
        "distance_mi": 2134,
        "drive_time_hr": null,
        "flight_time_hr": "<PENDING RESEARCH — see distance-calculator, Pass 4>"
      },
      {
        "anchor_label": "VA",
        "anchor_airport_code": "IAD",
        "distance_mi": 534,
        "drive_time_hr": "<PENDING RESEARCH — see distance-calculator, Pass 4 — a drive is plausible at this distance, don't default to null>",
        "flight_time_hr": "<PENDING RESEARCH — see distance-calculator, Pass 4>"
      },
      {
        "anchor_label": "AZ",
        "anchor_airport_code": "PHX",
        "distance_mi": 1584,
        "drive_time_hr": null,
        "flight_time_hr": "<PENDING RESEARCH — see distance-calculator, Pass 4>"
      }
    ],
    "nearby_pois": [
      "World of Coca-Cola",
      "Georgia Aquarium",
      "CNN Center",
      "Martin Luther King Jr. National Historical Park",
      "Atlanta BeltLine"
    ]
  },

  "ecosystem_climate": {
    "pests": "Fire ants (statewide), lone star tick & American dog tick, mosquitoes (Asian tiger), termites [Tier2|cross_verified:true|verified|2026-09-14]",
    "skin_irritants": "Poison ivy common statewide incl. metro Atlanta; poison oak also present [Tier2|cross_verified:true|verified|2026-09-14]",
    "air_quality": "Grade B+ / median AQI 49 (2025); ALA gives Fulton Co. an F for ozone [Tier2|cross_verified:true|verified|2026-09-14]",
    "daylight": "Computed (lat 33.75N): ~14.4h summer solstice, ~9.9h winter solstice [Tier3|cross_verified:false|estimated|2026-09-14]",
    "seasons": "Humid subtropical (Köppen Cfa) confirmed via NWS Peachtree City official climate-normal station (1991-2020 normal period); hot humid summers, mild winters with occasional ice [Tier1|cross_verified:true|verified|2026-09-14] Source: forecast.weather.gov (NWS Atlanta/Peachtree City CLI product). [Reconciled by seasons-reconciler, Pass 5 — this version won over the geography.seasons duplicate for citing a specific NWS station directly, per the reconciler's Tier-1-wins rule]"
  },

  "geography_context": {
    "proximity_highlights": [
      {"place": "NC/SC Blue Ridge foothills", "distance_mi": null, "note": "~2.5-3h drive"},
      {"place": "Gulf/Atlantic coast", "distance_mi": null, "note": "~4.5h drive"}
    ],
    "regional_centrality": "Primary economic, cultural, and transportation hub of the Southeast US ('capital of the New South'); major corporate HQ base (Delta, Coca-Cola, Home Depot, UPS) [Tier3|cross_verified:false|estimated|2026-09-14]"
  }
}
```

## Annotations — judgment calls made in producing this example

1. **`proximity_highlights` drops the "world's busiest airport" framing sentence.** The source prose opened with general airport-connectivity context that isn't a place-with-a-distance. Since `visitability.nearest_airport` already covers airport identity, this was dropped rather than force-fit into a place entry. This is now written into `field-reshaper`'s instructions as general guidance, not just an Atlanta-specific call.

2. **`distance_mi: null` appears twice in `proximity_highlights`.** The source prose gave drive times, not mileage, for both entries. This is the concrete case behind the Pass 8 schema-gap discussion — a real instance, not a hypothetical.

3. **`drive_time_hr` for VA is left pending rather than defaulted to `null`.** 534 miles is a plausible one-long-day or two-day drive, unlike the SF/Phoenix distances. `distance-calculator`'s instructions already say not to default to `null` when a drive is genuinely practical — this record is the worked example of that rule.

4. **Nothing in this file should be read as an assertion about Atlanta's actual tax rates, insurance market, or other pending-research facts.** Every `<PENDING RESEARCH>` marker is intentional and should stay that way until the real pass runs.
