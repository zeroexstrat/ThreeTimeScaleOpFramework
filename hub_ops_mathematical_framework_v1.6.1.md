---
title: "A Three-Timescale Mathematical Framework for Operational Training and Predictive Analytics in Large-Scale Package Sorting — v1.6.1 ε_schema(d) Weekly Fit"
author: "Rafael Almeida (Employee 6068314) — UPS Chelmsford Hub"
date: "May 2026"
geometry: margin=1in
fontsize: 11pt
---

<!--
  hub_ops_mathematical_framework_v1.6.1.md
  Author: Rafael Almeida, Employee 6068314
  const AUTHOR_SIGNATURE = "Rafael Almeida, Employee 6068314";
-->
<meta name="author" content="Rafael Almeida 6068314" />

**Version:** 1.6.1 — One increment toward v1.7. Defines the operational algorithm for the ε_schema(d) weekly fit and specifies how it is exposed in the tracker's Volume Reconciliation banner.
**Reads with:** `hub_ops_mathematical_framework_v1.6.md` (§§23–25) and the tracker source `tracker_v2.10.html` (`renderVolumeReconciliation` around L1793).
**Scope:** Theory + production-ready implementation specification. The reference Python and JavaScript snippets below are intended to drop in.

---

## 27. The Problem v1.6.1 Solves

v1.6 §23 demoted ε_schema from a structural constant to a function ε_schema(d) with the empirical relation $|\varepsilon_\text{schema}(d)| \approx 0.050 \cdot \exp(0.275 \cdot (d - \text{Mon}))$ fit to three Monday/Thursday/Friday observations.

That formula is only as good as the three points it sits on. The tracker's Volume Reconciliation banner currently uses a hardcoded 5% threshold to classify the variance between SOR Volume actual and Hub Net Outbound as "IN AGREEMENT," "MINOR VARIANCE," or "INVESTIGATE." Under v1.6 §23, that threshold is right for Monday but is exceeded *by mechanism* on Thursday and Friday — so the banner flags weeks 4 and 5 as anomalous when in fact they are routine.

The fix is to make the banner threshold day-of-week-aware, sourced from a rolling empirical fit that updates each week and is auditable from the tracker UI itself. This document specifies that algorithm and its UI surface.

## 28. The Weekly-Fit Algorithm

### 28.1 Definition

\begin{definition}[Weekly empirical $\varepsilon_\text{schema}$ fit]
\label{def:weekly-eps}
Let $\mathcal{H} = \{(d_i, \varepsilon_i)\}_{i=1}^{N}$ be the history of observed day-of-week / schema-offset pairs accumulated by the tracker over the most recent $W$ work-weeks (default $W = 4$). For day-of-week $d \in \{\text{Mon,Tue,Wed,Thu,Fri}\}$, define the fit
\[
\hat{\varepsilon}_\text{schema}(d) = \frac{1}{|\mathcal{H}_d|} \sum_{(d', \varepsilon') \in \mathcal{H}_d} \varepsilon', \qquad
\hat{\sigma}_\text{schema}(d) = \sqrt{\frac{1}{|\mathcal{H}_d|-1} \sum_{(d', \varepsilon') \in \mathcal{H}_d} (\varepsilon' - \hat{\varepsilon}_\text{schema}(d))^2}
\]
where $\mathcal{H}_d = \{(d', \varepsilon') \in \mathcal{H} : d' = d\}$.

When $|\mathcal{H}_d| < 2$ (insufficient history for a within-day standard error), fall back to the exponential prior from v1.6 §23:
\[
\hat{\varepsilon}_\text{schema}^{\text{prior}}(d) = -0.050 \cdot \exp(0.275 \cdot (d - \text{Mon})), \qquad
\hat{\sigma}_\text{schema}^{\text{prior}}(d) = 0.020.
\]
\end{definition}

### 28.2 Banner threshold

\begin{definition}[Reconciliation banner threshold]
\label{def:banner-threshold}
For the sort on day $d$ with observed Hub Net Outbound $V_H$ and SOR Volume actual $V_S$, define the day-aware reconciliation variance
\[
\nu(d) = \frac{V_S - V_H}{V_S} - \hat{\varepsilon}_\text{schema}(d),
\]
i.e., the *residual* after subtracting the expected schema offset. The banner status is set by
\[
\text{status} = \begin{cases}
\text{IN AGREEMENT} & |\nu(d)| \le \hat{\sigma}_\text{schema}(d) + 0.01 \\
\text{MINOR VARIANCE} & \hat{\sigma}_\text{schema}(d) + 0.01 < |\nu(d)| \le 2 \hat{\sigma}_\text{schema}(d) + 0.03 \\
\text{INVESTIGATE} & |\nu(d)| > 2 \hat{\sigma}_\text{schema}(d) + 0.03.
\end{cases}
\]
\end{definition}

The 1% and 3% terms are explicit operator-tolerance buffers — they make the threshold robust to rounding, mid-sort timing differences, and small Air Recovery volume swings that do not warrant investigation.

### 28.3 Sensitivity of the empirical week to the fit

Applying Definition 28.1 to the only fully-observed days in the 05.04–05.08 week (Mon/Thu/Fri — Tue/Wed had no SOR Volume actual):

| Day | $|\varepsilon_\text{schema}|$ observed | Fit (prior, single obs.) | Banner $|\nu(d)| \le$ for IN AGREEMENT |
|-----|---------------------------------------:|-------------------------:|---------------------------------------:|
| Mon | 5.05% | 5.0% | 3.0% |
| Thu |  9.88% | 9.5% | 3.0% |
| Fri | 15.13% | 14.6% | 3.0% |

A Friday SOR Volume that reads as Hub Net Outbound × (1 + 0.0505) — the *Monday* schema offset — would *correctly* trigger MINOR VARIANCE under v1.6.1 (the residual is 0.151 − 0.050 = 10.1pp, well outside the 3% IN AGREEMENT band), while under the current 5% hard threshold the same reading would already trigger INVESTIGATE — which is correct on Monday but wrong on Friday. The day-aware threshold prevents that confusion.

## 29. Implementation in `renderVolumeReconciliation`

### 29.1 Storage

Persist the history $\mathcal{H}$ in IndexedDB alongside the existing sort-day snapshots store. One row per (sortDate, $\varepsilon$) pair. The history is bounded to the most recent four work-weeks (20 rows max under the standard schedule).

```javascript
// IDB schema additive — add object store `eps_schema_history`, keyPath `sortDate`.
// row = { sortDate: 'YYYY-MM-DD', dow: 'Mon|Tue|Wed|Thu|Fri',
//         epsilon: number /* signed; e.g. -0.0505 */,
//         hubNetOutbound: number, sorVolumeActual: number, ts: ISO8601 }
```

### 29.2 Capture (called from `buildSortDaySnapshot` after a sort is finalized)

```javascript
function _captureEpsilonObs(snap){
  const v_h = snap.netVol;
  const v_s = (DATA.sor && DATA.sor.summary && DATA.sor.summary.actual && DATA.sor.summary.actual.Volume) || null;
  if (!v_h || !v_s || v_s === 0) return null;   // skip days with no SOR Volume actual
  const eps = (v_h / v_s) - 1;                  // signed: PDsum/SOR-1
  const dow = new Date(snap.sortDate + 'T00:00:00').toLocaleString('en-US', {weekday:'short'});
  const obs = { sortDate: snap.sortDate, dow, epsilon: eps,
                hubNetOutbound: v_h, sorVolumeActual: v_s, ts: new Date().toISOString() };
  // Persist into IDB store `eps_schema_history` and trim to W=4 weeks.
  putIDB('eps_schema_history', obs);
  trimEpsilonHistory(20);
  return obs;
}
```

### 29.3 Fit (called from `renderVolumeReconciliation`)

```javascript
const PRIOR_EPS = (dow) => {
  const idx = ['Mon','Tue','Wed','Thu','Fri'].indexOf(dow);
  return idx < 0 ? null : -0.050 * Math.exp(0.275 * idx);
};

function _epsilonFitForToday(dow){
  const hist = getIDBAll('eps_schema_history');
  const sameDow = hist.filter(r => r.dow === dow).map(r => r.epsilon);
  if (sameDow.length >= 2) {
    const mean = sameDow.reduce((a,b)=>a+b,0) / sameDow.length;
    const variance = sameDow.reduce((s,e)=>s+(e-mean)**2, 0) / (sameDow.length - 1);
    return { fit: mean, sigma: Math.sqrt(variance), source: 'empirical', n: sameDow.length };
  }
  // Fall back to v1.6 §23 exponential prior
  const p = PRIOR_EPS(dow);
  return p == null ? null : { fit: p, sigma: 0.020, source: 'prior', n: 0 };
}
```

### 29.4 Banner — revised classifier

Replace the v2.9 hardcoded thresholds (`if (pct <= 1) ... else if (pct <= 5)`) with:

```javascript
const dow = sortDow();                                // 'Mon' | 'Tue' | ...
const { fit, sigma, source, n } = _epsilonFitForToday(dow) || {};
const eps_obs   = sorVolActual > 0 ? (hubAllOutbound / sorVolActual - 1) : null;
const residual  = eps_obs != null && fit != null ? eps_obs - fit : null;
const absResid  = residual != null ? Math.abs(residual) : null;
const tol_in    = (sigma ?? 0.020) + 0.010;
const tol_warn  = 2 * (sigma ?? 0.020) + 0.030;
let status, statusLbl;
if (absResid == null)                  { status = 'warn'; statusLbl = 'AWAITING DATA'; }
else if (absResid <= tol_in)           { status = 'ok';   statusLbl = 'IN AGREEMENT'; }
else if (absResid <= tol_warn)         { status = 'warn'; statusLbl = 'MINOR VARIANCE'; }
else                                   { status = 'bad';  statusLbl = 'INVESTIGATE'; }

const statusMsg = `SOR Actual <strong>${fInt(sorVolActual)}</strong> vs Hub Net Outbound <strong>${fInt(hubAllOutbound)}</strong>: `
  + `observed ε=${(eps_obs*100).toFixed(2)}%, ${source} fit ε̂(${dow})=${(fit*100).toFixed(2)}% ± ${(sigma*100).toFixed(2)}% `
  + `(n=${n}). Residual ${(residual*100).toFixed(2)}%.`;
```

The banner now reports observed ε, the day-of-week fit, the fit's source and sample size, and the residual relative to expectations — every piece of information a coordinator or DM needs to decide whether to investigate.

### 29.5 UI affordance

Add a small "fit history" toggle inside the banner that, when expanded, shows the last 20 observations in a sparkline grouped by day-of-week, with the Monday/Thursday/Friday columns highlighted. This makes the fit auditable from the same UI element.

## 30. Validation Hook (deferred to v1.7)

The fit's correctness is testable: after $\ge 4$ weeks of capture, evaluate $\hat{\varepsilon}_\text{schema}(d)$ on held-out days and compare against the v1.6 §23 exponential prior. If $\hat{\varepsilon}_\text{schema}(d)$ deviates from the prior by more than $\hat{\sigma}_\text{schema}(d)$ for any $d$, the prior should be re-fit using the new history — at which point we are ready for v1.7 §22 (weekly $\gamma$ posterior + ε_schema(d) re-fit on a fixed cadence).

---

## Appendix v1.6.1.A — Session 18 Entry

**Session 18 — 2026-05-12 — v1.6.1 (Branch A).** One increment toward v1.7. Formalized the rolling-empirical-fit algorithm for ε_schema(d) (Def 28.1) with fallback to the v1.6 §23 exponential prior when within-day history is insufficient. Defined the day-aware banner classifier (Def 28.2) with explicit operator-tolerance buffers. Specified the IDB schema, capture hook, fit routine, and revised classifier as drop-in code for tracker `renderVolumeReconciliation`. Validation hook deferred to v1.7 once $\ge 4$ weeks of capture exist.

## Appendix v1.6.1.B — Added Symbols

| Symbol | Definition | First appearance |
|--------|------------|------------------|
| $\hat{\varepsilon}_\text{schema}(d)$ | Empirical day-of-week fit of $\varepsilon_\text{schema}$ | Def 28.1 |
| $\hat{\sigma}_\text{schema}(d)$ | Sample standard deviation of $\varepsilon_\text{schema}$ on day $d$ | Def 28.1 |
| $\nu(d)$ | Banner residual after subtracting expected schema offset | Def 28.2 |
| $\mathcal{H}, \mathcal{H}_d$ | Rolling history of $\varepsilon$ observations; subset for day $d$ | Def 28.1 |
| $W$ | Rolling-fit window in work-weeks (default 4) | §29.1 |

*End of Session 18 — v1.6.1, Branch A — ε_schema(d) Weekly Fit and Banner Exposure.*
