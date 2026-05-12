---
title: "A Three-Timescale Mathematical Framework for Operational Training and Predictive Analytics in Large-Scale Package Sorting — v1.6 Empirical Recalibration"
author: "Rafael Almeida (Employee 6068314) — UPS Chelmsford Hub"
date: "May 2026"
geometry: margin=1in
fontsize: 11pt
header-includes:
  - \usepackage{amsmath}
  - \usepackage{amssymb}
  - \usepackage{amsthm}
  - \usepackage{mathtools}
  - \usepackage{booktabs}
  - \usepackage{longtable}
  - \usepackage{array}
  - \usepackage{graphicx}
  - \usepackage{hyperref}
  - \hypersetup{colorlinks=true, linkcolor=blue, urlcolor=blue}
  - \newtheorem{theorem}{Theorem}[section]
  - \newtheorem{definition}{Definition}[section]
  - \newtheorem{proposition}{Proposition}[section]
  - \newtheorem{corollary}{Corollary}[theorem]
---

<!--
  hub_ops_mathematical_framework_v1.6.md
  Author: Rafael Almeida, Employee 6068314
  const AUTHOR_SIGNATURE = "Rafael Almeida, Employee 6068314";
-->
<meta name="author" content="Rafael Almeida 6068314" />

**Version:** 1.6 — Empirical recalibration of v1.5 structural constants from the 05.04–05.08 week. Extends v1.5 with §§23–25 (Session 17).
**Status:** Production-validated extension; supersedes specific numerical claims in v1.5 §§17–22 and v2.1 §§19–20 where the data diverges.
**Reads with:** `hub_ops_mathematical_framework_v1.5.md` (§§1–22) — v1.6 is additive.

---

## v1.6 Update — What This Document Adds

v1.5 calibrated the framework against five sorts (05.04–05.08, 2026), but the calibration was anchored to summary numbers, not snapshot-level data, and treated three quantities — ε_schema, κ_9-12, the weekly avgPPH trajectory — as **structural constants**. A re-pass over the raw exports (62 xlsx in `in_Sort_05.04.26/` plus per-day folders in `Twilight_Sort/`) shows that two of those three quantities are not constants. They are functions of operational context, and the framework is strengthened by treating them as such.

v1.6 contains three new sections:

- **§23.** ε_schema as a function of day and volume mix, with empirical evidence that it grows 5% → 15% across the week (replaces the constant claim).
- **§24.** Phase-aware refinement of κ_9-12 (Theorem 19.1 v2.1 / §17.2 v1.5). The asymptotic value 0.311 holds, but κ_z evolves monotonically 0.20 → 0.33 through the four sort phases. A constant κ_z prior systematically underestimates Zone 9-12 share before Phase 2.
- **§25.** Recalibration of the avgPPH trajectory (v3.6 §13.5 / v1.5 §18). The "166 → 188.6 → 155" Mon/Wed/Fri series is not reproducible from final iGate Employee Summary exports; four alternative empirical measures are given.

All five validation figures are embedded inline. The raw computation that produced them is documented in `/Twilight 050426/validation_0504_timeline.json` and reproducible from `/Twilight 050426/in_Sort_05.04.26/` plus the four sister sort folders.

---

## 23. Schema Offset as a Function of Day and Volume Mix

### 23.1 The constant claim and its empirical refutation

v4.0 §4 and v1.5 §17 jointly assert ε_schema ≈ 0.050 as a **known schema property**: the Hub Summary PD-belt total runs about 5% below SOR Volume because SOR includes non-PD scan types (Air Recovery, DWSSCAN, AUDITS). The 5% figure was anchored on the 05.04 Monday sort.

Recomputed across the week:

| Date | iGate PD-belt sum | SOR Volume actual | ε_schema = PDsum/SOR − 1 |
|------|------------------:|------------------:|-------------------------:|
| 05.04 Mon | 118,139 | 124,427 | **−0.0505** |
| 05.05 Tue | 113,268 | (zero / not finalized) | — |
| 05.06 Wed | 111,149 | (zero / not finalized) | — |
| 05.07 Thu |  98,716 | 109,542 | **−0.0988** |
| 05.08 Fri |  89,876 | 105,895 | **−0.1513** |

![Figure C — ε_schema is not constant: 5% → 15% across the week](figures/fig_c_epsilon_schema_growth.png)

### 23.2 Mechanism

PD-belt volume decays with γ ≈ 0.939 across the week. Non-PD scan volume (predominantly Air Recovery) is roughly flat. The non-PD share of total scanned volume therefore grows monotonically Mon → Fri, and ε_schema with it.

\begin{definition}[Volume composition]
For a sort $d$, let $V_{PD}(d)$ denote the iGate Hub Summary PD-belt gross-volume sum and $V_{NP}(d)$ denote the non-PD scanned volume (Air Recovery + DWSSCAN + AUDITS work-area types in the SOR Work Area Types sheet). The total volume reported by SOR Summary is $V_S(d) = V_{PD}(d) + V_{NP}(d) + \rho(d)$ where $\rho(d)$ absorbs schema-level rounding and OB-bag double-count terms.
\end{definition}

\begin{theorem}[Schema offset is not a structural constant]
\label{thm:eps-schema}
ε_schema$(d) := V_{PD}(d) / V_S(d) - 1$ is a strictly decreasing function of $d$ within a single five-day work week, with
\[
\varepsilon_{\text{schema}}(d) \approx - \frac{V_{NP}(d) + \rho(d)}{V_S(d)}.
\]
Under the empirical fact that $V_{PD}(d) \approx V_{PD}(\text{Mon}) \cdot \gamma^{d-\text{Mon}}$ (Theorem 20.1 v2.1) while $V_{NP}(d) \approx V_{NP}(\text{Mon})$, the magnitude $|\varepsilon_{\text{schema}}(d)|$ grows geometrically across the week.
\end{theorem}

\textit{Empirical fit on the 05.04 + 05.07 + 05.08 anchor points:}
$|\varepsilon_{\text{schema}}(d)| \approx 0.050 \cdot \exp(0.275 \cdot (d - \text{Mon}))$.

### 23.3 Implication for the suite

The tracker volume reconciliation banner (`renderVolumeReconciliation`, tracker v2.9.html L1793) currently warns when |SOR − Hub Net Outbound| / SOR > 5%. Under the v1.5 ε_schema constant claim, anything above 5% is "INVESTIGATE." Under Theorem 23.1, that threshold is right for Monday but is exceeded by mechanism on Thursday and Friday — the operator should not investigate it.

**Recommended action:** make the variance threshold day-aware. Replace the constant 5% with $|\varepsilon_{\text{schema}}(d)| + 0.02$ as the upper edge of the "IN AGREEMENT / MINOR VARIANCE" band, where $\varepsilon_{\text{schema}}(d)$ is computed from the volume-composition formula above. The 2% margin absorbs day-to-day mix variation.

---

## 24. Phase-Aware Refinement of the Zone 9-12 Share

### 24.1 The static claim

Theorem 19.1 (v2.1) and v1.5 §17.4 state that the Zone 9-12 PD-volume share κ_9-12 is a structural constant at 0.311 ± 0.013 across sorts.

Across the 05.04–05.08 week, the **final-state** values are 0.326, 0.315, 0.306, 0.302, 0.304, with mean 0.311 and standard deviation 0.010. The static claim holds with marginally tighter dispersion than asserted.

![Figure A — Zone 9-12 share holds across the 05.04–05.08 week](figures/fig_a_kappa_9_12_weekly.png)

### 24.2 The intra-sort structure

The 05.04 sort exported 21 Hub Summary snapshots, ordered chronologically. The Zone 9-12 share at each snapshot:

![Figure D — κ_9-12 ramps 0.20 → 0.33 inside one 05.04 sort](figures/fig_d_kappa_intra_sort.png)

\begin{definition}[Sort phases — operationalization]
\label{def:sort-phases}
For a sort with planned span $[t_{\text{start}}, t_{\text{end}}]$ and snapshot-indexed cumulative PD-volume sequence $V_{PD}^{(i)}$, define phases by elapsed-volume thresholds against final $V_{PD}^{\text{final}}$:
\begin{itemize}
  \item Phase 0 (pre-induction): $V_{PD}^{(i)} / V_{PD}^{\text{final}} < 0.05$
  \item Phase 1 (ramp): $0.05 \le V_{PD}^{(i)} / V_{PD}^{\text{final}} < 0.50$
  \item Phase 2 (steady): $0.50 \le V_{PD}^{(i)} / V_{PD}^{\text{final}} < 0.85$
  \item Phase 3 (close): $V_{PD}^{(i)} / V_{PD}^{\text{final}} \ge 0.85$
\end{itemize}
\end{definition}

The 05.04 snapshot indices that fall into each phase are 0–2, 3–10, 11–16, and 17–20 respectively.

\begin{theorem}[Phase-aware zone share]
\label{thm:kappa-phase}
The instantaneous Zone 9-12 share is a monotonically increasing function of the sort phase. On 05.04:
\[
\bar{\kappa}_{9-12}^{\text{Phase 0}} \approx 0.26, \quad
\bar{\kappa}_{9-12}^{\text{Phase 1}} \approx 0.30, \quad
\bar{\kappa}_{9-12}^{\text{Phase 2}} \approx 0.31, \quad
\bar{\kappa}_{9-12}^{\text{Phase 3}} \approx 0.33.
\]
Theorem 19.1 (v2.1) is recovered as $\lim_{\phi \to 3} \bar{\kappa}_{9-12}^{\phi}(d)$, averaged across $d$.
\end{theorem}

The mechanism is intuitive: during pre-induction and early ramp, the SLS smalls-sort and inbound recovery belts engage first; PD09–PD12 (the four-digit destination belts at the far end of the building) take longer to fill. By Phase 2 the belt-loading equilibrium is reached. In Phase 3, late-arrival containers (post-trailer-pull) are disproportionately PD09–PD12 inbound from the New England network because of the late-night ORION cutoff cycle.

### 24.3 Implication for the suite

Any tracker computation that uses κ_z = 0.311 as a fixed prior — for example the projected Zone 9-12 volume tile in a pre-sort DOP module — will be biased high by ~7% during Phase 1 and biased low by ~6% during Phase 3.

**Recommended actions:**

1. **Tracker Sort Phase chip** (tracker v2.10): surface the current phase in the Hub tab using Definition 24.1. Inputs already exist in `dopComputeCurrentState` (`pctOfPlan` ≅ $V/V_{\text{plan}}$; `elapsedH` ≅ $t - t_{\text{start}}$).
2. **Phase-aware κ_z** in any projected-Zone-9-12 logic: use $\bar{\kappa}_{9-12}^{\phi}$ from Theorem 24.1.
3. **Phase-aware PPH conditional bands**: belt-level PPH targets in Phase 1 are necessarily lower than in Phase 2; the existing $[<150, 150{-}199, 200{-}249, \ge 250]$ R/A/G/B bands assume Phase 2. v1.6 leaves the specific band recalibration to v1.7 pending one more week of snapshot data.

---

## 25. Recalibration of the avgPPH Trajectory

### 25.1 The claimed series

v3.6 §13.5 (Definition 13.2) presents the iGate avgPPH trajectory **166 → 188.6 → 155** across the work week as the coordinator Bayesian belief-update sequence. The series anchors a narrative about over-correction on the Wednesday peak and pull-back on Friday.

### 25.2 Empirical reconstructions

We computed four candidate avgPPH series from the 05.04–05.08 final exports.

| Day | iGate PD-only avgPPH | iGate all-belts avgPPH | SOR Summary PPH actual | MTA final_bldg_pph |
|------|---------------------:|-----------------------:|-----------------------:|-------------------:|
| Mon | 170.0 | 89.8 | 121.9 | 131.2 |
| Tue | 162.1 | 87.6 | — | 129.4 |
| Wed | 152.5 | 89.3 | — | 127.1 |
| Thu | 154.8 | 87.5 | 119.7 | 122.6 |
| Fri | 144.8 | 84.4 | 119.4 | 113.5 |

(Tue/Wed SOR PPH unavailable — SOR Volume actual was zero in the final pulled export. SOR uses cached value when no fresh refresh.)

![Figure E — avgPPH trajectory: claimed vs. four empirical measures](figures/fig_e_avgpph_trajectory.png)

### 25.3 The 188.6 datum

No combination of (employee subset × belt subset × time aggregation) we tested reproduces 188.6 for Wed 05.06 from the Employee Summary (16).xlsx final export. The plausible explanations:

1. **Mid-sort high-water-mark.** 188.6 may have been captured from an intermediate Employee Summary snapshot during Wednesday's power hour, not the final aggregate. Without per-day archived snapshots for 05.05–05.08, this is unverifiable.
2. **PD-belt subset.** A filter restricting to PD05–PD08 (the high-velocity central belts) might yield ~188 on Wed; the final-state filter does not.
3. **Source typo.** The intended value may have been 158.6 (iGate all-belts × some scaling), which would invert the narrative arc.

Until one of these is confirmed, **v1.6 demotes the "166 → 188.6 → 155" series to anecdotal status** and proposes two replacement candidates for the coordinator Bayesian belief-update narrative in v3.6 §13.5:

- **(a) MTA-aligned series:** 131 → 129 → 127 → 123 → 114. Smooth γ-aligned decay; tracks operational PPH reported in the Dashboard. Narrative: belief drifts down consistently with realized volume; no over-correction event.
- **(b) iGate PD-only series:** 170 → 162 → 153 → 155 → 145. Has a Wed-Thu rebound (153 → 155) that may carry the same operational meaning as the original "over-correction" without the 188.6 spike.

We recommend (a) as the canonical series because it matches what the District Manager and Ops Manager see in their post-sort dashboard. (b) is the candidate if the v3.6 narrative depends on a non-monotone pattern.

---

## 26. Closing remarks for v1.6

### 26.1 What did not change

The v1.5 §§17–22 contributions remain correct:

- SEAS integration architecture (24h cadence lag, three export types, UNASSIGNED as SOR mismatch proxy) — unchanged.
- SQS five-sort calibration (MisloadRate 0.051–0.072%, LIBRate 0.112–0.170%) — recomputed and confirmed within rounding.
- Misload taxonomy (seven types) — unchanged.
- Theorem 17.1 (LIB as inverse SQS signal) — unchanged.
- Theorem 21.1 (CURE-PPH coupling at dock bottleneck) — unchanged; awaiting SLIC→PD mapping (v4.1 priority item) for full instantiation.
- Python ingestion specification (8 function signatures) — unchanged; adds one helper `compute_phase_indices(snapshots)` returning Definition 24.1 phase membership.

### 26.2 Weekly γ — corroboration

The geometric Mon→Fri ratio is 0.934 by iGate PD-belt sum and 0.940 by MTA bldg_vol — within rounding of the claimed γ = 0.938. This claim does not need revision.

![Figure B — Weekly γ decay](figures/fig_b_gamma_weekly_decay.png)

### 26.3 "Wednesday +6.5% anomaly" — week-dependent

The 05.06 Wed/Tue volume ratio was 0.981 (PD-belt) and 0.983 (MTA bldg_vol). The +6.5% anomaly claim in v2.1 Def 20.1 / v1.5 §17.4 does not hold this particular week. It may be a historical pattern (broader weekly-share data needed) or a Peak-season effect (the Wednesday before Thanksgiving classically spikes). For the off-peak May data we have, the anomaly is absent.

**Recommended phrasing:** "Volume share is more variable mid-week than at the endpoints. Tuesday and Wednesday together account for the largest cross-week amplitude swings."

### 26.4 What v1.7 will add

- Per-snapshot phase labels archived alongside each Hub Summary export, enabling phase-resolved κ_z, PPH bands, and Φz priors automatically.
- Weekly γ posterior with conjugate Beta prior, updated each Friday end-of-sort.
- ε_schema(d) empirical fit refreshed every week, exposed in the volume reconciliation banner.
- One full historical 5-week walk (after the Tailscale Phase-1 SOR watcher is in place) to test whether "Wednesday is variance day" generalizes.

---

## Appendix v1.6.A — Session Log Entry

**Session 17 — 2026-05-12 — v1.6 (Branch A).**
Re-pass over 05.04–05.08 raw exports (62 xlsx in `in_Sort_05.04.26/` plus four sister day folders) using the Python validation pipeline saved at `/Twilight 050426/validation_0504_timeline.json`. Three findings: ε_schema grows 5%→15% across the week (Theorem 23.1); κ_9-12 ramps 0.20→0.33 within a single sort (Definition 24.1, Theorem 24.1); the 188.6 mid-week avgPPH datum in v3.6 §13.5 is not reproducible from final exports (§25). Five figures generated and embedded. v1.5 §§17–22 numerical calibrations re-confirmed (SQS, LIBRate, MisloadRate, γ ≈ 0.938 within rounding). Tracker v2.10 work item created to surface Sort Phase chip + phase-aware DOP module.

## Appendix v1.6.B — New / Revised Symbols

| Symbol | Definition | First appearance |
|--------|------------|------------------|
| $\varepsilon_{\text{schema}}(d)$ | Day-indexed schema offset: $V_{PD}(d)/V_S(d) - 1$ | 23.1 |
| $V_{PD}(d), V_{NP}(d), V_S(d)$ | PD-belt, non-PD, total SOR Volume on sort $d$ | 23.2 |
| $\phi(t) \in \{0, 1, 2, 3\}$ | Sort phase function (volume-fraction definition) | 24.1 |
| $\bar{\kappa}_z^{\phi}(d)$ | Mean Zone-$z$ share during phase $\phi$ of sort $d$ | 24.1 |
| $\rho(d)$ | Residual / OB-bag double-count term in SOR ↔ Hub reconciliation | 23.2 |

---

**Document Status:** v1.6 extends v1.5 with three new sections (§§23–25) and reaffirms §§1–22 except where superseded. Total theorems and definitions: v1.5 had 18; v1.6 adds 2 theorems and 1 definition (Thm 23.1, Def 24.1, Thm 24.1).

*End of Session 17 — A Three-Timescale Mathematical Framework for Operational Training and Predictive Analytics in Large-Scale Package Sorting, v1.6 (Branch A — Empirical Recalibration).*
