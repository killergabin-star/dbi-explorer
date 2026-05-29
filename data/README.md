# Demographic Blindness Index (DBI) — Data Package

**Version:** 1.0.0
**Release date:** 2026-05-29
**Author:** Eric Gabin Kilama
**License:** Creative Commons Attribution 4.0 International (CC-BY-4.0)

---

## 1. What the DBI is

The **Demographic Blindness Index (DBI)** is a country-level measure of how poorly a
state *sees* its own population for the purposes of public administration. It captures
the gap between the population a government can locate and count, and the population that
actually exists — the operational blind spot that distorts per-capita resource
allocation, service planning, and statistical capacity.

The DBI is the composite of two observable, institutionally non-circular components:

```
DBI = 0.60 * D2  +  0.40 * D4
```

| Component | Concept | Definition | Range |
|-----------|---------|------------|-------|
| **D2** | Census staleness | `(year − year of last census) / 40`, capped at [0, 1] | [0, 1] |
| **D4** | Birth-registration shortfall | `1 − birth_registration_rate` | [0, 1] |

Both components live on [0, 1], so the DBI itself is bounded in [0, 1], where **0 = a
state that sees its population well** and **1 = near-total demographic blindness**.

A third candidate component (**D1**, derived from WorldPop gridded-population error) was
**demoted** during construction because it is mechanically circular with the census
inputs used by gridded-population models — including it would have measured the inputs,
not an independent signal. See `METHODOLOGY.md` §4.

### Coverage

- **46 Sub-Saharan African countries** — the validated cross-section (published headline figures).
- **100-country annual panel, 2000–2024** — **1,227 country-year observations**
  (`dbi_annual_panel_2000_2024.csv`, the file in this package).

### Headline distribution (100-country panel)

| Statistic | Value |
|-----------|-------|
| DBI range | [0.000, 0.931] |
| Most blind | Somalia (0.931, 2024) |
| | DR Congo (0.840), Ethiopia (0.644), Nigeria (0.500) |
| Least blind (example) | South Africa (0.076, 2024) |

(The published **46-country cross-section** reports a median DBI of **0.222** and IQR
[0.101, 0.338]; the 100-country annual panel has a lower pooled median because it
includes country-years immediately after a census, when D2 resets toward 0.)

### What the DBI is *for* — and what it is not

The DBI is a **measurement instrument** (a measurement-type contribution), accompanied by
a small structural model of allocative welfare loss and one within-country mechanism test.
It is **descriptive of state capacity to see**, not itself a causal estimate. Validation
evidence (see `METHODOLOGY.md` §5) shows the DBI behaves as a capacity measure should: it
correlates with independent footprint signals (mobile penetration, building footprints,
electoral-roll deviations) and predicts downstream coverage gaps and statistical-capacity
(SPI) shortfalls. Blindness, where tested, appears **structural** (a capacity constraint),
**not chosen** (no evidence of strategic manipulation at testable levels).

---

## 2. Files in this package

| File | Description |
|------|-------------|
| `dbi_annual_panel_2000_2024.csv` | The DBI annual panel — 100 countries × 2000–2024, 1,227 obs. Primary data product. |
| `README.md` | This file. |
| `CODEBOOK.md` | Column-by-column documentation of the CSV (definitions, units, sources). |
| `METHODOLOGY.md` | Construction formula, weighting, uncertainty, demotion rationale, known limits. |
| `CITATION.cff` | Machine-readable citation metadata. |
| `LICENSE.txt` | Full CC-BY-4.0 license text. |

The CSV is UTF-8 encoded, comma-separated, one header row, 1,227 data rows. Note that
several `country` fields contain commas inside double quotes (e.g.
`"Congo, Dem. Rep."`) — parse with a standards-compliant CSV reader, not naive
splitting on commas.

---

## 3. How to cite

> Kilama, Eric Gabin (2026). *Demographic Blindness Index (DBI): Annual Country Panel,
> 2000–2024* (Version 1.0.0) [Data set]. Zenodo. https://doi.org/10.5281/zenodo.XXXXXXX

A machine-readable version is provided in `CITATION.cff`. Replace `zenodo.XXXXXXX`
with the DOI minted on deposit. **[TODO: insert Zenodo DOI on deposit]**

---

## 4. Versioning

This package follows **semantic versioning** (`MAJOR.MINOR.PATCH`):

- **MAJOR** — a change to the DBI formula, component definitions, or weights.
- **MINOR** — added country-years, added countries, or a new component column.
- **PATCH** — corrections to existing values, source metadata, or documentation.

| Version | Date | Notes |
|---------|------|-------|
| 1.0.0 | 2026-05-29 | First public release. 100-country annual panel (2000–2024); continuity check against the published 2024 cross-section: `max|diff| = 0`. |

Each Zenodo release receives a version-specific DOI; the concept DOI always resolves to
the latest version.

---

## 5. License

This dataset is released under the **Creative Commons Attribution 4.0 International
(CC-BY-4.0)** license. You are free to share and adapt the material for any purpose,
including commercially, provided you give appropriate credit (see §3). Full text in
`LICENSE.txt`.

---

## 6. Provenance and sources

- **D2** (census staleness): year of last census from the **United Nations Statistics
  Division (UNSD)** census archive.
- **D4** (birth-registration shortfall): birth-registration completeness from
  **UNICEF** (most recent available household-survey round per country).
- Construction script: `scripts/92_dbi_annual_panel.R` (build) and
  `scripts/42_dbi_expanded_48countries.R` (46-country cross-section).
- All numerical values in this documentation trace to
  `VERITE_RESULTATS.md` (single source of truth) and `output/dbi_annual_panel_results.txt`.

---

## 7. Contact

Eric Gabin Kilama — author and maintainer.
For corrections or questions about a specific country-year, reference the `iso3` and
`year` of the observation.
