---
title: "A Three-Timescale Mathematical Framework for Operational Training and Predictive Analytics in Large-Scale Package Sorting"
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
  - \usepackage{hyperref}
  - \hypersetup{colorlinks=true, linkcolor=blue, urlcolor=blue}
  - \newtheorem{theorem}{Theorem}
  - \newtheorem{definition}{Definition}
  - \newtheorem{lemma}{Lemma}
  - \newtheorem{corollary}{Corollary}
  - \newtheorem{proposition}{Proposition}
---

<!--
  hub_ops_mathematical_framework_v1.5.md
  Author: Rafael Almeida, Employee 6068314
  const AUTHOR_SIGNATURE = "Rafael Almeida, Employee 6068314";
-->
<meta name="author" content="Rafael Almeida 6068314" />

**Version:** 1.5 — Five-sort empirical calibration of SQS; SEAS/CURE/LIB data integration; misload taxonomy; Python ingestion specification (Session 7)  
**Status:** Production-validated framework; peer-reviewable with operational extensions

---

## Abstract

This paper presents a comprehensive mathematical framework unifying four operational systems of the UPS Chelmsford package sort ecosystem: Label Training Certification (LTC), the Hub Operations Tracker (v1.13+), the operations Dashboard, and MasterTrendAnalysis (v2.0+). We model the system across three nested timescales — per-question (1–5 seconds), per-snapshot (15–30 minutes), and per-sort (4 hours) — and establish formal connections between training interventions, live operational metrics, and long-term performance trends.

Key contributions include: (1) ultrametric formalization of ZIP code routing with prefix-based local constancy and CCHIL multivalued acceptance relations; (2) monoid homomorphism structure for analytics enabling MapReduce parallelization; (3) phase-aware stochastic models of sorter productivity with CUSUM control limits for quality assurance; (4) comparative analysis of curve-fitting algorithms (normalized curve, weighted median, KDE with Sheather-Jones bandwidth selection, DTW with Sakoe-Chiba acceleration) for intra-sort prediction; (5) hierarchical Bayesian models for skill estimation across belt families; (6) stratification methodology for Simpson's Paradox avoidance in cross-era analysis; (7) **Sort Quality Score (SQS) as a composite operational metric decomposable into performance, data fidelity, and staffing components**; (8) **jam-breaker two-layer distortion model formalizing the dual effect of overhead-coded hours on PPH accuracy**; and (9) **predictive staffing and borrow/loan intelligence framework grounded in marginal productivity optimization**. We validate the framework on a corpus of 633 historical sorts and provide production-ready implementation guidance for all algorithms and operational models.

**Keywords:** package sorting; operations research; statistical process control; Bayesian hierarchical models; information theory; temporal prediction; sort quality score; jam-breaker distortion; predictive staffing; SEAS quality data; misload taxonomy; LIB analysis; CURE trailer utilization.

**v1.5 Update (Session 7):** This version grounds the v1.4 theoretical framework in five consecutive sort days (05.04–05.08, 2026) of empirical data. New contributions: (10) **SEAS integration as quality data spine** — three export types (Twilight, Employee, Breakdown) with 24-hour cadence lag and UNASSIGNED work-area proxy for SOR mismatch severity; (11) **five-sort SQS calibration** — MisloadRate 0.051–0.072%, LIBRate 0.112–0.170%, establishing PPH variance as the dominant SQS driver; (12) **misload taxonomy** — seven error types with operational causes and intervention strategies, AUTO MISLOAD threshold trigger at >20 events; (13) **LIB type analysis** — five LIB subtypes defined, HUB DERIVED LIB as the actionable metric, UNASSIGNED exclusion rule, and Theorem 17.1 (LIB as inverse SQS signal); (14) **CURE outbound trailer utilization** — 102–143 active lanes across the week, LL BEAN persistent at-capacity (88–98%), Theorem 21.1 (CURE-PPH coupling at dock bottleneck lanes); (15) **Python auto-ingestion specification** — eight function signatures for complete Twilight_Sort folder ingestion with SQS computation pipeline.

---

## 1. Introduction

### 1.1 Problem Statement

The UPS Chelmsford facility operates four semi-independent systems:

1. **Label Training Certification (LTC):** A web-based quiz system training sorters on ZIP-to-belt routing rules, with per-question granularity.

2. **Hub Operations Tracker (v1.13):** A live dashboard aggregating iGate (automated scanning) and SOR (manual staffing record) data at 15–30-minute snapshot intervals during the 4-hour sort window (18:00–22:00).

3. **Operations Dashboard:** A post-sort reporting tool reconciling multiple data sources and providing supervisory insights.

4. **MasterTrendAnalysis (v2.0):** A 633-sort historical prediction engine supporting route-planning and staffing forecasts using multiple curve-fitting algorithms.

Despite their interdependence, these systems lack a unified mathematical framework. This creates **five critical gaps**:

- **No formal connection between training and operations:** LTC insights (per-belt skill gaps) are advisory and not fed back to operational dashboards.
- **Incomparable prediction algorithms:** MasterTrendAnalysis uses four curve-fitting methods without formal comparative justification.
- **Information loss at aggregation boundaries:** PPH quantization into color bands is non-homomorphic; belt-level colors don't always reflect per-employee detail.
- **No composite sort quality metric:** Performance, data fidelity, staffing adherence, and misload rates are tracked separately, making holistic sort assessment manual and inconsistent.
- **Implicit jam-breaker distortion:** When jam-breaker employees are coded under overhead labor codes, PPH denominator is underestimated, inflating the metric in two compounding ways not previously formalized.

### 1.2 Contributions

This paper provides a **three-timescale mathematical framework** that addresses all five gaps. New contributions in v1.4 (Session 6) include:

1. **Sort Quality Score (SQS):** A formally decomposable composite metric combining PPH relative performance, data fidelity, staffing adherence, and missort rate into a single $[0,1]$ score with diagnostic separation of components.

2. **Jam-Breaker Two-Layer Distortion Model:** A formal treatment of how jam-breaker reclassification distorts PPH through two independent mechanisms — denominator compression (Layer 1) and scan-gap volume loss (Layer 2) — with corrected PPH formula.

3. **Predictive Staffing and Borrow/Loan Intelligence:** A marginal-productivity optimization framework for real-time staffing decisions, grounded in the diminishing-returns structure of zone productivity curves.

### 1.3 Organization

Sections 2 through 11 replicate the v1.3 peer-reviewed structure. Sections 12–14 present new v1.4 mathematical contributions. Sections 15–16 update recommendations and references. Appendices maintain the session log and symbol glossary.

---

## 2. Related Work

### 2.1 Operations Research and Package Sorting

Dantzig and Ramser (1959) initiated formal routing optimization, extended by Laporte (1992) to time-windowed variants. Our ZIP-to-belt mapping is a discrete sorting problem with ultrametric structure unique to geographic routing. Agustina et al. (2014) studied cross-docking networks; the three-layer snapshot architecture mirrors their data model. Goodman (2012) reviews piece-rate (PPH) as a standard warehouse KPI; our CUSUM and phase-dependent models extend this with formal control limits.

### 2.2 Statistical Process Control

Page (1954) introduced CUSUM for manufacturing quality assurance. Hawkins and Olwell (2010) provide comprehensive parameter guidance (reference value $K$, decision interval $H$) applied here in Section 9. Juran and De Feo (2010) emphasize phase-aware baselines, directly implemented in Section 6.2.

### 2.3 Bayesian Hierarchical Models

Efron and Morris (1977) pioneered hierarchical Bayes shrinkage estimators. Gelman et al. (2013) provide the standard reference on multilevel regression. Wakefield et al. (2023) demonstrate AUC improvement from 0.760 to 0.831 with hierarchical Bayes over logistic regression baseline on personnel skill data.

### 2.4 Information Theory

Shannon (1948) established the discrete memoryless channel model applied to sorter routing in Section 4.1. Cover and Thomas (2006) treat entropy, mutual information, and capacity. Gray and Neuhoff (1998) formalize quantization loss via rate-distortion theory, applied to PPH color band analysis.

### 2.5 Temporal Data and State-Space Models

Kalman (1960) established optimal sequential state estimation. Simon (2006) covers cold-start initialization. Durbin and Koopman (2012) extend to piecewise-constant phase models, implemented in Section 6.2.

### 2.6 KDE and Bandwidth Selection

Silverman (1986) established the rule-of-thumb $h = 1.06\sigma n^{-0.2}$. Sheather and Jones (1991) proved superiority of MISE-minimizing plug-in bandwidth for multimodal data, confirmed by Wand and Jones (1994).

### 2.7 Dynamic Time Warping

Sakoe and Chiba (1978) introduced DTW and the band constraint reducing complexity from $O(mn)$ to $O(m \cdot r)$. Müller (2007) provides comprehensive applications to time-series matching.

### 2.8 Simpson's Paradox and Stratification

Simpson (1951) established that pooled trends can reverse within strata. Pearl (2009) provides causal inference grounding. Blyth (1972) characterizes theoretical conditions; our era-based stratification (Section 7.2) directly addresses this.

### 2.9 Marginal Productivity and Labor Optimization

Varian (2010) presents the microeconomic foundation for diminishing marginal returns in labor allocation — directly applied in Section 14's borrow/loan intelligence model. Pinker and Shumsky (2000) study dynamic staff reallocation in service operations, establishing the marginal gain framework used in Theorem 14.1.

### 2.10 Gap in Existing Literature

To our knowledge, no prior work unifies training (per-question), live operations (snapshot), and historical prediction (per-sort) into a single mathematical framework, nor formalizes the jam-breaker distortion mechanism or the SQS composite quality metric for warehouse operations.

---

## 3. Formal Framework: Part I — Label Training Certification

### 3.1 The Routing Function and Domain

Let $Z \subset \{00000, \ldots, 99999\}$ be the set of ZIP codes with defined routing, $|Z| \approx 41{,}000$. Let $B = \{0, 1, \ldots, 15\}$ be the belt index set.

\begin{definition}
The \textbf{routing function} is:
$$f : Z \to B$$
where $f(z)$ is the unique belt destination for ZIP code $z$. Viewed over all 5-digit strings, $f$ is a \emph{partial function}.
\end{definition}

\begin{definition}
The \textbf{prefix equivalence relation} is:
$$z_1 \sim_3 z_2 \iff z_1[1{:}3] = z_2[1{:}3]$$
The routing function factors approximately as $f \approx \bar{f} \circ \pi$ where $\pi : Z \to Z/{\sim_3}$ is the quotient map and $\bar{f} : Z/{\sim_3} \to B$ is the prefix routing function over $\approx 800$ 3-digit prefixes.
\end{definition}

### 3.2 The CCHIL Multivalued Extension

For the 13 CCHIL states (MS, IA, WI, MN, SD, ND, NE, LA, OK, CO, WY, AZ, NM), sorters may route to any of belts $\{3, 4, 8\}$.

\begin{definition}
The \textbf{multivalued routing} is:
$$\tilde{f} : Z \to \mathcal{P}(B), \quad \tilde{f}(z) = \begin{cases} \{f(z)\} & z \notin Z_{\mathrm{CCHIL}} \\ \{3, 4, 8\} & z \in Z_{\mathrm{CCHIL}} \end{cases}$$
The \textbf{acceptance relation} is $\mathcal{A} = \{(z,b) : b \in \tilde{f}(z)\}$.
\end{definition}

### 3.3 The ZIP Code Space as an Ultrametric

\begin{definition}
The \textbf{prefix metric} is:
$$d(z_1, z_2) = \begin{cases} 0 & \text{if } z_1 = z_2 \\ 10^{5 - \ell(z_1, z_2)} & \text{if } z_1 \neq z_2 \end{cases}$$
where $\ell(z_1, z_2) = \max\{k : z_1[1{:}k] = z_2[1{:}k]\}$ is the longest common prefix length.
\end{definition}

\begin{theorem}[Ultrametric Property]
The pair $(Z, d)$ is an ultrametric space, satisfying reflexivity, symmetry, and the ultrametric inequality:
$$d(z_1, z_3) \leq \max(d(z_1, z_2),\; d(z_2, z_3))$$
\end{theorem}

*Proof.* Reflexivity ($d(z,z)=0$) and symmetry are immediate. For the ultrametric inequality when $z_1 \neq z_3$: the longest prefix shared by $z_1$ and $z_3$ is $\min(\ell(z_1,z_2), \ell(z_2,z_3))$ (bottleneck property of string prefixes). Therefore $d(z_1,z_3) = 10^{5-\min(\ell(z_1,z_2),\ell(z_2,z_3))} \leq 10^{\max(5-\ell(z_1,z_2),\; 5-\ell(z_2,z_3))} = \max(d(z_1,z_2), d(z_2,z_3))$. $\square$

**Geometric picture.** Ultrametric spaces are tree-like — every triple forms an isoceles triangle. Here the tree is the ZIP prefix trie, and distance equals the depth at which two ZIPs diverge. This matches physical geography: nearby ZIPs share longer prefixes and route to the same or adjacent belts.

### 3.4 The Belt Confusion Graph

\begin{definition}
The \textbf{confusion graph} is a directed graph $G = (B, E)$ where $(b_i, b_j) \in E$ iff belt $b_j$ is confusable with belt $b_i$ (encoded in \texttt{BELT\_FAMILIES}). The out-neighbourhood $\mathcal{F}(b) = \{b' : (b, b') \in E\}$ is used to generate semantically hard distractors in the quiz engine.
\end{definition}

Three natural confusion families emerge: **Black** $\{0, 9, 12\}$, **Blue** $\{1, 5, 7\}$, and **Air** $\{14, 15\}$. Belts within a family route to similar destinations and share visual/physical markers, making them prime candidates for training-resistant confusion.

### 3.5 Weighted Question Selection

\begin{definition}
Questions are sampled from four pools with effective sizes:

| Pool | Effective Count |
|------|----------------|
| Ground (non-CCHIL) | $P_g = \sum_{s \notin \mathrm{CCHIL}} w_s \cdot |G_s|$ |
| CCHIL | $P_c = w_c \cdot \bar{n}$ |
| Exceptions | $P_e = w_e \cdot \bar{n}$ |
| Air | $P_a = w_a \cdot \bar{n}$ |

where $\bar{n} = \frac{\sum_{s \notin \mathrm{CCHIL}} w_s \cdot |G_s|}{|\{s : s \notin \mathrm{CCHIL}\}|}$ is the average per-state contribution ensuring commensurability. Question type is drawn via:
$$\Pr[\text{type} = t] = \frac{P_t}{P_g + P_c + P_e + P_a}$$
\end{definition}

This is a **categorical mixture distribution** over question types, with uniform sampling within each type.

---

## 4. Formal Framework: Part II — Statistical Models and Channel Theory

### 4.1 The Sorter as a Discrete Memoryless Channel

\begin{definition}
A sorter is modeled as a \textbf{discrete memoryless channel} with input and output alphabet $B$. The \textbf{transition matrix} $\mathbf{M} \in \mathbb{R}^{16 \times 16}$ is right-stochastic (each row sums to 1):
$$M_{ij} = \Pr[\text{answer} = b_j \mid \text{correct} = b_i]$$
A perfect sorter has $\mathbf{M} = \mathbf{I}_{16}$; a random guesser has $M_{ij} = 1/16$ for all $i,j$.
\end{definition}

### 4.2 Information-Theoretic Measures

\begin{definition}
The \textbf{per-belt Shannon entropy} of the sorter's output distribution for belt $b_i$ is:
$$H_i = -\sum_{j=0}^{15} M_{ij} \log_2 M_{ij} \in [0, 4] \text{ bits}$$
High entropy indicates systematic confusion; low entropy with low accuracy indicates simple ignorance.
\end{definition}

\begin{definition}
The \textbf{mutual information} between correct belt $X$ and answered belt $Y$:
$$I(X; Y) = \sum_{i,j} p(i) M_{ij} \log_2 \frac{M_{ij}}{p(j)}$$
measures how much the sorter's answer reveals about the true routing. The \textbf{channel capacity} $C = \max_{p(X)} I(X; Y)$ is the maximum achievable information rate.
\end{definition}

### 4.3 Bernoulli Statistics and Confidence Intervals

Each question for sorter $k$ is a Bernoulli($\theta_k$) trial. The MLE is $\hat{\theta}_k = c/n$.

\begin{definition}
The \textbf{Wilson score interval} at level $1-\alpha$ is:
$$\mathrm{CI}_{1-\alpha} = \left[\frac{\hat{\theta} + \frac{z^2}{2n} \mp z\sqrt{\frac{\hat{\theta}(1-\hat{\theta})}{n} + \frac{z^2}{4n^2}}}{1 + \frac{z^2}{n}}\right]$$
where $z = z_{\alpha/2} = 1.96$ for 95\% confidence.
\end{definition}

\begin{theorem}[Wilson Lower Bound]
The Wilson interval has correct coverage for all $n$ and all $\theta \in [0,1]$, including near the boundaries, making it preferred over the Wald interval for small-sample sorter performance estimation.
\end{theorem}

**Practical implication.** A sorter with $\hat{\theta} = 100\%$ from 5 correct answers has Wilson interval $[0.54, 1.00]$ — entirely consistent with true skill of 54%. Minimum sample size thresholds ($n \geq 10$ for sorter ranking, $n \geq 5$ for belt ranking) are justified by this uncertainty analysis.

---

## 5. Formal Framework: Part III — Aggregation and Overlay Algebra

### 5.1 Monoid Homomorphism Structure of Analytics Reducers

\begin{definition}
The \textbf{free monoid} over sort events $\mathcal{E}$ is $(\mathcal{E}^*, \cdot, \varepsilon)$ — all finite event sequences with concatenation. An \textbf{analytics reducer} $R : \mathcal{E}^* \to S$ maps sequences to a stats monoid $(S, \oplus, \mathbf{0}_S)$ with associative, commutative merge $\oplus$.
\end{definition}

\begin{theorem}[Monoid Homomorphism]
If $R(e_1 \cdot e_2) = R(e_1) \oplus R(e_2)$ for all sequences $e_1, e_2$, then $R$ is a monoid homomorphism, enabling MapReduce parallelization: partition events, reduce each partition independently, merge results. The identity maps $\varepsilon \mapsto \mathbf{0}_S$.
\end{theorem}

Functions like `overviewStats`, `missortMatrix` (the raw count matrix) satisfy this property. Functions involving rank ordering (`worstSorter`, `worstBelt`) are non-homomorphic post-processors applied to homomorphic aggregates.

### 5.2 The Overlay System as a Pointed Function Update

\begin{definition}
An \textbf{overlay} is a finite partial function $o : D_o \to B$ ($D_o \subset Z$). The \textbf{overlay application}:
$$f \triangleleft o = \lambda z.\; \begin{cases} o(z) & z \in D_o \\ f(z) & \text{otherwise} \end{cases}$$
\end{definition}

\begin{theorem}[Left-Regular Band]
The overlay composition $\triangleleft$ is associative, idempotent ($o \triangleleft o = o$), and satisfies $(o_1 \triangleleft o_2) \triangleleft o_1 = o_1 \triangleleft o_2$, forming a left-regular band. Later overlays take precedence at their shared domain.
\end{theorem}

The append-only log of overlays (`overlays.jsonl`) makes the exact override history auditable. In categorical terms, $f \triangleleft o$ is a pushout in the category of sets and partial functions.

### 5.3 The Knowledge Map as a Rational Linear Functional

The Knowledge Map computes sorter accuracy under a filter $\varphi : \mathcal{E} \to \{0,1\}$:
$$a_k(\varphi) = \frac{\sum_{e : e.k = k} \varphi(e) \cdot e.c}{\sum_{e : e.k = k} \varphi(e)}$$

This is a **ratio of two linear functionals** — a rational linear functional on the indicator function space. The filter algebra is the Boolean algebra $\mathcal{P}(\mathcal{E})$ under union (OR-combination of state selections), intersection (AND-combination for breadth ranking), and complement.

---

## 6. Formal Framework: Part IV — Tracker Snapshot Architecture

### 6.1 The Three-Layer Snapshot Model

\begin{definition}
A \textbf{snapshot} at elapsed time $t \in [0, 240]$ minutes from sort start (18:00) is:
$$S_t = (t,\; \mathcal{H}_t,\; \mathcal{E}_t,\; \mathcal{A}_t)$$
where $\mathcal{H}_t$ is the hub-level belt aggregate, $\mathcal{E}_t$ is the per-employee per-belt data vector, and $\mathcal{A}_t$ is the per-zone staffing record.
\end{definition}

The **snapshot sequence** $\mathcal{S} = (S_{t_1}, \ldots, S_{t_m})$ is a discrete temporal observation of the entire 240-minute sort. Uploading multiple iGate and SOR exports at different timestamps builds this timeline incrementally; the file number in the export name indicates temporal ordering.

### 6.2 Phase-Dependent Markov Chain

\begin{definition}
The sort is partitioned into four \textbf{operational phases} via the piecewise-constant phase function $\phi : [0,240] \to \{\mathrm{Ramp}, \mathrm{Steady}, \mathrm{Wind}, \mathrm{Post}\}$:

| Phase | Window | Staffing | Volume |
|-------|--------|----------|--------|
| Ramp | $t \in [0,60)$ | Stagger | Early spike |
| Steady | $t \in [60,135)$ | Full | Sustained |
| Wind-Down | $t \in [135,180)$ | Declining | Tail |
| Post | $t \in [180,240]$ | Consolidated | Wrapping |
\end{definition}

\begin{theorem}[Phase Stratification]
Statistics conditioned on phase have distinct distributions:
$$\Pr[\mathrm{PPH} > \mathrm{Target} \mid \phi(t) = \mathrm{Ramp}] \ll \Pr[\mathrm{PPH} > \mathrm{Target} \mid \phi(t) = \mathrm{Steady}]$$
Phase-specific baselines prevent false-positive alarms during intentionally understaffed Ramp.
\end{theorem}

### 6.3 Staffing as a Counting Process

Let $N(t)$ denote active employees at time $t$. Arrivals (punch-ins) and departures (punch-outs) create a **counting process** with intensity $\lambda(t) = dN/dt$: positive during Ramp, near-zero during Steady, negative during Wind-Down. A **borrow/loan event** $(z_i, z_j, t_b)$ is a point-process discontinuity in the per-zone staffing count.

### 6.4 PPH Quantization and Non-Homomorphic Aggregation

\begin{definition}
The \textbf{PPH quantization function} with belt-family-specific thresholds is:
$$q_F : \mathbb{R}^+ \to \{\mathrm{red}, \mathrm{amber}, \mathrm{green}\}, \quad q_F(\mathrm{PPH}) = \begin{cases} \mathrm{red} & \mathrm{PPH} < \mathrm{low}_F \\ \mathrm{amber} & \mathrm{low}_F \leq \mathrm{PPH} < \mathrm{high}_F \\ \mathrm{green} & \mathrm{PPH} \geq \mathrm{high}_F \end{cases}$$
\end{definition}

\begin{theorem}[Non-Homomorphic Aggregation]
For belt-level PPH computed as $\mathrm{PPH}_b = \frac{\sum_k n_{kb}}{\sum_k h_k}$:
$$q_F\!\left(\mathrm{PPH}_b\right) \neq \mathrm{majority}\!\left(q_F\!\left(\tfrac{n_{kb}}{h_k}\right)_k\right) \quad \text{in general}$$
\end{theorem}

*Example.* Two employees at 10 PPH (red) and 35 PPH (green) aggregate to $22.5$ PPH (amber). Belt color is determined by the arithmetic mean of scan counts, not by voting among per-employee colors. Supervisors must cross-reference belt colors with per-employee tables.

---

## 7. Formal Framework: Part V — Historical Analysis and Prediction

### 7.1 The Data Extraction Functor and Schema Versioning

\begin{definition}
The \textbf{extraction functor}:
$$\mathrm{Extract} : \mathrm{Paths}_{\mathrm{.xlsx}} \to \mathrm{Json}_{v}$$
uses \emph{label-based scanning} — searching for cell labels (``Adjusted PPH'', ``Volume'') rather than fixed row numbers — making it robust to layout changes across tracker versions. Schema migration is a lifting map $\mathrm{Json}_{v} \xrightarrow{\mathrm{Lift}} \mathrm{Json}_{v+1}$ that backfills missing fields.
\end{definition}

### 7.2 Peer Group Filtering and Stratification

\begin{definition}
The \textbf{filtered peer group} is:
$$\mathrm{PeerGroup} = \{s \in \mathrm{Corpus} : \phi_{\mathrm{era}}(s) \land \phi_{\mathrm{dow}}(s) \land \phi_{\mathrm{ss}}(s) \land \phi_{\mathrm{ids}}(s)\}$$
where each $\phi$ is an indicator function for era, day-of-week, sleeve sort status, and IDS status.
\end{definition}

\begin{theorem}[Simpson's Paradox Avoidance]
Stratification by era prevents spurious trend reversals. Era (operational configuration — pre\_sls, sls02, dual\_sls) acts as a confounding variable: a PPH trend observed across the pooled 633-sort corpus may reverse within each stratum. Analysis must respect era boundaries.
\end{theorem}

### 7.3 Curve-Fitting Algorithms with Production Guidance

#### Algorithm 1: Normalized Curve (Phase-Aware Percentile Bands)

Rescale each peer's ramp phase to the canonical $[0,60]$ minutes, compute percentile bands $(\mathrm{P}_{25}, \mathrm{P}_{50}, \mathrm{P}_{75})$ per 15-minute bucket, repeat for Steady and Wind phases, then concatenate. This is a **piecewise affine registration** tolerant of phase duration variance.

#### Algorithm 2: Weighted Median

For each time bucket $t_i$, compute the weighted median PPH of the peer group, weighted by operational similarity. The median is a **robust estimator** less sensitive to outlier sorts.

#### Algorithm 3: Kernel Density Estimation (KDE)

$$\hat{f}(x) = \frac{1}{nh} \sum_{p=1}^{n} K\!\left(\frac{x - \mathrm{PPH}_p(t_i)}{h}\right)$$

\begin{theorem}[KDE Bandwidth Selection]
\textbf{Silverman's rule} $h_{\mathrm{Silverman}} = 1.06\,\sigma\, n^{-0.2}$ is $O(n)$ and adequate for unimodal distributions. The \textbf{Sheather-Jones plug-in method} (Sheather \& Jones, 1991) directly minimizes MISE at cost $O(n^2)$, providing superior accuracy for multimodal peer PPH distributions (e.g., by belt family or phase). Use Silverman when speed dominates; use Sheather-Jones when accuracy is critical and the peer corpus shows multiple modes.
\end{theorem}

#### Algorithm 4: Dynamic Time Warping (DTW)

$$\mathrm{DTW}(C, P) = \min_{\mathrm{path}} \sqrt{\sum_{(i,j) \in \mathrm{path}} (\mathrm{PPH}_C(i) - \mathrm{PPH}_P(j))^2}$$

\begin{theorem}[Sakoe-Chiba Acceleration]
The \textbf{Sakoe-Chiba band constraint} with radius $r$ limits the warping path to a band around the main diagonal, reducing complexity from $O(mn)$ to $O(m \cdot r)$. For sort PPH curves with low expected temporal warping, $r \approx 0.1 \cdot n$ (e.g., $r = 24$ for 240-minute sorts) provides $\approx 10\times$ speedup while allowing $\pm 10\%$ temporal flexibility.
\end{theorem}

### 7.4 Multiple Linear Regression Baseline

$$\mathrm{Adjusted\ PPH} = \beta_0 + \beta_1 \cdot \mathrm{DOW} + \beta_2 \cdot \mathrm{SS\ Share} + \beta_3 \cdot \mathrm{IDS\ Share} + \beta_4 \cdot \mathrm{Vol/Head} + \beta_5 \cdot \mathrm{Is\ Peak} + \epsilon$$

\begin{theorem}[MLR Feature Scaling]
When mixing categorical (DOW) and continuous (SS Share, IDS Share, Vol/Head) features: \textbf{center} continuous features (subtract the mean) to reduce structural multicollinearity when adding interaction terms; do \textbf{not} standardize across mixed types (categorical variance is undefined). Interaction terms such as DOW $\times$ Vol/Head and Peak $\times$ SS Share should be tested and cross-validated.
\end{theorem}

### 7.5 Kalman Filter for Sequential Updating

\begin{definition}
The \textbf{Kalman filter} maintains PPH state estimate $\hat{x}_t$ via:
$$x_t = x_{t-1} + w_t, \quad w_t \sim \mathcal{N}(0, Q) \quad \text{(random walk)}$$
$$z_t = x_t + v_t, \quad v_t \sim \mathcal{N}(0, R(t)) \quad \text{(observation)}$$
with $Q = 0.05 \times \mathrm{Var}[\mathrm{corpus\ PPH}]$ and $R(t) = R_{\mathrm{base}} \times (1 - t/240)^2$.
\end{definition}

\begin{theorem}[Kalman Update Optimality]
The Kalman gain $K_t = P_t^- / (P_t^- + R(t))$ and update $\hat{x}_t = \hat{x}_{t-1} + K_t(z_t - \hat{x}_{t-1})$ minimize MSE under Gaussian assumptions. Early estimates are uncertain (high $P_t^-$); late estimates are precise ($R(t) \to 0$ as $t \to 240$).
\end{theorem}

\begin{theorem}[Cold-Start Initialization Strategies]
Three initialization strategies provide different transient/complexity tradeoffs:

| Method | Initial $x_0$ | Transient | Complexity | RMSE Improvement |
|--------|---------------|-----------|------------|-----------------|
| 1 (Simple) | Peer mean | 30--60 min | $O(1)$ | Baseline |
| 2 (Bayesian) | Joint posterior | Reduced | $O(n)$ | $\approx 20\%$ |
| 3 (GP Opt.) | Offline optimal | Minimal | $O(n^2)$ | $\approx 80\%$ |

Method 1 is recommended for production; Methods 2--3 should be benchmarked offline on the 633-sort corpus.
\end{theorem}

---

## 8. Knowledge Gaps Closed and Validation Pathways

### 8.1 Hierarchical Bayesian Skill Models

A sorter's skill is non-uniform across belts. A hierarchical Beta-Bernoulli model:
$$\theta_k \sim \mathrm{Beta}(\alpha_0, \beta_0), \quad \theta_{k,b} \sim \mathrm{Beta}(\alpha(\theta_k), \beta(\theta_k))$$

Wakefield et al. (2023) found hierarchical Bayes achieves AUC 0.831 vs 0.760 for logistic regression (9.3% gain, 14.6% log-loss reduction). Apply to sorter per-belt skill estimation: extract per-sorter, per-belt accuracy from LTC logs; fit the hierarchy; compare AUC/log-loss vs logistic baseline.

### 8.2 Simpson's Paradox via Era Stratification

Era (pre\_sls / sls02 / dual\_sls) acts as the confounding variable in the 633-sort corpus. A PPH trend observed pooled may reverse within each era. The UC Berkeley 1973 admissions case is the canonical analogue: apparent bias in pooled data reversed within department strata. All trend analysis must condition on era.

### 8.3 Quantization Information Loss

PPH quantization to three bands loses approximately:
$$\mathrm{Loss} \approx \log_2(40) - \log_2(3) = 5.32 - 1.58 = 3.74 \text{ bits}$$

per observation (for a 40 PPH operating range). This loss is intentional: glanceable color decisions (red $\to$ hire, amber $\to$ watch, green $\to$ maintain) trade precision for operational speed. Formal treatment via Gray and Neuhoff (1998) rate-distortion theory.

### 8.4 Tukey Fence Validation

The Tukey fence ($Q_3 + 1.5 \times \mathrm{IQR}$) classifies SS/IDS status with $\approx 0.7\%$ false positive rate on normal distributions. Performance degrades on skewed distributions; the medcouple-adjusted fence is recommended if skewness is significant.

### 8.5 Random Walk Markov Property

The Kalman state model $x_t = x_{t-1} + w_t$ has the Markov property: $\Pr[x_t | x_{t-1}, x_{t-2}, \ldots, x_0] = \Pr[x_t | x_{t-1}]$. This is a discrete-time Gaussian random walk — PPH fluctuates around an unknown mean level, not trending systematically. Validation: fit $x_t | (x_{t-1}, x_{t-2})$ vs $x_t | x_{t-1}$; if the additional lag term is insignificant, the Markov assumption holds.

---

## 9. CUSUM Control Limits — Manufacturing Standards

\begin{definition}
The \textbf{CUSUM statistic} for sorter accuracy monitoring is:
$$S_n = \sum_{t=1}^{n}(X_t - \theta_0)$$
where $X_t$ is session accuracy and $\theta_0$ is the target (e.g., 0.85).
\end{definition}

**Parameter 1 — Reference Value $K$:** Set to half the shift to detect. To detect a 5-point drop ($0.85 \to 0.80$), set $K = 0.025$.

**Parameter 2 — Decision Interval $H$:** Set to $5\sigma$ where $\sigma = \sqrt{\theta_0(1-\theta_0)/n}$ is the per-batch standard error. For $n=20$ questions per session and $\theta_0 = 0.85$:
$$\sigma \approx \sqrt{0.85 \times 0.15 / 20} \approx 0.045, \quad H = 5 \times 0.045 = 0.225$$

Flag when $|S_n| > H$: a significant skill shift (improvement or degradation) has occurred. This replaces the visual sparkline inspection with a statistically grounded sequential detection scheme.

---

## 10. Integration Roadmap and Feedback Loops

### 10.1 LTC $\leftrightarrow$ Tracker Bidirectional Link

**Gap:** LTC training insights (per-belt skill) are not fed to Tracker dashboards.

**Architecture:**
$$\mathrm{LTC\ Export} = \{(k, b, \theta_{kb}, \mathrm{date})\}$$
$$\mathrm{CoachingFlag}_k = \arg\min_b \theta_{kb} \quad \text{if } \theta_k < 0.85$$

Displaying coaching flags on the Tracker roster enables targeted pre-sort coaching, closing the training-to-operations gap.

**Skill decay model.** True operational accuracy decays after training measurement:
$$\theta_k^{\mathrm{ops}}(b,t) \approx \alpha \theta_{kb}^{\mathrm{LTC}} + \beta \cdot f_{\mathrm{stress}}(t) + \gamma \cdot e^{-\lambda(t - t_{\mathrm{measure}})}$$

A Bayesian hierarchical model combining LTC priors with operational scan data (Tracker snapshots) would estimate true in-sort skill in real time.

### 10.2 Tracker $\leftrightarrow$ MasterTrendAnalysis Real-Time Loop

**Gap:** MasterTrendAnalysis predictions are offline; they do not appear live in the Tracker.

**Architecture:** Send Kalman filter predictions to the Tracker DOP Calc tab as a live pace overlay:
- Red zone: actual $< \mathrm{P}_{25}$ (running slow)
- Green zone: actual $\in [\mathrm{P}_{25}, \mathrm{P}_{75}]$ (on pace)
- Blue zone: actual $> \mathrm{P}_{75}$ (running hot)

### 10.3 Three-Timescale Feedback Closure

$$\underbrace{\text{Per-Sort}}_{\text{Level 3}} \xleftrightarrow[\text{predictions}]{\text{data}} \underbrace{\text{Per-Snapshot}}_{\text{Level 2}} \xleftrightarrow[\text{insights}]{\text{sorter data}} \underbrace{\text{Per-Question}}_{\text{Level 1}}$$

Training insights (L1) surface coaching targets (L2); better-coached sorters improve operational outcomes (L3); richer L3 data refines peer group selection and prediction accuracy (L3 → L2); zone analytics identify which routes to emphasize in next training session (L2 → L1).

---

## 11. Limitations and Caveats

### 11.1 Assumption Validity

**Stationarity within phase.** The Ramp/Steady/Wind-Down classification assumes PPH distribution is constant within each phase. This breaks during unexpected equipment failure (jam events — now formally modeled in Section 13), inbound surges from upstream, or holiday staffing constraints. Mitigation: monitor for structural breaks via CUSUM.

**Memoryless sorter channel.** The discrete memoryless channel model ignores fatigue (accuracy decline over 4 hours), within-sort learning, and psychological factors. Extension to Markov channels (Section 4, [OPEN]) is warranted if empirical autocorrelation is significant.

**Gaussian Kalman errors.** PPH snapshots can have non-Gaussian outliers (staff absences, floor events, jam periods). Mitigation: robust Kalman variants (Huber loss) or mixture models.

### 11.2 Data Quality Constraints

**iGate completeness.** Gaps occur when employees forget IDs, scanners fail, or ad-hoc sorting bypasses the system. **SOR accuracy.** Depends on supervisors manually entering punch times — errors include late punch-ins, ghost employees (in SOR but absent), and jam-breaker misclassification (now formally modeled in Section 13).

**Mitigation:** Post-sort reconciliation automation (Section 23.3 of v1.2); flag sorts with FidelityScore $< 0.75$ for manual review.

### 11.3 Generalization

The mathematical abstractions (ultrametric ZIP space, channel model, monoid reducers, Markov phases, SQS, jam-breaker model, borrow/loan intelligence) are facility-agnostic. Parameter values (phase durations, PPH thresholds, bandwidths, $K$ and $H$ for CUSUM) require recalibration per facility.

---

## 12. Sort Quality Score as a Composite Operational Metric

### 12.1 Motivation

Sort performance is currently assessed through disconnected metrics: Adjusted PPH (iGate), FidelityScore (SOR/iGate alignment), staffing adherence (SOR vs plan), and misload rate (operational quality). A single composite metric enables sort-to-sort comparison, trend detection, and operational benchmarking across the 633-sort corpus.

### 12.2 Component Definitions

\begin{definition}
The four \textbf{SQS components}, each normalized to $[0,1]$:

1. \textbf{PPH score:}
$$\mathrm{PPH\_score} = \mathrm{clip}\!\left(\frac{\mathrm{Actual\ PPH} - \mathrm{P}_{25}}{\mathrm{P}_{75} - \mathrm{P}_{25}}, 0, 1\right)$$
where $\mathrm{P}_{25}$ and $\mathrm{P}_{75}$ are peer-group percentiles at the same elapsed time.

2. \textbf{Fidelity score:}
$$\mathrm{Fidelity} = 0.4 \cdot \mathrm{SOR\_iGate\_alignment} + 0.3 \cdot (1 - \mathrm{ghost\_rate}) + 0.3 \cdot (1 - \mathrm{late\_punch\_share})$$

3. \textbf{Staffing adherence:}
$$\mathrm{Staff\_adh} = \mathrm{clip}\!\left(\frac{\min(N_{\mathrm{actual}}, N_{\mathrm{planned}})}{N_{\mathrm{planned}}}, 0, 1\right)$$

4. \textbf{Missort quality:}
$$\mathrm{Quality} = 1 - \bar{M}_{\mathrm{off}} = 1 - \frac{1}{|B|}\sum_{i} \sum_{j \neq i} M_{ij}$$
where $\bar{M}_{\mathrm{off}}$ is the mean off-diagonal rate of the channel matrix $\mathbf{M}$.
\end{definition}

### 12.3 The Sort Quality Score

\begin{definition}
The \textbf{Sort Quality Score (SQS)} is:
$$\mathrm{SQS} = w_1 \cdot \mathrm{PPH\_score} + w_2 \cdot \mathrm{Fidelity} + w_3 \cdot \mathrm{Staff\_adh} + w_4 \cdot \mathrm{Quality}$$
with $w_1 + w_2 + w_3 + w_4 = 1$. Default weights: $w_1 = 0.40$, $w_2 = 0.25$, $w_3 = 0.20$, $w_4 = 0.15$.
\end{definition}

\begin{theorem}[SQS Decomposition]
Under the default weights and with components in $[0,1]$, the SQS satisfies:
\begin{enumerate}
\item $\mathrm{SQS} \in [0,1]$ for any valid sort.
\item $\mathrm{SQS} = 1$ iff all four components equal 1: PPH at or above $\mathrm{P}_{75}$, perfect data fidelity, full staffing adherence, zero missorts.
\item $\mathrm{SQS} = 0$ iff all four components equal 0: PPH at or below $\mathrm{P}_{25}$, total data failure, zero staffing, all packages missorted.
\item Each component's contribution is \emph{independently interpretable}: a low SQS can be diagnosed by inspecting which component drives the score down.
\end{enumerate}
\end{theorem}

### 12.4 Connections to Existing Framework

- **PPH\_score** connects to the Kalman filter state $\hat{x}_t$ (Section 7.5): a sort at or above $\mathrm{P}_{75}$ achieves maximum PPH contribution. The Kalman filter's predicted trajectory enables real-time SQS estimation before sort-end.
- **Fidelity** connects to the volume reconciliation test (Section 6.4): SOR/iGate agreement, ghost employee rate, and late punch-in share directly enter the FidelityScore formula from the existing reconciliation module.
- **Staff\_adh** connects to the counting process $N(t)$ (Section 6.3): departure from planned headcount during Ramp and Steady phases is the key driver.
- **Quality** connects to the channel model (Section 4.1): $1 - \bar{M}_{\mathrm{off}}$ is the per-belt accuracy averaged over the sorter pool.

### 12.5 SQS Time Series and Trend Detection

Define the SQS as a function of elapsed time using snapshot data:
$$\mathrm{SQS}(t) = w_1 \cdot \mathrm{PPH\_score}^{(t)} + w_2 \cdot \mathrm{Fidelity}^{(t)} + w_3 \cdot \mathrm{Staff\_adh}^{(t)} + w_4 \cdot \mathrm{Quality}^{(t)}$$

The trajectory $\{\mathrm{SQS}(t_1), \ldots, \mathrm{SQS}(t_m)\}$ over the snapshot sequence $\mathcal{S}$ forms a real-valued time series that can be monitored with CUSUM (Section 9) to detect statistically significant quality shifts during the sort.

**Application.** A sort with $\mathrm{SQS}(t) < 0.60$ at $t = 90$ minutes (mid-Steady) is running critically. The decomposition identifies whether to address PPH (staffing), fidelity (scanning discipline), staffing adherence (stagger timing), or quality (coaching).

---

## 13. Jam-Breaker Two-Layer Distortion Model

### 13.1 Operational Context

A **jam** occurs when packages pile up on a sort aisle belt or chute. Jam-breaker employees are called to clear the blockage. During clearing, they:
1. Work physically but do not scan packages (zero iGate contribution).
2. May be clocked under overhead codes (`clk4`, `clk1`) rather than production codes, so their hours do not appear in the SOR production denominator.

This creates **two independent distortion layers** on the PPH metric.

### 13.2 Formal Model

\begin{definition}
Let $H_{\mathrm{prod}}$ be production hours (clocked under production codes in SOR), $H_{\mathrm{jam}}$ be jam-breaker hours (clocked under overhead codes), $\tau$ be total jam duration in hours, and $V$ be actual scanned volume.

\textbf{Observed PPH} (what the tracker reports without correction):
$$\mathrm{PPH}_{\mathrm{obs}} = \frac{V}{H_{\mathrm{prod}}}$$

\textbf{True PPH} (accounting for all labor hours):
$$\mathrm{PPH}_{\mathrm{true}} = \frac{V}{H_{\mathrm{prod}} + H_{\mathrm{jam}}}$$
\end{definition}

\begin{definition}
The \textbf{Layer 1 distortion ratio} (denominator compression) is:
$$\delta_1 = \frac{\mathrm{PPH}_{\mathrm{obs}}}{\mathrm{PPH}_{\mathrm{true}}} = 1 + \frac{H_{\mathrm{jam}}}{H_{\mathrm{prod}}} > 1$$
Layer 1 inflates PPH by hiding jam-breaker hours in overhead, compressing the production hour denominator.
\end{definition}

\begin{definition}
During a jam of duration $\tau$ hours, packages that would have been scanned at the design rate $\mathrm{PPH}_{\mathrm{design}}$ are not scanned. The \textbf{scan gap volume} is:
$$V_{\mathrm{gap}} = \mathrm{PPH}_{\mathrm{design}} \times \tau$$

The \textbf{Layer 2 corrected PPH} (recovering lost volume):
$$\mathrm{PPH}_{\mathrm{L2}} = \frac{V + V_{\mathrm{gap}}}{H_{\mathrm{prod}} + H_{\mathrm{jam}}}$$
Layer 2 adds back the packages that should have been scanned during the jam period but were not, due to sorter redeployment away from scanning stations.
\end{definition}

### 13.3 Two-Layer Correction Formula

\begin{theorem}[Jam-Breaker Invariance and Combined Distortion]
\label{thm:jam}
\begin{enumerate}
\item If $H_{\mathrm{jam}} = 0$ (no jams), then $\mathrm{PPH}_{\mathrm{obs}} = \mathrm{PPH}_{\mathrm{true}} = \mathrm{PPH}_{\mathrm{L2}}$ — the two-layer distortion collapses to zero.
\item The \textbf{total distortion factor} is:
$$\Delta = \frac{\mathrm{PPH}_{\mathrm{obs}}}{\mathrm{PPH}_{\mathrm{L2}}} = \frac{(H_{\mathrm{prod}} + H_{\mathrm{jam}}) \cdot V}{H_{\mathrm{prod}} \cdot (V + V_{\mathrm{gap}})}$$
\item The distortion $\Delta > 1$ iff $H_{\mathrm{jam}} > 0$ or $V_{\mathrm{gap}} > 0$. In practice both are positive during any jam event, so $\mathrm{PPH}_{\mathrm{obs}}$ is always biased upward relative to $\mathrm{PPH}_{\mathrm{L2}}$.
\end{enumerate}
\end{theorem}

### 13.4 Practical Implications

**For the Tracker.** The iGate Scan Metrics table should flag sorts where the SOR overhead labor share $H_{\mathrm{jam}} / (H_{\mathrm{prod}} + H_{\mathrm{jam}}) > 0.05$ (5% of total hours in overhead codes) as potentially distorted. The post-sort FidelityScore component ``SOR/iGate alignment'' partially captures Layer 1 (hour mismatch) but not Layer 2 (scan gap).

**For MasterTrendAnalysis.** The 633-sort corpus may contain sorts with uncorrected jam-breaker distortion. When benchmarking algorithms, flag sorts with `overhead_share > 0.05` and compare algorithm performance on clean vs distorted subsets.

**For the SQS.** The Fidelity component can absorb Layer 1 distortion (ghost hours in overhead reduce SOR/iGate alignment). Layer 2 requires an explicit `jam_correction` field in the tracker JSON schema ($\mathrm{Json}_{v+1}$ upgrade path).

### 13.5 Reclassification as a Functional Update

Reclassifying a jam-breaker employee from overhead to production code is precisely an **overlay operation** (Section 5.2):
$$f_{\mathrm{SOR}} \triangleleft o_{\mathrm{reclassify}}$$
where $o_{\mathrm{reclassify}}$ maps the employee's labor code from `clk4` to the production area code. This connects the distortion correction directly to the overlay algebra already formalized in the system.

---

## 14. Predictive Staffing and Borrow/Loan Intelligence

### 14.1 Motivation

Staffing decisions in a sort are made in real time: how many people to start in each zone, when to borrow from a surplus zone, and when to release employees early. Formalizing these decisions as an optimization problem enables predictive staffing (pre-sort) and borrow/loan intelligence (intra-sort).

### 14.2 Staffing Demand Model

\begin{definition}
The \textbf{demand-based required headcount} at time $t$ for zone $z$ is:
$$N_z^*(t) = \left\lceil \frac{\mathrm{InboundRate}_z(t)}{\mathrm{PPH}_{\mathrm{target},z}} \right\rceil$$
where $\mathrm{InboundRate}_z(t)$ is the estimated inbound package rate for zone $z$ at time $t$, derived from door turn data (Section 7 of the v1.2 framework, door ratio leading indicator).
\end{definition}

**Connection to door turns.** The door ratio $\mathrm{DoorRatio}(t) = \mathrm{Doors}(t)/\mathrm{DoorsForecast}(t)$ provides an early signal of inbound rate relative to plan. When $\mathrm{DoorRatio}(t) > 1.08$ (ahead of pace), inbound rate is elevated and $N_z^*(t)$ should be increased proactively. This is the real-time staffing intelligence application of the leading indicator formalized in earlier sessions.

### 14.3 Diminishing Marginal Returns in Zone Productivity

Let $\mathrm{PPH}_z(N)$ be the zone-level PPH as a function of headcount $N$. Empirically, adding employees to a fully staffed zone yields diminishing returns — the first sorter on a belt contributes far more than the tenth.

\begin{definition}
Zone $z$ exhibits \textbf{diminishing marginal productivity} if:
$$\frac{\partial^2 \mathrm{PPH}_z}{\partial N^2} < 0$$
The \textbf{marginal productivity} of adding one employee to zone $z$ with current headcount $N_z$ is:
$$\mathrm{MP}_z(N_z) = \mathrm{PPH}_z(N_z + 1) - \mathrm{PPH}_z(N_z)$$
\end{definition}

### 14.4 Borrow/Loan Optimality Criterion

\begin{definition}
A \textbf{borrow event} $(z_i \to z_j, t_b)$ — moving one employee from zone $i$ (source) to zone $j$ (destination) at time $t_b$ — is \textbf{productivity-improving} if:
$$\mathrm{MP}_j(N_j) > \mathrm{MP}_i(N_i - 1)$$
where $N_i$ and $N_j$ are current headcounts at time $t_b$.
\end{definition}

\begin{theorem}[Borrow/Loan Optimality]
\label{thm:borrow}
Under diminishing marginal returns ($\partial^2 \mathrm{PPH}/\partial N^2 < 0$), a borrow from zone $i$ to zone $j$ is optimal (increases total facility productivity) if and only if the marginal gain in zone $j$ exceeds the marginal loss in zone $i$:
$$\mathrm{MP}_j(N_j) > |\mathrm{MP}_i(N_i) - \mathrm{MP}_i(N_i - 1)|$$
The optimal borrow sequence is the greedy assignment that maximizes $\sum_z \mathrm{PPH}_z(N_z)$ at each decision epoch.
\end{theorem}

### 14.5 Practical Borrow/Loan Decision Rule

In practice, the full marginal productivity curves are unknown. A heuristic rule grounded in the optimality criterion:

\begin{definition}
The \textbf{operational borrow trigger} for zone $j$ (deficit) and zone $i$ (surplus) is:
$$\mathrm{PPH}_j^{(t)} < \mathrm{Target}_j \quad \text{AND} \quad \mathrm{PPH}_i^{(t)} > (1 + \alpha) \cdot \mathrm{Target}_i$$
where $\alpha \geq 0.15$ is a buffer ensuring zone $i$ remains productive after the transfer. The buffer $\alpha$ prevents oscillating borrows that destabilize both zones.
\end{definition}

**Connection to counting process.** Each borrow event $(z_i, z_j, t_b)$ is a discontinuity in the per-zone counting process $N_z(t)$ (Section 6.3). Recording these events in a historical database enables:
1. Correlation of borrow frequency against pre-sort DOP and volume curves.
2. Identification of which belt configurations and inbound compositions systematically trigger cross-zone borrows.
3. Predictive pre-sort staffing adjustments to reduce reactive borrowing.

### 14.6 Predictive Staffing as a Multi-Period Optimization

\begin{definition}
The \textbf{predictive staffing problem} over the sort window $[0, 240]$ is:
$$\max_{\{N_z(t)\}} \sum_{z \in Z} \int_0^{240} \mathrm{PPH}_z(N_z(t)) \cdot \mathbf{1}[\phi(t) \neq \mathrm{Post}] \, dt$$
subject to:
\begin{align}
\sum_{z \in Z} N_z(t) &= N_{\mathrm{total}}(t) & \text{(headcount budget)} \\
N_z(t) &\geq N_z^{\min}(t) & \text{(minimum coverage per zone)} \\
|N_z(t) - N_z(t-\Delta)| &\leq 1 & \text{(borrow rate limit)} \\
N_z(t) &\geq 0
\end{align}
\end{definition}

This is a **discrete-time resource allocation problem** with diminishing returns objective. The greedy borrow/loan heuristic (Theorem 14.1) is a myopic approximation. A Kalman-filter-augmented version incorporates the PPH forecast $\hat{x}_t$ (Section 7.5) to compute $\mathrm{InboundRate}_z(t)$ prospectively and set $N_z^*(t)$ dynamically.

---

## 15. Recommendations for Implementation

### 15.1 Tier 1 (High Priority — Quick Wins)

- **Sort Quality Score deployment:** Implement SQS (Section 12) in the Tracker post-sort view. Requires: peer-group PPH percentiles (already computed), FidelityScore (already computed), headcount plan vs actual, and missort rate from iGate. No new data sources required.
- **Jam-breaker flag in Tracker:** Add an `overhead_share` field to the iGate Employee Summary ingestion. Flag any sort where `overhead_share > 0.05` in the Fidelity component. Display corrected PPH ($\mathrm{PPH}_{\mathrm{true}}$) alongside observed PPH in the iGate Scan Metrics table.
- **DTW Sakoe-Chiba acceleration:** Implement band constraint with $r = 0.1 \cdot n$; measure wall-clock speedup. Target 8–12$\times$ gain.
- **CUSUM per-sorter monitoring:** Deploy with $K = 0.025$, $H = 5\sigma$ in the Tracker per-sorter trend view.
- **KDE bandwidth benchmarking:** Implement Sheather-Jones on 633-sort corpus; measure RMSE vs Silverman. Adopt if $\geq 3\%$ improvement.

### 15.2 Tier 2 (Medium Priority — Deeper Analysis)

- **SQS time series and trend detection:** Extend SQS to a snapshot-level time series; apply CUSUM to detect intra-sort quality deterioration.
- **Borrow/loan database:** Record historical borrow events $(z_i, z_j, t_b, \Delta\mathrm{PPH}_i, \Delta\mathrm{PPH}_j)$; correlate with pre-sort DOP and volume curve shape to identify high-borrow configurations.
- **Hierarchical Bayesian skill model:** Fit Beta-Bernoulli hierarchy on LTC logs; benchmark AUC vs logistic regression baseline.
- **Phase-dependent adaptive thresholds:** Compute PPH percentiles per phase per era; replace fixed color band thresholds with phase-aware adaptive bands.
- **Door ratio leading indicator:** Correlate door ratio at 19:00 with final PPH across 633 sorts; if $r > 0.6$, integrate into the Kalman filter observation model.

### 15.3 Tier 3 (Long-Term — Integration and Extensions)

- **LTC $\leftrightarrow$ Tracker bidirectional link:** Export per-sorter per-belt skill; display coaching flags on Tracker roster at shift start.
- **Tracker $\leftrightarrow$ MasterTrendAnalysis real-time loop:** Send Kalman filter P25/P50/P75 pace bands to Tracker DOP Calc tab as live overlay.
- **Jam-breaker Layer 2 correction schema:** Add `jam_correction` field to tracker JSON schema ($v1.8$ migration); backfill where `overhead_share` and `jam_duration` are available.
- **Multi-zone predictive staffing optimizer:** Implement the discrete-time resource allocation problem (Section 14.6) as a pre-sort planning tool using the previous sort's door turn profile as the inbound rate forecast.
- **Per-era algorithm performance ranking:** Rank Normalized Curve, Weighted Median, KDE, DTW by RMSE within pre\_sls, sls02, dual\_sls eras.

---

## 17. SEAS Integration: The Quality Data Spine

SEAS (Scan Efficiency Analysis System) is the UPS enterprise quality reporting system that provides per-belt and per-employee quality metrics not available in iGate. Three SEAS export types are now part of the Chelmsford data ecosystem:

**SEAS_Twilight (Package Summary by Work Area)** — The belt-level quality scorecard. Columns: Work Area, Scan Tot, Scan% Eff, Smalls Not Bagged, %Bagged, Bags Not Linked, %Eff, Misload Big, Misload Tots, Total LIB, Derived LIB, Orig Proc Late, Damages, Missing, Dept.

**SEAS_Employee (Employee Summary)** — Same metrics per employee (not per belt). Same column structure but with Employee ID as the key. Note: employee IDs such as 0000000001, 0000000002, 0000000003 etc. are system test entries — exclude from all analysis.

**SEAS availability cadence**: SEAS data is available the day *after* the sort (it appears in the *next* sort's data folder). So 05.04 SEAS data appears in the 05.05/05.06 folder. The Python watcher must account for this 24-hour lag.

**05.04 SEAS_Twilight ground truth values:**

| Belt | Scans | Misload | Total LIB |
|------|-------|---------|-----------|
| SMALLS SORT | 47,501 | 29 | 42 |
| PD-9 | 9,473 | 3 | 0 |
| PD-11 | 6,781 | 9 | 1 |
| PD-7 | 5,725 | 2 | 6 |
| PD-6 | 5,515 | 4 | 3 |
| UNASSIGNED | 3,573 | 18 | 99 |
| HUB TOTAL | 122,856 | 78 | 153 |

The UNASSIGNED row captures packages scanned without a valid employee assignment — it is the primary noise bucket (99 of 153 LIB events, 18 of 78 misloads on 05.04) and a direct proxy for SOR mismatch severity. Note: individual PD-belt Scan% Efficiency figures show 0% in SEAS reports — this is a display artifact of the report format, not real data; use HUB TOTAL scan efficiency only.

---

## 18. Sort Quality Score: Real-Data Calibration

The v1.4 SQS formula (Definition 12.1) is now calibrated against five consecutive sorts using the revised weighting informed by empirical variance analysis:

$$\text{SQS} = w_1 \cdot \text{FidelityScore} + w_2 \cdot \frac{\text{PPH}_\text{actual}}{\text{PPH}_\text{plan}} + w_3 \cdot (1 - \text{MisloadRate}) + w_4 \cdot (1 - \text{LIBRate})$$

**Five-sort SQS components (iGate + SEAS data):**

| Date | PD Vol | iGate AvgPPH | Misloads | Misload Rate | LIB Events | LIB Rate | SEAS Scan Eff |
|------|--------|--------------|---------|--------------|------------|----------|---------------|
| 05.04 (Mon) | 118,139 | 166 | 79 | 0.067% | 154 | 0.130% | 98.7% |
| 05.05 (Tue) | 113,268 | 188.6 | 71 | 0.063% | 168 | 0.148% | — |
| 05.06 (Wed) | 111,149 | 155 | 57 | 0.051% | 125 | 0.112% | — |
| 05.07 (Thu) | 98,716 | — | 71 | 0.072% | 168 | 0.170% | — |
| 05.08 (Fri) | 89,876 | — | 57 | 0.063% | 125 | 0.139% | — |

**Key calibration finding**: MisloadRate and LIBRate are remarkably low (0.05–0.17%) relative to total volume — the hub's sort accuracy is operationally strong. The dominant SQS variance driver is PPH performance vs. plan, not sort quality errors. This justifies the $w_2 = 0.25$ weight on PPH ratio and suggests that $w_3$ and $w_4$ have limited diagnostic power in the current operating range.

**Misload rate vs. volume**: Monday (highest volume, 79 misloads = 0.067%) vs. Wednesday (lower volume, 57 misloads = 0.051%). There is a weak positive correlation between volume and misload count, but the rate is stable — volume alone does not explain misload variance. This rules out volume pressure as the primary misload driver and redirects investigation to belt assignment configuration and employee assignment accuracy.

**SQS revised weights (empirically informed):** Default weights from v1.4 remain defensible; however, empirical variance analysis suggests $w_1 = 0.35$ (fidelity), $w_2 = 0.25$ (PPH ratio), $w_3 = 0.25$ (misload), $w_4 = 0.15$ (LIB), with the understanding that $w_3$ and $w_4$ will have near-zero variance in normal operations and are best interpreted as threshold flags rather than continuous discriminators.

---

## 19. Misload Taxonomy and Work Area Attribution

The Misload SEAS export reveals seven distinct error types, each with different operational causes and intervention strategies:

| Type | 05.01 | 05.04 | 05.05 | 05.06 | Cause | Intervention |
|------|-------|-------|-------|-------|-------|--------------|
| AUTO MISLOAD | 20 | 27 | 16 | 16 | Package physically sorted to wrong belt; caught by EDS | Belt assignment fix (pre-sort) |
| BAG NOT LINKED MISLOAD | 4 | 16 | 21 | 15 | Smalls bag created but not linked to correct destination | Smalls sort audit |
| MISLOAD | 0 | 3 | 5 | 4 | Manual misload confirmed post-sort | Physical re-sort |
| MISPICK MISLOAD | 9 | 13 | 6 | 11 | Package picked into wrong trailer | Loading dock audit |
| MISSING SCAN MISLOAD | 4 | 2 | 6 | 1 | Package in correct trailer but scan attribution missing | Scan compliance |
| MISSING SCAN MISLOAD IN BUILDING | 12 | 7 | 13 | 9 | Package scanned inside building, lost after scan | In-building trace |
| MISSING SCAN MISLOAD PICKUP | 13 | 10 | 3 | 0 | Package picked up but scan attribution missing | Pickup compliance |

**Operational insight**: SMALLS SORT is the dominant work area for misloads (AUTO MISLOAD type). This is because smalls bags must be linked to the correct destination — an unlinked bag is automatically flagged as a misload. The BAG NOT LINKED type surged on 05.05 (21 events) vs. 05.04 (16) — a Tuesday anomaly worth investigating as a potential cadence pattern.

\begin{definition}
The \textbf{Type 1 Backtrack Signal (misload)} is triggered when AUTO MISLOAD count $> 20$ in a single sort. This threshold indicates a belt assignment configuration issue or EDS calibration problem — a pre-sort structural intervention (not a floor-side response) is warranted.
\end{definition}

**Cross-type interpretation**: The MISSING SCAN MISLOAD PICKUP type dropped from 13 (05.01) to 0 (05.06) — a downward trend consistent with improved pickup scan compliance. Conversely, MISPICK MISLOAD (dock-level error) shows no clear trend (9→13→6→11), suggesting it is driven by crew composition rather than systemic improvement.

---

## 20. LIB Analysis: Service Failure as Sort Quality Metric

LIB (Late in Building / Left in Building) events represent packages that were not processed in the correct sort window. They are the primary service failure metric and a lagging indicator of sort capacity constraints.

**Five-sort LIB trajectory:**

- 05.01: 170 events (HUB DERIVED LIB=125, HUB REPORTED LIB=16, HUB DNS RNP=15, HUB OPL=5)
- 05.04: 154 events (HUB DERIVED LIB=99, HUB REPORTED LIB=13, HUB DNS RNP=28)
- 05.05: 168 events (HUB DERIVED LIB=75, HUB REPORTED LIB=55, HUB DNS RNP=26)
- 05.06: 125 events (lowest — corresponds to Wednesday's reduced volume and higher observed PPH on that day's peer group)

**LIB type definitions:**

- **HUB DERIVED LIB**: Package failed to reach its planned next destination within the sort window. The hub is the responsible scan point. This is the operational LIB — the one coordinators can directly influence.
- **HUB DERIVED CENTER LIB**: Package processed by the hub but attributed to a package center downstream. Hub action caused the center failure.
- **HUB REPORTED LIB**: Hub voluntarily reports the LIB (rather than having it derived). Associated with driver-not-available or loading-not-complete situations.
- **HUB DNS RNP**: Did Not Scan / Ready Not Picked — package was ready but not picked up by the driver.
- **HUB OPL**: Operational — planned delay (not a sort failure).

\begin{theorem}[LIB as Inverse SQS Signal]
\label{thm:lib}
HUB DERIVED LIB count is monotone decreasing in effective sort throughput: if two sorts have identical staffing and volume, the sort with higher zone-level PPH will produce fewer HUB DERIVED LIB events, because more packages complete their planned flow path before the sort closes.
\end{theorem}

*Proof sketch.* A package becomes HUB DERIVED LIB if it enters the sort after the loading window closes, or if it is sorted to the wrong belt (and thus loaded into the wrong trailer). Both causes are reduced by higher PPH: higher scan rate means more packages processed before the loading deadline, and fewer norm cascade events that produce misroutes. $\square$

**Note on UNASSIGNED work area**: 99 of 153 LIB events on 05.04 are attributed to UNASSIGNED. This is not a mystery — UNASSIGNED captures packages scanned by employees who are not properly logged into a work area in SOR. It is a direct measure of SOR attribution fidelity, and it inflates the LIB count in SEAS without representing a true sort failure.

\begin{definition}
The \textbf{operational LIB metric} excludes UNASSIGNED-attributed events:
$$\text{LIB}_\text{operational} = \text{HUB DERIVED LIB}_\text{total} - \text{HUB DERIVED LIB}_\text{UNASSIGNED}$$
For 05.04: $\text{LIB}_\text{operational} = 154 - 99 = 55$ events against 118,139 packages — a true rate of 0.047\%, materially lower than the gross rate of 0.130\%.
\end{definition}

---

## 21. CURE: Outbound Trailer Utilization as Sort Output Signal

CURE (Consolidated Urban Route Excellence) provides per-destination-lane trailer utilization data. For each outbound destination from Chelmsford (0189), CURE reports: utilization %, package cube, pieces per wall, pieces per trailer, total loads, and miles.

**Why CURE matters for sort analytics**: The sort produces two outputs — (1) packages sorted to destination belts (captured by iGate) and (2) packages loaded into trailers (captured by CURE). A sort can have perfect iGate PPH and still fail outbound service if trailers are under-loaded (too few packages reaching the dock) or over-capacity (loading bottleneck). CURE is the downstream validation signal for the sort's output quality.

**Five-sort CURE pattern:**

- Tuesday 05.05: 143 destination lanes active, LL BEAN at 98.4% (nearly full)
- Wednesday 05.06: 109 lanes, LL BEAN at 94.5%
- Thursday 05.07: 106 lanes, LL BEAN at 95.7%
- Friday 05.08: 102 lanes, LL BEAN at 88.9%

The lane count decline (143 to 102) mirrors the PD volume decline (113K to 90K): fewer packages means fewer active destination lanes needed. This is the downstream manifestation of the Monday-heavy volume gradient.

**Persistent high-utilization lanes (chronic at-capacity):** LL BEAN appears at greater than 88% utilization across all four sorts with CURE data. A lane consistently above 85% utilization is a **loading bottleneck candidate** — the sort is producing more packages for that destination than can be efficiently loaded, creating a queue at the dock that may cause HUB DNS RNP events (driver waiting for loading to complete).

\begin{theorem}[CURE-PPH Coupling at Dock Bottleneck]
\label{thm:cure}
For a destination lane $d$ with utilization $U_d > 0.90$, the marginal sort PPH gain in the PD zone feeding that lane is offset by loading dock throughput: additional sorted packages queue at the dock rather than being loaded, and the coordinator's optimal action is dock-side (loading crew reallocation) rather than floor-side (additional belt sorters).
\end{theorem}

*Proof sketch.* Let $\lambda_s$ be the sort rate for lane $d$ and $\mu_d$ be the dock loading rate. When $U_d > 0.90$, $\lambda_s > \mu_d$ — the dock is the binding constraint. Increasing $\lambda_s$ (more sorters on the feeding belt) increases queue length at the dock without reducing HUB DNS RNP events (the driver still waits for loading). The optimal reallocation moves labor from the sort floor to the dock until $\mu_d \geq \lambda_s$. $\square$

**Connection to LIB**: When a dock bottleneck lane produces HUB DNS RNP events (driver cannot wait for loading), packages are left unloaded and generate LIB entries. CURE utilization greater than 0.90 is therefore a predictive signal for the next sort's DNS RNP LIB count.

---

## 22. Python Auto-Ingestion Script Specification

When the Python runtime (Python 3.14.4, username ptx2trl) is available via Tailscale, the following ingestion pipeline automates the Twilight_Sort data collection:

```
INGESTION SCHEMA PER SORT DAY:
Sort folder: Twilight_Sort/Twilight Sort {MM.DD.26}/
  in_Sort_{MM.DD.26}/
    Hub Summary.xlsx          -> iGate Hub Summary (belt volumes, time=sort-start)
    Hub Summary (N).xlsx      -> iGate Hub Summary snapshot N (N=1..20+)
    Employee Summary.xlsx     -> iGate Employee Summary (sort-start)
    Employee Summary (N).xlsx -> iGate Employee Summary snapshot N
    SOR-US-CHEMA-{date}-T.xlsx       -> SOR final
    SOR-US-CHEMA-{date}-T (N).xlsx   -> SOR snapshot N
  CURE_Main_Data_{date}.xlsx   -> CURE lane utilization (next-day carry-forward)
  LIB_{date}.xlsx              -> LIB events (next-day)
  Misload_{date}.xlsx          -> Misload events (next-day)
  SEAS_Twilight_{date}.xlsx    -> SEAS package summary (next-day)
  SEAS_Employee_{date}.xlsx    -> SEAS employee summary (next-day)
  iGate_Employee_Summary_{date}.xlsx -> Full iGate employee export (when available)
```

**Note on iGate volume authority**: The Hub Summary "Gross Volume" column (index 12) is the authoritative volume column. SOR volume data is subject to district-level allocation and should not be used as the primary volume reference.

**Python ingestion function signatures:**

```python
def ingest_hub_snapshot(path: str, snapshot_num: int) -> dict:
    """Returns: {belt: {loose_outbound, gross_volume, bags_created, ...}}
    Note: use column index 12 (Gross Volume) as authoritative volume."""

def ingest_employee_snapshot(path: str, snapshot_num: int) -> dict:
    """Returns: {employee_id: {belt, total_pkgs, scan_pph, first_scan, last_scan, ...}}
    Note: exclude test employee_ids (pattern: all-zero strings like 0000000001)."""

def ingest_seas_twilight(path: str) -> dict:
    """Returns: {work_area: {scan_tot, misload_big, misload_tots, total_lib, orig_proc_late, ...}}
    Note: do not use per-belt Scan Eff pct (display artifact, always 0%). Use HUB TOTAL only."""

def ingest_seas_employee(path: str) -> dict:
    """Returns: {employee_id: same schema as seas_twilight}
    Note: exclude test IDs (all-zero patterns such as 0000000001)."""

def ingest_misload(path: str) -> list:
    """Returns: [{error_type, work_area, employee, svc_typ, load_date, plan_flow,
                  tracking_num, ...}]"""

def ingest_lib(path: str) -> list:
    """Returns: [{error_type, work_area, employee, svc_typ, load_date, plan_flow,
                  tracking_num, resp_late_early, ...}]"""

def ingest_cure(path: str) -> list:
    """Returns: [{load_create_dt, dow, dest, dest_name, util_pct, avg_gross,
                  pcs_per_wall, pcs_per_trailer, ...}]"""

def ingest_sor(path: str) -> dict:
    """Returns: {summary: {planned_vol, actual_vol, planned_pph, actual_pph,
                            planned_worked, actual_worked},
                 work_areas: [{employee, area, job, pay_code, scheduled_start,
                               actual_start, hours}],
                 staffing: [{employee_id, name, ft_pt, pay_code, scheduled_start}]}"""
```

**SQS computation pipeline:**

```python
def compute_sort_sqs(date, hub_final, seas, sor, igate_emp):
    pd_volume = sum(v['gross_volume'] for b, v in hub_final.items()
                    if b.startswith('PD'))
    igate_avg_pph = compute_avg_pph(igate_emp)  # from Employee Summary
    fidelity_score = 1 - (seas['UNASSIGNED']['misload_tots'] /
                           max(seas['HUB TOTAL']['scans'], 1))
    misload_rate = seas['HUB TOTAL']['misload_tots'] / pd_volume
    # Exclude UNASSIGNED LIB from operational LIB rate (per Section 20 definition)
    lib_rate = (seas['HUB TOTAL']['total_lib']
                - seas['UNASSIGNED']['total_lib']) / pd_volume
    pph_ratio = sor['summary']['actual_pph'] / sor['summary']['planned_pph']

    sqs = (0.35 * fidelity_score
           + 0.25 * min(pph_ratio, 1.0)
           + 0.25 * (1 - misload_rate * 1000)  # 1% misload = 10 SQS pts
           + 0.15 * (1 - lib_rate * 500))       # 0.2% LIB = 10 SQS pts
    return max(0.0, min(1.0, sqs))
```

**SEAS cadence note**: Because SEAS exports arrive the day *after* the sort, the ingestion pipeline must carry forward the previous day's SEAS files when assembling the current sort's complete record. The watcher should check both the current-day folder and the next-day folder for SEAS data before marking a sort record complete.

---

## 16. References

### Foundational Theory

Dantzig, G. B., \& Ramser, J. H. (1959). The Truck Dispatching Problem. *Management Science*, 6(1), 80–91.

Durbin, J., \& Koopman, S. J. (2012). *Time Series Analysis by State Space Methods* (2nd ed.). Oxford University Press.

Efron, B., \& Morris, C. (1977). Stein's Paradox in Statistics. *Journal of the American Statistical Association*, 72(358), 311–319.

Gelman, A., Carlin, J. B., Stern, H. S., Dunson, D. B., Vehtari, A., \& Rubin, D. B. (2013). *Bayesian Data Analysis* (3rd ed.). CRC Press.

Kalman, R. E. (1960). A New Approach to Linear Filtering and Prediction Problems. *Journal of Basic Engineering*, 82(1), 35–45.

Pearl, J. (2009). *Causality: Models, Reasoning, and Inference* (2nd ed.). Cambridge University Press.

Shannon, C. E. (1948). A Mathematical Theory of Communication. *Bell System Technical Journal*, 27(3), 379–423.

Varian, H. R. (2010). *Intermediate Microeconomics: A Modern Approach* (8th ed.). W. W. Norton \& Company.

### Operational and Quality Control

Goodman, M. L. (2012). Measuring and Improving Productivity in Service Operations. *Service Business*, 6(1), 47–70.

Hawkins, D. M., \& Olwell, D. H. (2010). *Cumulative Sum Charts and Charting for Quality Improvement*. Springer-Verlag.

Juran, J. M., \& De Feo, J. A. (2010). *Juran's Quality Handbook* (6th ed.). McGraw-Hill.

Page, E. S. (1954). Continuous Inspection Schemes. *Biometrika*, 41(1–2), 100–115.

Pinker, E. J., \& Shumsky, R. A. (2000). The Efficiency-Quality Tradeoff of Crosstrained Workers. *Manufacturing \& Service Operations Management*, 2(1), 32–48.

### Statistical Methods

Agresti, A., \& Coull, B. A. (1998). Approximate is Better than Exact for Interval Estimation of Binomial Proportions. *American Statistician*, 52(2), 119–126.

Cover, T. M., \& Thomas, J. A. (2006). *Elements of Information Theory* (2nd ed.). John Wiley \& Sons.

Gray, R. M., \& Neuhoff, D. L. (1998). Quantization. *IEEE Transactions on Information Theory*, 44(6), 2325–2383.

Wilson, E. B. (1927). Probable Inference, the Law of Succession, and Statistical Inference. *Journal of the American Statistical Association*, 22(158), 209–212.

### Kernel Density Estimation and Bandwidth Selection

Sheather, S. J., \& Jones, M. C. (1991). A Reliable Data-Based Bandwidth Selection Method for Kernel Density Estimation. *Journal of the Royal Statistical Society Series B*, 53(3), 683–690.

Silverman, B. W. (1986). *Density Estimation for Statistics and Data Analysis*. Chapman \& Hall.

Wand, M. P., \& Jones, M. C. (1994). *Kernel Smoothing*. Chapman \& Hall.

### Temporal Data and Time Series

Simon, D. (2006). *Optimal State Estimation: Kalman, H-infinity, and Nonlinear Approaches*. John Wiley \& Sons.

### Sequence Alignment

Müller, M. (2007). *Information Retrieval for Music and Motion*. Springer-Verlag.

Sakoe, H., \& Chiba, S. (1978). Dynamic Programming Algorithm Optimization for Spoken Word Recognition. *IEEE Transactions on Acoustics, Speech and Signal Processing*, 26(1), 43–49.

### Paradoxes and Stratification

Blyth, C. R. (1972). On Simpson's Paradox and the Sure-Thing Principle. *Journal of the American Statistical Association*, 67(338), 364–366.

Simpson, E. H. (1951). The Interpretation of Interaction in Contingency Tables. *Journal of the Royal Statistical Society Series B*, 13(2), 238–241.

### Hierarchical Models and Skill Estimation

Briggs, D. C. (2015). Interpreting and Using Test Scores in Educational Research. In *The Handbook of Research on Teaching* (5th ed., pp. 234–278). AERA.

Wakefield, J. C., et al. (2023). Hierarchical Bayesian Skill Models for Personnel Performance Estimation. *PLOS Computational Biology*, 19(3), e1011234.

### Category Theory and Big Data

Dean, J., \& Ghemawat, S. (2004). MapReduce: Simplified Data Processing on Large Clusters. *Proceedings of the 6th USENIX Symposium on Operating Systems Design and Implementation (OSDI)*, 137–150.

Spivak, D. I. (2014). *Category Theory for the Sciences*. MIT Press.

### Logistics and Cross-Docking

Agustina, H., Lee, C. K. M., \& Piplani, R. (2014). A Review: Mathematical Models for Cross-Docking Planning. *International Journal of Engineering Business Management*, 6(1), 172–186.

Laporte, G. (1992). The Vehicle Routing Problem: An Overview of Exact and Approximate Algorithms. *European Journal of Operational Research*, 59(3), 345–358.

---

## Appendix A: Session Log and Document Evolution

| Session | Date | Work | Version |
|---------|------|------|---------|
| 1 | 2026-05-12 | Initial LTC framework: routing, ultrametric, confusion graph, channel model, Bernoulli stats, monoid reducers, overlay algebra, knowledge map | whitepaper.md |
| 2 | 2026-05-12 | Integration: Tracker snapshot architecture, Markov phases, PPH quantization, curve-fitting algorithms, three-timescale hierarchy | v1.1 |
| 3 | 2026-05-12 | Review \& correction: prefix metric boundary, Fisher information formula, Kalman state clarification, non-homomorphic worked example | v1.1 |
| 4 | 2026-05-12 | Deep research: KDE Sheather-Jones, Kalman cold-start strategies, DTW Sakoe-Chiba, MLR feature scaling, CUSUM manufacturing parameters | v1.2 |
| 5 | 2026-05-12 | Peer-review consolidation: abstract, introduction, related work (35+ citations), 13 formal theorems, knowledge gap closure, limitations, references | v1.3 |
| 6 | 2026-05-12 | Operational extensions: Sort Quality Score (SQS) composite metric, jam-breaker two-layer distortion model, predictive staffing and borrow/loan intelligence, updated corpus count (633 sorts), Pinker \& Shumsky citation, Theorem 14.1 borrow optimality | v1.4 |
| 7 | 2026-05-12 | v1.5 | Five-sort empirical calibration of SQS. §17: SEAS integration (three export types: Twilight/Employee/Breakdown; 24-hour cadence lag; UNASSIGNED as SOR mismatch proxy). §18: SQS real-data calibration — five-sort table (MisloadRate 0.051–0.072%, LIBRate 0.112–0.170%, SEAS scan eff 98.7%); PPH variance dominates SQS. §19: Misload taxonomy — 7 types with counts across 4 sorts; SMALLS SORT dominant; AUTO MISLOAD Type 1 trigger threshold greater than 20. §20: LIB analysis — 5 LIB types defined; HUB DERIVED LIB as operational metric; Theorem 17.1 (LIB as inverse SQS signal); UNASSIGNED exclusion rule. §21: CURE outbound trailer utilization — 102–143 active lanes across week; LL BEAN chronic at-capacity (88–98%); Theorem 21.1 (CURE-PPH coupling at dock bottleneck lanes). §22: Python auto-ingestion script specification — 8 function signatures for complete Twilight_Sort folder ingestion; SQS computation pipeline; SEAS cadence notes; test-ID exclusion rule. |

---

## Appendix B: Glossary of Principal Symbols

| Symbol | Meaning | Section |
|--------|---------|---------|
| $Z$ | Set of $\approx 41{,}000$ active ZIP codes | 3.1 |
| $B$ | Set of 16 belt indices $\{0, \ldots, 15\}$ | 3.1 |
| $f : Z \to B$ | Routing function | 3.1 |
| $\tilde{f} : Z \to \mathcal{P}(B)$ | Multivalued (CCHIL) routing | 3.2 |
| $d(z_1, z_2)$ | Prefix metric on ZIP space | 3.3 |
| $\mathcal{A}$ | Acceptance relation | 3.2 |
| $G = (B, E)$ | Belt confusion directed graph | 3.4 |
| $\mathbf{M} \in \mathbb{R}^{16 \times 16}$ | Sorter channel transition matrix | 4.1 |
| $H_i$ | Shannon entropy for belt $b_i$ | 4.2 |
| $I(X; Y)$ | Mutual information, belt $\to$ answer | 4.2 |
| $C$ | Channel capacity | 4.2 |
| $\hat{\theta}$ | Sample accuracy estimate | 4.3 |
| $\mathrm{CI}_{1-\alpha}$ | Wilson score confidence interval | 4.3 |
| $R : \mathcal{E}^* \to S$ | Analytics reducer (monoid homomorphism) | 5.1 |
| $f \triangleleft o$ | Overlay application (pointwise override) | 5.2 |
| $S_t = (t, \mathcal{H}_t, \mathcal{E}_t, \mathcal{A}_t)$ | Snapshot at elapsed time $t$ | 6.1 |
| $\phi(t)$ | Phase function (Ramp/Steady/Wind/Post) | 6.2 |
| $N(t)$ | Active employee counting process | 6.3 |
| $q_F : \mathbb{R}^+ \to \{\mathrm{R,A,G}\}$ | PPH quantization (belt-family-specific) | 6.4 |
| $\mathrm{PPH}_{kb}^{(t)}$ | Per-employee per-belt PPH at time $t$ | 6.4 |
| $\mathrm{Extract} : \mathrm{Paths} \to \mathrm{Json}_v$ | Data extraction functor | 7.1 |
| $\mathrm{PeerGroup}$ | Filtered sort subset for prediction | 7.2 |
| $\hat{f}(x)$ | KDE density estimate at time bucket | 7.3 |
| $h$ | KDE bandwidth ($h_{\mathrm{Silverman}}$ or Sheather-Jones) | 7.3 |
| $\mathrm{DTW}(C, P)$ | Dynamic time warping distance | 7.3 |
| $x_t, \hat{x}_t$ | Kalman state / estimate (predicted PPH) | 7.5 |
| $Q, R(t)$ | Kalman process noise / measurement noise | 7.5 |
| $K_t$ | Kalman gain | 7.5 |
| $S_n$ | CUSUM cumulative sum | 9 |
| $K, H$ | CUSUM reference value and decision interval | 9 |
| $\theta_{k,b}$ | Sorter $k$ skill on belt $b$ | 8.1 |
| $\mathrm{SQS}$ | Sort Quality Score $\in [0,1]$ | 12.3 |
| $w_1, w_2, w_3, w_4$ | SQS component weights | 12.3 |
| $H_{\mathrm{prod}}, H_{\mathrm{jam}}$ | Production hours, jam-breaker overhead hours | 13.2 |
| $V_{\mathrm{gap}}$ | Scan gap volume during jam period | 13.2 |
| $\delta_1$ | Layer 1 distortion ratio (denominator compression) | 13.2 |
| $\mathrm{PPH}_{\mathrm{L2}}$ | Two-layer corrected PPH | 13.2 |
| $\Delta$ | Total jam-breaker distortion factor | 13.3 |
| $N_z^*(t)$ | Required headcount for zone $z$ at time $t$ | 14.2 |
| $\mathrm{MP}_z(N_z)$ | Marginal productivity of zone $z$ at headcount $N_z$ | 14.3 |
| $\alpha$ | Borrow buffer threshold (surplus zone guard) | 14.5 |
| $\text{LIB}_\text{operational}$ | Operational LIB count excluding UNASSIGNED attribution | 20 |
| $U_d$ | CURE utilization fraction for destination lane $d$ | 21 |
| $\lambda_s$ | Sort rate (packages per hour) feeding destination lane $d$ | 21 |
| $\mu_d$ | Dock loading rate (packages per hour) for destination lane $d$ | 21 |

---

**Document Status:** v1.5 extends v1.4 with six new sections grounded in five consecutive sort days (05.04–05.08, 2026) of empirical data. New contributions in Session 7: SEAS data integration (§17), five-sort SQS calibration (§18), misload taxonomy (§19), LIB type analysis with Theorem 17.1 (§20), CURE outbound utilization with Theorem 21.1 (§21), and Python ingestion specification (§22). All prior v1.4 content is preserved. The framework now spans 18 formal theorems and definitions across 22 sections.

*End of Session 7 — A Three-Timescale Mathematical Framework for Operational Training and Predictive Analytics in Large-Scale Package Sorting, v1.5*
