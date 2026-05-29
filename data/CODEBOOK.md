# CODEBOOK — `dbi_annual_panel_2000_2024.csv`

**Dataset:** Demographic Blindness Index (DBI), annual country panel, 2000–2024.
**Rows:** 1,227 country-year observations (header row excluded).
**Units of observation:** one row per country × year.
**Countries:** 100. **Years:** 2000–2024.
**Encoding:** UTF-8, comma-separated, one header row.
**Parsing note:** some `country` values contain a comma inside double quotes
(e.g. `"Congo, Dem. Rep."`). Use a standards-compliant CSV parser.

The file has **15 columns**, in the order below. (Note: the canonical analytic columns
requested for the data product — `iso3`, `country`, `year`, `D2_t`, `D4`, `DBI_t`,
`dbi_lo`, `dbi_hi`, `band_width`, `ssa` — are all present; the file additionally ships
the provenance/uncertainty columns `birth_reg_pct`, `survey_year`, `last_census_at_t`,
`single_round_D4`, and `dbi_median` so the package is self-documenting.)

---

## Column dictionary

### 1. `iso3`
- **Type:** string (3-letter code).
- **Definition:** ISO 3166-1 alpha-3 country code (e.g. `NGA`, `SOM`, `COD`, `ZAF`).
- **Units:** categorical identifier.
- **Source:** ISO 3166-1.

### 2. `year`
- **Type:** integer.
- **Definition:** Calendar year of the observation.
- **Range:** 2000–2024.
- **Units:** year.

### 3. `country`
- **Type:** string (may contain commas; quoted in CSV).
- **Definition:** Country name in World Bank short form (e.g. `Nigeria`,
  `"Congo, Dem. Rep."`, `"Somalia, Fed. Rep."`).
- **Units:** categorical label.
- **Source:** World Bank country naming convention.

### 4. `D4`
- **Type:** float, [0, 1].
- **Definition:** **Birth-registration shortfall** = `1 − birth_registration_rate`.
  The share of births *not* registered. Higher = worse (more blindness).
- **Units:** proportion (fraction of births unregistered).
- **Source:** UNICEF birth-registration completeness (most recent survey round).
- **Note:** Single-round per country in this release (see `single_round_D4`).

### 5. `birth_reg_pct`
- **Type:** float, [0, 100].
- **Definition:** Birth-registration **completeness rate** in percent. Provenance value
  underlying `D4`: `D4 = 1 − birth_reg_pct/100`.
- **Units:** percent of births registered.
- **Source:** UNICEF.

### 6. `survey_year`
- **Type:** integer (year).
- **Definition:** Year of the household survey from which `birth_reg_pct` / `D4` is
  taken. Documents which round D4 is anchored to.
- **Units:** year.
- **Source:** UNICEF survey metadata (DHS/MICS round year).

### 7. `ssa`
- **Type:** integer (0/1) — binary indicator.
- **Definition:** `1` if the country is in **Sub-Saharan Africa**, `0` otherwise. Flags
  membership in the 46-country validated SSA cross-section.
- **Units:** indicator.
- **Source:** World Bank regional classification.

### 8. `last_census_at_t`
- **Type:** integer (year).
- **Definition:** Year of the **most recent census on or before** the observation `year`.
  Drives D2; updates (steps) when a new census occurs.
- **Units:** year.
- **Source:** UNSD census archive.

### 9. `D2_t`
- **Type:** float, [0, 1].
- **Definition:** **Census staleness** at year `t` = `(year − last_census_at_t) / 40`,
  capped at [0, 1]. Years elapsed since the last census, normalized by a 40-year anchor.
  Higher = more stale = more blindness.
- **Units:** proportion (fraction of the 40-year maximum gap).
- **Source:** derived from `year` and `last_census_at_t` (UNSD).

### 10. `single_round_D4`
- **Type:** boolean (`TRUE` in all rows of this release).
- **Definition:** Flag indicating `D4` is built from a **single** birth-registration
  survey round per country (no within-country time variation in D4). Documents the known
  limit that, in v1.0.0, within-country DBI movement is **D2-driven**.
- **Units:** indicator.
- **Source:** construction flag (`scripts/92_dbi_annual_panel.R`).

### 11. `DBI_t`
- **Type:** float, [0, 1].
- **Definition:** **The Demographic Blindness Index** at year `t` =
  `0.60 * D2_t + 0.40 * D4`. Primary outcome variable.
- **Units:** index, [0, 1] (0 = sees population well; 1 = near-total blindness).
- **Range in file:** [0.000, 0.931].
- **Reference values (2024):** Somalia 0.931, DR Congo 0.840, Ethiopia 0.644,
  Nigeria 0.500, South Africa 0.076.
- **Source:** derived (`scripts/92_dbi_annual_panel.R`).

### 12. `dbi_median`
- **Type:** float, [0, 1].
- **Definition:** **Median** of the DBI across the Monte-Carlo uncertainty draws for that
  country-year (the central tendency of the simulated distribution). Numerically
  ≈ `DBI_t`.
- **Units:** index.
- **Source:** Monte-Carlo simulation (see `METHODOLOGY.md` §3).

### 13. `dbi_lo`
- **Type:** float, [0, 1].
- **Definition:** **Lower bound** of the Monte-Carlo uncertainty band — the 5th
  percentile of the simulated DBI distribution (lower edge of the 90% band).
- **Units:** index.
- **Source:** Monte-Carlo simulation.

### 14. `dbi_hi`
- **Type:** float, [0, 1].
- **Definition:** **Upper bound** of the Monte-Carlo uncertainty band — the 95th
  percentile of the simulated DBI distribution (upper edge of the 90% band).
- **Units:** index.
- **Source:** Monte-Carlo simulation.

### 15. `band_width`
- **Type:** float, ≥ 0.
- **Definition:** Width of the 90% Monte-Carlo uncertainty band = `dbi_hi − dbi_lo`.
  Smaller = more precise DBI estimate for that country-year.
- **Units:** index points.
- **Median across panel:** ≈ 0.040.
- **Source:** derived (`dbi_hi − dbi_lo`).

---

## Quick reference — column → concept

| Column | Concept | Range/Type | Source |
|--------|---------|------------|--------|
| `iso3` | Country code | str | ISO 3166-1 |
| `year` | Year | 2000–2024 | — |
| `country` | Country name | str (quoted) | World Bank |
| `D4` | Birth-reg. shortfall = 1−rate | [0,1] | UNICEF |
| `birth_reg_pct` | Birth-reg. completeness | [0,100] % | UNICEF |
| `survey_year` | D4 survey round year | year | UNICEF |
| `ssa` | Sub-Saharan Africa flag | 0/1 | World Bank |
| `last_census_at_t` | Most recent census ≤ year | year | UNSD |
| `D2_t` | Census staleness = (t−census)/40 | [0,1] | UNSD (derived) |
| `single_round_D4` | D4 single-round flag | bool (TRUE) | construction |
| `DBI_t` | **DBI = 0.60·D2 + 0.40·D4** | [0,1] | derived |
| `dbi_median` | MC median DBI | [0,1] | Monte-Carlo |
| `dbi_lo` | MC 90% lower bound | [0,1] | Monte-Carlo |
| `dbi_hi` | MC 90% upper bound | [0,1] | Monte-Carlo |
| `band_width` | dbi_hi − dbi_lo | ≥0 | derived |
