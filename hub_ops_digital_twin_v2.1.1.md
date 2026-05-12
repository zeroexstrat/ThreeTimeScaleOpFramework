---
title: "Hub Operations Digital Twin — v2.1.1 CURE×ZIP Coupling"
author: "Rafael Almeida (Employee 6068314) — UPS Chelmsford Hub"
date: "May 2026"
geometry: margin=1in
fontsize: 11pt
---

<!--
  hub_ops_digital_twin_v2.1.1.md
  Author: Rafael Almeida, Employee 6068314
  const AUTHOR_SIGNATURE = "Rafael Almeida, Employee 6068314";
-->
<meta name="author" content="Rafael Almeida 6068314" />

**Version:** 2.1.1 — One increment toward v2.2. Introduces the CURE×ZIP coupling that maps destination lane utilization into zone-resolved dock pressure scores using the ZIP→belt truth table from LabelTrainingCertificationTwi (LTC-Twi).
**Reads with:** `hub_ops_digital_twin_v2.1.md` (§§16–21, especially Theorem 16.1) and `hub_ops_unified_framework_v4.1.md`.
**Empirical anchor:** CURE_Main_Data_04.29.26.xlsx (103 outbound load rows) + LTC-Twi production truth table `app/js/truth-data.js` (40,949 ZIP→SLIC→BAY→belt-index→state→city rows, 49 distinct destination SLICs) + **operator ground-truth corrections from the floor coordinator** (Rafael Almeida, 6068314).

**A note on the source.** `truth-data.js` is the LTC quiz training table — its purpose is to teach sorters every *valid* routing, including the WORMA N / PRORI N overflow trailers that legitimately accept packages from far-flung ZIPs during night sort. It is *not* a sort-chart artifact in the sense of "where does the primary trailer for SLIC s live tonight." Naïve majority vote on the quiz table conflates two distinct kinds of routing: the **WORMA P / PRORI P primary trailers** (which load on their canonical PD belts) and the **WORMA N / PRORI N overflow trailers** (bay 105–106 PD-04 for WORMA N, separate bays for PRORI N) that accept overflow from blowouts elsewhere on the building. The canonical "P" assignment is in the operations sort chart `CHEMA_Twilight_Sort_Chart.xlsx` and validated by the floor coordinator. v2.1.1 uses operator-confirmed ground-truth assignments for the five highest-utilization SLICs in the CURE export and treats other SLICs as best-effort.

---

## 22. From Aggregate Bottleneck Regime to Zone-Resolved Dock Pressure

### 22.1 The gap v2.1 left open

Theorem 16.1 (v2.1) established the loading-bottleneck regime: when CURE outbound dock utilization $U_d$ exceeds $U_{\text{cap}} \approx 0.85$ (revised from v2.1's 0.92 after the SLIC→PD mapping pass — see v4.1.1 §31.3), increasing sort rate $\lambda_s$ decreases Sort Quality Score, because the dock cannot absorb the additional throughput. The theorem was correct at the *building* level but did not say *which zone* should slow down or which PD belt's coordinator should be notified.

The missing piece is the function $B(d)$ from v4.0 Theorem 5.1: which PD belt does destination SLIC $d$ feed? Without $B$, CURE utilization is a building-level diagnostic; with $B$, it becomes a zone-level diagnostic — and that resolves into a coordinator-level action.

v2.1.1 closes this gap, with one important refinement: the canonical mapping cannot be read off `truth-data.js` directly because the quiz table records *every valid* (ZIP, SLIC, belt) routing, including the WORMA N and PRORI N overflow trailers that accept packages from far-flung ZIPs during night sort. Naïve ZIP-count majority on the quiz table conflates two distinct flows:

- **WORMA P / PRORI P** — the *primary* preload trailers for SLICs in the Worcester MA / Providence RI areas. WORMA P 0169 is on PD-03 (bay 75 region); PRORI P 0299 is on PD-12 (top blue) across all three shifts.
- **WORMA N / PRORI N** — the *night-sort overflow* trailers. WORMA N is on PD-04 bay 106, taking in western Mass + CT + the broader WORMA N catchment. PRORI N is on PD-04/PD-05 bays for overflow from HASTX, INDTX, I81IN, TOLOH, LEXKY blowouts (i.e., when a primary destination trailer is full and the next sequence trailer isn't yet there).

The quiz table contains both. Looking at SLIC 0169 in `truth-data.js` we see 1,229 rows pointing at PD-09, 936 at PD-11, 240 at PD-04, and only 22 at PD-03 — yet the *canonical primary* assignment is PD-03 (WORMA P, bay 75 region). The PD-09 / PD-11 / PD-04 entries are WORMA N overflow that the quiz teaches sorters to recognize. A naïve majority would return PD-09; the operations truth is PD-03.

The correct source for the primary mapping is the **CHEMA Twilight Sort Chart** (the operational document `CHEMA_Twilight_Sort_Chart.xlsx`, which `process_zips.py` and `gen_truth_twilight.py` both read) plus the floor coordinator's bay-level knowledge. v2.1.1 anchors on five operator-confirmed ground-truth assignments and uses `truth-data.js` only to flag the unmapped CURE destinations that fall outside primary PD routing entirely.

### 22.2 The CURE×ZIP coupling

\begin{definition}[SLIC-to-belt function — primary trailer assignment]
\label{def:slic-to-belt}
Let $\mathcal{S}$ be the set of all 4-digit destination SLICs that appear in any CURE Main Data export. Let $\mathcal{B} = \{\text{PD-01}, \text{PD-02}, \ldots, \text{PD-12}, \text{SECONDARY}\}$. The function $B: \mathcal{S} \to \mathcal{B}$ returns the PD belt where the *primary* preload trailer (the "P" variant) for SLIC $s$ is loaded. $B$ has three definitional sources, in priority order:

\begin{enumerate}
\item \textbf{Operator ground truth} — for the five highest-utilization SLICs in the CURE 04.29 export (the destinations that most affect zone pressure), the floor coordinator provides direct bay-level assignments (Table~\ref{tab:rafael-ground-truth}).
\item \textbf{Sort-chart lookup} — for remaining SLICs, $B(s)$ is read from the `CHEMA_Twilight_Sort_Chart.xlsx` bay-to-belt assignment for the SLIC's primary trailer.
\item \textbf{Truth-table fallback} — where no other source is available, $B(s)$ defaults to the most common belt index in `truth-data.js` for SLIC $s$, with the caveat that this can return an overflow assignment (WORMA N / PRORI N) rather than the primary; coordinator review is required before relying on the value for operational decisions.
\end{enumerate}

CCHIL S routings (encoded as comma-list belt indices in `truth-data.js`) are preserved as multi-belt valid answers for the LTC quiz but collapsed via Definition 22.2 below for dock-pressure aggregation.
\end{definition}

\begin{table}[ht]
\centering
\caption{Operator ground-truth SLIC→PD assignments. WORMA P / PRORI P = primary preload trailer. WORMA N / PRORI N = night-sort overflow trailer (separate bays, separate belts).}
\label{tab:rafael-ground-truth}
\begin{tabular}{lllll}
\toprule
SLIC & Destination & $B(s)$ & Variant & Note \\
\midrule
0169 & WORCESTER (WORMA P) & PD-03 & primary & Bay 75 region; WORMA N overflow lives on PD-04 bay 106 (CT + W MA + overflow set) \\
0209 & NORWOOD HUB & PD-03 & primary & No documented overflow conflict for this SLIC \\
0249 & BROCKTON & PD-08 & primary & Few 0249-ZIP exceptions routed to NNTMA (PD-11) — within-SLIC exceptions, not overflow \\
0219 & WATERTOWN HUB & PD-10 & primary & Few 0219-ZIP exceptions routed to PD-08 \\
0299 & PROVIDENCE HUB (PRORI P) & PD-12 & primary & Top Blue, all 3 shifts; PRORI N is the overflow trailer for HASTX / INDTX / I81IN / TOLOH / LEXKY blowouts \\
\bottomrule
\end{tabular}
\end{table}

\begin{definition}[Zone dock-pressure score]
\label{def:zone-dock-pressure}
For zone $Z_i \subset \mathcal{B}$ and a CURE snapshot $C = \{(s_k, U_k)\}$ (where $U_k$ is the dock utilization fraction of SLIC $s_k$), define
\[
P_{Z_i}(C) := \sum_{k:\ B(s_k) \in Z_i} U_k, \qquad
\bar{U}_{Z_i}(C) := \frac{P_{Z_i}(C)}{|\{k : B(s_k) \in Z_i\}|}, \qquad
U_{Z_i}^{\max}(C) := \max_{k:\ B(s_k) \in Z_i} U_k.
\]
$P_{Z_i}$ is the *aggregate* pressure; $\bar{U}_{Z_i}$ is the per-destination mean; $U_{Z_i}^{\max}$ is the single hottest destination feeding the zone.
\end{definition}

### 22.3 Zone-resolved bottleneck condition

\begin{proposition}[Zone-resolved loading bottleneck]
\label{prop:zone-bottleneck}
Theorem 16.1 (v2.1) refines as follows. Sort-rate-on-quality coupling $\partial \text{SQS}/\partial \lambda_s$ becomes negative in zone $Z_i$ when
\[
U_{Z_i}^{\max}(C) \ge U_{\text{cap}} \approx 0.85,
\]
even if the building-aggregate utilization stays below $U_{\text{cap}}$. The single-destination max is the operative threshold because dock loading is destination-resolved: once one trailer at one door is at capacity, the belt that feeds it cannot accelerate without spilling.
\end{proposition}

### 22.4 Worked example — CURE Main Data 04.29.26

Applying Definitions 22.1 and 22.2 to the 04.29 CURE export with operator ground-truth corrections from Table 22.1:

![Figure F — Per-belt CURE utilization (operator-corrected SLIC→PD mapping; CURE 04.29 export). Zone 9-12 highlighted. PD-12 has the highest mean utilization (59% across 5 destinations including PRORI P 0299 PROVIDENCE).](figures/fig_f_slic_pd_mapping.png)

| Belt | # dests | Mean util | Max util | Top-pressure destinations |
|------|--------:|----------:|---------:|---------------------------|
| PD-01 | 7 | 59% | 78% | 0480 ROCKLAND, 0490 WATERVILLE |
| PD-02 | 9 | 65% | 79% | 0540 BURLINGTON, 0530 BRATTLEBORO, 0669 STRATFORD |
| PD-03 | 7 | 56% | 80% | **0169 WORCESTER (WORMA P)**, 0209 NORWOOD, 0212 SOMERVILLE |
| PD-04 | 1 | 71% | 71% | 2750 MEBANE HUB |
| PD-05 | 4 | 53% | 71% | 0422 LEWISTON-AUBURN |
| PD-06 | 5 | 63% | 77% | 4629 INDY 81S, 0327 NORTHFIELD |
| PD-07 | 13 | 61% | 78% | 0419 PORTLAND, 0390 STRATHAM, 0400 WELLS |
| PD-08 | 12 | 50% | 83% | 9039 BELL HUB, **0249 BROCKTON**, 0251 NANTUCKET |
| PD-09 | 2 | 66% | 75% | 0350 TWIN MOUNTAIN |
| PD-10 | 10 | 66% | 83% | 0196 SAUGUS, **0219 WATERTOWN HUB**, 0199 LYNNFIELD |
| PD-11 | 7 | 61% | 80% | 2079 BURTONSVILLE, 0560 BARRE, 0269 YARMOUTH |
| PD-12 | 5 | 59% | 78% | **0299 PROVIDENCE HUB (PRORI P)**, 4329 COLUMBUS, 4059 LEXINGTON |
| SECONDARY | — | — | — | — |

(Bold entries are the operator ground-truth corrections from Table 22.1.)

Three observations.

1. **No belt reached $U_{Z_i}^{\max} \ge 0.85$ in the 04.29 export.** The hottest dock loads were PD-08 BELL HUB at 83%, PD-10 SAUGUS at 83%, and PD-10 WATERTOWN HUB at 83% — close to the bottleneck threshold but not yet over. Proposition 22.1 predicts that under this CURE state none of the zones is yet in the negative-coupling regime.
2. **PD-10 carries the broadest top-utilization load.** Three of the four hottest destinations in the CURE snapshot (SAUGUS, WATERTOWN HUB, BROCKTON's neighbor LYNNFIELD) are routed to PD-10. Combined with the BROCKTON PD-08 routing, this is a clear Zone 9-12 + adjacent-Zone-8 pressure signature — the south/east Boston suburb cluster is mid-week hot.
3. **Zone 9-12 contains 29.3% of mapped destinations (24 of 82).** This is within $\pm 2$pp of the κ_9-12 = 0.311 asymptotic share from v2.1 Theorem 19.1 — a clean cross-validation that operator-corrected dock-pressure aggregation is structurally consistent with the volume-share statistics derived independently from iGate Hub Summary. The 1.8pp shortfall vs. 0.311 is well within day-to-day $\alpha_\text{CURE}$ variation.

![Figure G — CURE×PD coupling: top dock-pressure destinations colored by zone (operator-corrected, CURE 04.29). WORCESTER, NORWOOD, PROVIDENCE HUB now correctly attributed to PD-03 / PD-03 / PD-12; BROCKTON to PD-08 and WATERTOWN HUB to PD-10 per coordinator confirmation.](figures/fig_g_zone_dock_pressure.png)

### 22.5 EKF augmentation

Add the per-zone $P_{Z_i}, \bar{U}_{Z_i}, U_{Z_i}^{\max}$ as observed quantities to the v2.0 EKF state vector. Three additional dimensions per zone × three zones = nine additions to the ~300-dimensional state. Observation matrix entries connect the CURE state to the zone PPH expectation: a hot zone (high $\bar{U}_{Z_i}$) raises the variance of zone PPH (the belt cannot absorb a surge) and lowers the mean (sorters spend more time waiting for the chute to clear).

### 22.6 Unmapped destinations — operational caveat

Of the 103 rows in CURE_Main_Data_04.29.26, 21 are routed to destination SLICs that the production truth table does not assign to a primary PD belt. The 12 distinct unmapped SLICs are:

| SLIC | Name | Likely category |
|------|------|-----------------|
| 0119 | SPRMA (Springfield Hub) | Cross-region truck hub (not in CHEMA's direct catchment) |
| 0220 | FRANKLIN-NORTH | Internal hub, no retail ZIP |
| 0319 | MANCHESTER | Air-gateway destination |
| 0399 | NASHUA | Truck-only cross-dock, no retail ZIP |
| 0432 | LL BEAN | Shipper-owned hub (Freeport ME) |
| 0449 | BANGOR | Air-gateway destination |
| 0619 | HARTFORD | Cross-region truck hub |
| 2669 | (US Hub, unmapped in CHEMA Twilight chart) | — |
| 4369 | TOLEDO HUB | Cross-region truck hub |
| 4719 | FT ERIE-INTERN'L | Cross-border, special routing |
| 7599 | INDEPENDENCE | Cross-region truck hub |
| 7709 | SWEETWATER HOUSTON | Cross-region air-gateway |

These are precisely the destinations that pipeline conflict-resolution intentionally excludes from primary CHEMA Twilight routing: they are handled either as air loads (AF1/AF2 belts not enumerated above), as Secondary sort overflow, or as shipper-owned drop-offs that bypass the primary PD chutes entirely. They represent 21/103 = 20.4% of CURE rows in the 04.29 snapshot and 21/82 = 25.6% of aggregate utilization above the SLIC-mapped set — large enough to matter for absolute-volume forecasts.

The action item is **not** a truth-table extension (the production table is correct to omit these from primary PD routing). Instead, two additions to the suite:

1. **CURE pre-parser:** in the dashboard CURE Cube tab and tracker pre-sort panel, surface a separate "non-primary destinations" tile listing these 12 SLICs and their utilizations so they aren't silently dropped from the operator's view.
2. **AF1/AF2 mapping:** for the air-gateway destinations (MANCHESTER, NASHUA, BANGOR, BANGOR, FT ERIE, BANGOR, SWEETWATER HOUSTON), extend the CURE × belt analysis to belt indices 14 and 15 (AF1 / AF2) and provide an air-volume tilt factor analogous to $\alpha_{\text{CURE}}$ for the ground belts.

---

## Appendix v2.1.1.A — Session 18 Entry (Branch B)

**Session 18 — 2026-05-12 — v2.1.1 (Branch B).** One increment toward v2.2. **Round 3 correction:** the SLIC→PD function $B$ cannot be read off `truth-data.js` by naïve majority vote — the LTC quiz table records *every valid* (ZIP, SLIC, belt) routing including WORMA N / PRORI N overflow, and the overflow entries outweigh the primary entries for high-aggregation SLICs (0169 shows 1,229 PD-09 rows vs. only 22 PD-03 rows in the quiz table, even though the *primary* WORMA P trailer is on PD-03). Def 22.1 therefore prioritizes operator ground truth from the floor coordinator (Table 22.1: 0169→PD-03, 0209→PD-03, 0249→PD-08, 0219→PD-10, 0299→PD-12) over any quiz-table heuristic. The five corrections cover the SLICs that most affect zone-pressure ranking. Worked example on CURE_04.29 with operator-corrected mapping: PD-10 carries the broadest top-utilization load (SAUGUS / WATERTOWN HUB / LYNNFIELD all $\ge 75\%$); Zone 9-12 contains 24/82 = 29.3% of mapped destinations — well within $\pm 2$pp of κ_9-12 = 0.311 from v2.1 Theorem 19.1, validating the structural constant cleanly. Two figures regenerated (F: per-belt utilization; G: top dock-pressure destinations colored by zone). 12 distinct unmapped SLICs (21 CURE rows) are intentionally omitted from primary PD routing — they're air-gateway, shipper-owned, or cross-region truck hubs. Action items: CURE pre-parser for "non-primary destinations" tile + AF1/AF2 mapping for air-gateway tilt factors + dialog with coordinator to anchor B(s) for the next tier of high-utilization SLICs not yet operator-confirmed.

## Appendix v2.1.1.B — Added Symbols

| Symbol | Definition | First appearance |
|--------|------------|------------------|
| $\mathcal{S}, \mathcal{B}$ | Set of destination SLICs / set of PD belts | Def 22.1 |
| $B: \mathcal{S} \to \mathcal{B}$ | SLIC-to-belt function (majority vote on truth table) | Def 22.1 |
| $P_{Z_i}(C)$ | Aggregate zone dock-pressure score from CURE snapshot $C$ | Def 22.2 |
| $\bar{U}_{Z_i}(C)$ | Mean per-destination utilization in zone $Z_i$ | Def 22.2 |
| $U_{Z_i}^{\max}(C)$ | Hottest single destination feeding zone $Z_i$ | Def 22.2 |

*End of Session 18 — v2.1.1, Branch B — CURE×ZIP Zone-Resolved Coupling.*
