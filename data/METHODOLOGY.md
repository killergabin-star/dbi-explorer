# METHODOLOGY — Demographic Blindness Index (DBI)

Version 1.0.0 · 2026-05-29 · Eric Gabin Kilama

This document specifies how the DBI is constructed, weighted, and bounded with
uncertainty; why one candidate component was demoted; and what the measure can and
cannot support. All numerical values trace to `VERITE_RESULTATS.md` (single source of
truth) and the build outputs cited inline.

---

## 1. Construction: components and formula

The DBI is a two-component composite of institutionally **non-circular** signals of a
state's capacity to see its population:

```
DBI = 0.60 * D2  +  0.40 * D4
```

with both components on [0, 1], so DBI ∈ [0, 1].

### D2 — census staleness (UNSD)

```
D2_t = (year − last_census_at_t) / max_gap ,   capped to [0, 1]
max_gap = 40
```

`D2` measures how long it has been since the state last conducted a full enumeration,
normalized by a **40-year absolute anchor** (`max_gap = 40`). A country censused this
year scores D2 ≈ 0; a country 40+ years stale scores D2 = 1. The anchor is **absolute**
(not data-relative), which is what makes the annual panel **continuous** with the
published 2024 cross-section (`max|diff| = 0`; see §6).

- **Source:** year of last census, United Nations Statistics Division (UNSD) census archive.

### D4 — birth-registration shortfall (UNICEF)

```
D4 = 1 − birth_registration_rate
```

`D4` measures the share of births the state fails to record — a flow measure of
demographic blindness complementary to the stock measure D2.

- **Source:** UNICEF birth-registration completeness, most recent household-survey round.

---

## 2. Weights: 60/40, and their robustness

The baseline weights are **60% D2 / 40% D4** (a design choice giving slightly more weight
to the stock of staleness than the registration flow). The country **ranking** is highly
robust to this choice:

| Aggregation alternative | Spearman ρ vs linear 60/40 | Verdict |
|-------------------------|----------------------------|---------|
| PCA (first principal component) | **0.999** | Linear ≈ PCA-optimal |
| max (worst-dimension, non-compensatory) | 0.953 | Ranking preserved |
| Weight sensitivity (full sweep) | **ρ ≥ 0.88** | Robust |
| Weight sweep, central 30/70–80/20 | ρ ≥ 0.95 | Very robust |

PCA-optimal weights are ≈ 50/50 and outcome-optimized weights ≈ 53/47, both close to the
60/40 baseline; the linear index is essentially weight-invariant in its ordering. The
**linear 60/40 index is retained** as the headline specification.
(Source: `output/noncompensatory_robustness_results.txt`,
`output/governance_weight_robustness_results.txt`.)

---

## 3. Monte-Carlo uncertainty bands

Each country-year DBI is reported with a **90% uncertainty band** (`dbi_lo`, `dbi_hi`,
`band_width`) obtained by Monte-Carlo simulation that propagates measurement noise in the
component inputs (census-year and birth-registration uncertainty) through the index.

- `dbi_median` — median of the simulated draws (central estimate).
- `dbi_lo` / `dbi_hi` — 5th / 95th percentiles (the 90% band).
- `band_width = dbi_hi − dbi_lo`.

**Median band width across the panel ≈ 0.040.** Rank stability under the simulation is
high: the **median standard deviation of a country's 2024 rank** across Monte-Carlo draws
is **2.09 positions out of 100**. (Source: `output/dbi_annual_panel_results.txt`.)

---

## 4. D1 demotion rationale (WorldPop circularity)

A third candidate component, **D1**, was derived from **WorldPop** gridded-population
error. It was **demoted** (excluded from the index) because gridded-population products
**ingest census counts as a primary input**: D1 would have been mechanically circular
with D2 and with the census foundation of the index. Including D1 would have measured the
inputs to the population model, not an independent dimension of blindness. The DBI is
therefore built from **D2 and D4 only**, both of which are sourced from registries that
are institutionally distinct from one another (UNSD census archive; UNICEF survey-based
registration) and from the downstream outcomes used in validation.
(Source: `scripts/92_dbi_annual_panel.R`; `VERITE_RESULTATS.md` line 33.)

---

## 5. Validation summary (why the DBI behaves like a capacity measure)

The DBI is validated against independent footprint sensors and downstream outcomes
(all SSA cross-section unless noted; full statistics in `VERITE_RESULTATS.md`):

- **Independent footprint sensors** (the state should be *more* blind where independent
  population signals are weak):
  - Mobile subscriptions / DBI: **−64.3** (p = 0.015, N = 46).
  - Google building-footprint density / DBI: **+2.79** (p = 0.018, N = 29).
  - Electoral-roll deviation / DBI: **−0.162** (p = 0.063, N = 44, suggestive).
- **Downstream coverage gap:** DBI → health-coverage gap **+16.22*** (baseline, N = 46);
  remains large with a statistical-capacity (**SPI**) control (**+18.4***), i.e. the DBI
  is a **complement to SPI, not redundant** with it. (The Wantchekon orthogonality check
  likewise shows the DBI coefficient is stable to adding SPI: 22.11 → 25.05.)
  *Note: the specific N=46 "+SPI = 18.4" cell is flagged for re-derivation in
  `VERITE_RESULTATS.md` line 306; the complementarity finding itself is corroborated by
  the Wantchekon FE result (3.098 → 3.101, unchanged with SPI).*
- **Statistical capacity:** DBI → SPI (SSA) **−20.8** (p < 0.01, N = 46).

### Discriminant validity

The DBI **concentrates on the data / public-financial-management (PFM) pillar** of
governance rather than reflecting general state capacity:

- DBI → CPIA cluster D (data/PFM): **−1.159*** (p = 0.001).
- DBI → CPIA clusters A/B/C (general capacity): **−0.816*** (p = 0.024) — **weaker, but
  still present**.
- The data/PFM pillar accounts for ≈ **77%** of the governance channel vs ≈ **22%** for
  general capacity.
- CPIA_D is scored from **budget/PFM documents**, *not* from DHS/WHO outcome data, so this
  is a **non-circular** association.

> **Wording discipline:** state that the DBI **"concentrates on"** the data/PFM pillar.
> Do **NOT** claim it is "absent from general-capacity measures" — that is an overclaim
> (the CPIA_ABC effect is significant at p = 0.024). See `VERITE_RESULTATS.md` §CPIA flag.

The DBI is **stable to legal origin**: adding legal-origin fixed effects changes the main
coefficient only 1.1% (**24.39*** → 24.13***), i.e. coefficient stability of a
significant effect (not an underpowered null).

---

## 6. Annual-panel continuity and census drops

- **Continuity:** the 2024 slice of the annual panel reproduces the published 2024
  cross-section exactly — `max|diff| = 0`, `mean|diff| = 0` (N = 100).
- **Census responsiveness:** **77 of 100** countries show a DBI **drop** at a census event
  (D2 resets toward 0 when a new census lands), confirming the index moves the right way.
- Mean within-country DBI span 2000–2024 ≈ 0.153.

---

## 7. Companion structural model (allocative welfare)

The data product is accompanied by a small structural model
(`manuscript/model_demographic_blindness.tex`): fixed-budget per-capita allocation under
blindness `ε_j`, with welfare loss per unit of budget:

```
welfare_loss / B = (1/2) * γ * Var_s(ε)
```

**Key result:** welfare loss is **first-order in the *dispersion* of blindness and ZERO in
its mean** — a uniform mis-count cancels in a fixed per-capita rule; only *unequal*
blindness is costly.

Calibration on Nigeria (742 LGAs):
- Population-weighted **Var_s(ε) = 0.150** (SD 0.388).
- **Within-state share of dispersion = 93.9%** — confirms Corollary 1 and matches the
  independent 91% finding.
- Welfare loss = **7.5% / 15.0% / 22.5%** of the pool for γ = 1 / 2 / 3, i.e.
  **$163M / $326M / $489M** on an illustrative pool B = $2.17bn.
- Counterfactual: halving dispersion (γ = 2) yields a welfare gain of **≈ $163M/yr**.

These figures are **illustrative order-of-magnitude** calibrations, not estimated
treatment effects.

---

## 8. Mechanism evidence (suggestive, within-design)

- **Brazil (recording channel):** municipalities **bunch just above** fiscal-transfer
  population thresholds and register **more births** with **no fertility response** — a
  *recording* response, not a real demographic one. **Proof-of-mechanism (suggestive),
  not "demonstrated."**
- **Nigeria fiscal (illustrative, ranged):** mean national ghost ≈ **7.3%**;
  ≈ **$105M/yr** transfer misdirection in the 774-LGA FAAC system; an IDA-reallocation
  simulation spans **$0.7–2.9bn/yr** depending on the assumed causal share. **All fiscal
  numbers are ranged and illustrative order-of-magnitude.**

---

## 9. What the DBI is NOT — chosen-blindness and the event study

- **Blindness is structural, not chosen.** The "chosen-blindness" thesis is **not
  supported at testable levels**: national census timing is approximately *scheduled*, and
  Nigerian over-count tracks **administrative-unit size, not North–South politics**
  (political-geography R² ≈ 0.003). Political economy enters on the **consequence** side
  (mis-allocation), not the cause side.
- **The census event study is a DEMOTED robustness item, not a causal pillar.**
  TWFE shows admin coverage rising **+3.51*** post-census, but the
  heterogeneity-robust estimators are **null**: stacked DiD on the recording gap **−0.04**
  (p = 0.97), Callaway–Sant'Anna short-run **+0.57** (n.s.). The clean design is
  **underpowered** (MDE 3.23pp > the 2.43pp implied divergence), and WUENIC is a WHO model
  that ingests admin reports, so it is **not an independent placebo**.
  **"We do not rest a causal claim on it."**

---

## 10. Known limits

1. **D4 is single-round per country** (`single_round_D4 = TRUE`). Within-country *time*
   variation in the DBI is therefore **D2-driven**; D4 is held fixed at its most recent
   survey value. Acquiring multi-round birth-registration (DHS/MICS panel) is the next
   step to give D4 genuine time variation.
2. **Coverage = 100 countries** in the annual panel (46 validated SSA + the global
   extension). Not yet global-universal.
3. **D1 (WorldPop) is excluded** by design (circularity, §4) — the index deliberately
   omits a gridded-population dimension.
4. **Uncertainty bands** reflect input-measurement noise propagated by Monte-Carlo; they
   do **not** capture model/specification uncertainty in the weights (addressed separately
   by the robustness sweep in §2).
5. **Welfare and fiscal figures are illustrative** order-of-magnitude calibrations
   (§§7–8), not estimated causal effects.

---

## 11. Contribution type

This is a **measurement contribution (type b)** — a new index — augmented by a structural
model (§7) and one within-design mechanism test (§8). Its honest publication ceiling is
**top-field-very-high**; a top-5 placement is the target **conditional** on the structural
model and identification convincing referees.
