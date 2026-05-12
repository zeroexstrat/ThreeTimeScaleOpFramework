# CHEMA Analytics Integration Roadmap
## From Whitepaper Theory to Operational Tool Suite

**Author:** Rafael Almeida (Employee 6068314) — UPS Chelmsford Hub  
**Document version:** v1.0  
**Date:** May 12, 2026  
**Context:** This roadmap follows the completion of v1.5 (Operational Baseline with SEAS/CURE/LIB/Misload grounding), v2.1 (Digital Twin with EKF augmentation), and v3.6 (Five-Sort Week as Empirical Functor Application). It specifies how the mathematical findings from these three branches are operationalized across the tracker, dashboard, DOP Calculator, and sort-snapshot analytics system.

---

## 1. The Integration Vision

The research program has now produced three things of operational consequence that did not exist before the 05.04–05.08 data collection week:

1. **An empirically grounded intra-sort timeline architecture** — the numbered Hub Summary and Employee Summary snapshots form a discrete time series over the sort window (18:00–22:00), not just a single end-of-night export. Every snapshot is a state vector in the EKF framework.

2. **Five new quality data streams** — SEAS (quality by belt and by employee), CURE (outbound lane utilization), LIB (service failure events), Misload (7-category scan error taxonomy), and the daily SOR with its hour-allocation context — each of which can be ingested programmatically and mapped to a model variable.

3. **Two validated structural constants** — the Zone 9–12 volume share (31.1% ± 0.9%, Theorem 19.1 in v2.1) and the weekly volume decay gradient (γ ≈ 0.938, Definition 20.1 in v2.1) — that reduce the degrees of freedom in sort-end forecasting from an open problem to a constrained prediction.

This roadmap translates these three advances into specific changes to the CHEMA tool suite: the tracker HTML, the hub ops dashboard, the DOP Calculator tab, and a new sort-snapshot analytics module.

---

## 2. Data Source Inventory and Ingestion Architecture

### 2.1 Data Sources

| Source | File Pattern | Cadence | Operational Variable |
|--------|-------------|---------|---------------------|
| iGate Hub Summary | `Hub Summary (N).xlsx` | Every ~10–12 min intra-sort | Belt-level GrossVolume, PPH proxy by belt |
| iGate Employee Summary | `Employee Summary (N).xlsx` | Every ~10–12 min intra-sort | Per-employee scans, avgPPH, hours |
| SOR | `SOR-US-CHEMA-YYYY-MM-DD-T.xlsx` | Multiple exports, latest = highest suffix `(N)` | Planned hours, actual hours, planned PPH, area/zone assignment |
| SEAS Twilight | `SEAS_Twilight_MM.DD.YY.xlsx` | Next-day (t+24h) | Belt-level misload count, misload%, LIB count, scan count |
| SEAS Employee | `SEAS_Employee_Data_MM.DD.YY.xlsx` | Next-day (t+24h) | Per-employee misload, LIB, scan totals |
| LIB | `LIB_MM.DD.YY.xlsx` | Next-day (t+24h) | LIB events by type: HUB DERIVED, CENTER, REPORTED, DNS RNP, OPL, UNASSIGNED |
| Misload | `Misload_MM.DD.YY.xlsx` | Next-day (t+24h) | 7-type misload taxonomy by employee |
| CURE | `CURE_Main_Data_MM.DD.YY.xlsx` | Next-day (reflects prior sort) | Per-lane utilization %, peak utilization, lane×sort matrix |
| oi_archive | `oi_archive_CHEMA_YYYY-MM-DD_HHMM.json` | Intra-sort (manual export) | Full snapshot: coordinator view at a point in time |
| MasterTrendAnalysis | `633_sort_history.json` | Static; grows per sort | Historical avgPPH, volume, hours, zone data across 633 sorts |

### 2.2 File Naming Convention and Snapshot Ordering

**Critical rule:** The number in the filename suffix denotes the export sequence. `Hub Summary (19).xlsx` is a later snapshot than `Hub Summary (1).xlsx`. The unnumbered `Hub Summary.xlsx` is the base/first export. Therefore the canonical snapshot ordering is:

```
Hub Summary.xlsx → Hub Summary (1).xlsx → Hub Summary (2).xlsx → ... → Hub Summary (N).xlsx
```

where `N` is the highest number present in the folder. **The highest-numbered file is always the most current state.** When loading data from a sort folder, the ingestion pipeline must always select the highest-suffix file as the authoritative end-of-sort state, while preserving the full sequence for timeline analysis.

**SOR files follow the same rule.** `SOR-US-CHEMA-2026-05-04-T (19).xlsx` supersedes all earlier SOR exports from the same sort.

### 2.3 Python Ingestion Pipeline

The eight canonical ingestion functions specified in v1.5 §22 form the backbone of the pipeline:

```python
def ingest_hub_snapshot(path: str, snapshot_num: int) -> dict
    # Returns: {belt: {gross_volume, loose_outbound, bags_linked, ...}, _meta: {snapshot_num, timestamp}}

def ingest_employee_snapshot(path: str, snapshot_num: int) -> dict
    # Returns: {employee_id: {scans, avg_pph, hours, area}, _meta: {...}}
    # Exclusion rule: skip employee IDs matching r'^0{5,}' (test IDs)

def ingest_seas_twilight(path: str) -> dict
    # Returns: {belt: {scans, misloads, misload_pct, lib_count, scan_eff}, _meta: {sort_date}}

def ingest_seas_employee(path: str) -> dict
    # Returns: {employee_id: {scans, misloads, lib_count}, _meta: {sort_date}}
    # Exclusion rule: same test ID filter

def ingest_misload(path: str) -> list[dict]
    # Returns: [{employee_id, misload_type, count, ...}, ...]
    # 7 types: AUTO MISLOAD, BAG NOT LINKED, MISLOAD, MISPICK,
    #          MISSING SCAN, MISSING SCAN IN BUILDING, MISSING SCAN PICKUP

def ingest_lib(path: str) -> list[dict]
    # Returns: [{lib_type, belt, count, ...}, ...]
    # Operational LIB = HUB DERIVED LIB (exclude UNASSIGNED for SQS computation)

def ingest_cure(path: str) -> list[dict]
    # Returns: [{lane, destination, utilization_pct, peak_pct, ...}, ...]

def ingest_sor(path: str) -> dict
    # Returns: {area: {planned_hrs, actual_hrs, planned_pph, actual_vol}, _meta: {sort_date}}
    # Use SOR hours for staffing context ONLY, not volume (iGate primacy per Proposition 13.1)
```

**Folder watcher** — the Tailscale-enabled SOR watcher (v3.5 §12.9, v3.6 §13.9) triggers ingestion when a new file appears in the sort folder. On a non-Tailscale system, the pipeline runs as a batch job at sort-end.

### 2.4 iGate Data Primacy (Proposition 13.1 in v3.6)

SOR volume data is subject to district-level hour allocation decisions that decouple it from actual floor throughput. When SOR planned volume and iGate Hub Summary GrossVolume disagree:

- **Use iGate GrossVolume as the authoritative package count.**
- **Use SOR planned/actual hours as the authoritative staffing record.**
- The ~5% schema offset between SOR Volume and iGate Hub Summary (Air + DWSSCAN + AUDITS = non-PD scans) is a known property of the schema, not an anomaly. Do not flag this offset as a data quality issue.

---

## 3. Tracker Enhancements (v1.5+)

### 3.1 Current State: Tracker v1.13 in Hub Operations Container

The tracker (currently v1.13, hosted in hub_operations_v4.x.html) already implements:
- Daily KPI grid with live PPH and staffing
- iGate Scan Metrics table (Hub tab)
- History/Trends/Cube tabs with cross-sort heatmaps
- IDB v3 persistence
- DOP Calculator with PPH scenarios

### 3.2 Area Tab — iGate Scan Metrics Parity

**Problem:** The Area tab does not currently show iGate Scan Metrics with the same conditional color formatting as the Hub tab. The Hub tab has a PPH color scale calibrated to hub-level throughput. The Area tab should show an identical table for each zone/area, with PPH conditional formatting appropriate for area-level throughput.

**Resolution:** The Area tab should render the same iGate Scan Metrics table structure as the Hub tab, pulling from the same snapshot data but filtered by area/belt assignment. The PPH conditional color bands for belt-level PPH (from v3.4 §10.9 and formalized in the PPH color bands memory) apply:

| avgPPH Range | Color |
|---|---|
| ≥ 250 | `#1a7a2e` (deep green) |
| 220–249 | `#2ecc71` (green) |
| 190–219 | `#a8d5a2` (light green) |
| 160–189 | `#f0e68c` (yellow) |
| 130–159 | `#f4a460` (amber) |
| 100–129 | `#e74c3c` (red) |
| < 100 | `#8b0000` (dark red) |

For iGate **employee-level** PPH, a higher-scale palette applies (employees can exceed 300 PPH in high-density zones). The Hub tab already uses the correct palette for employee-level data — the Area tab should match this exactly, with no deviation.

**Implementation rule:** There must be no difference between the Hub tab and Area tab iGate Scan Metrics table in terms of column structure, conditional formatting logic, and color thresholds. The only difference is the data scope (hub-wide vs. area-filtered).

### 3.3 DOP Calculator — Fix and Enhancement

**Current bug:** The DOP Calculator tab is not functioning correctly. The fix and enhancement are related — fixing the calculator while simultaneously improving its predictive basis.

**Fix:** Audit the DOP Calculator JavaScript for broken event handlers, stale DOM references, or missing data bindings. The calculator should accept: planned volume (from SOR), current snapshot volume (from latest Hub Summary), current elapsed time, planned hours, actual hours-to-date, and target sort-end time.

**Enhancement with structural constants:**

The weekly volume gradient γ ≈ 0.938 (v2.1 Definition 20.1) and Zone 9-12 structural constant 0.311 ± 0.009 (v2.1 Theorem 19.1) provide two constraints that improve the DOP forecast:

1. **Volume anchor:** If the sort date is known, the expected total volume can be estimated from the Monday baseline and the day-of-week index:
   ```
   V_expected(d) = V_Monday × γ^(d-1)
   ```
   where d=1 (Monday) through d=5 (Friday). This gives a prior on tonight's total volume that is more informative than the SOR planned volume alone (which reflects district allocation, not operational reality).

2. **Zone load distribution:** Zone 9-12 will absorb approximately 31.1% of V_total regardless of the nightly volume level. This means the DOP for zones 9-12 can be anchored to the overall hub DOP — the coordinator does not need to forecast zone load independently.

3. **PPH scenario modeling:** The DOP Calculator should offer three PPH scenarios anchored to the five-sort week empirical data:
   - Conservative: 155 PPH (Wednesday low, observed 05.07)
   - Expected: 175 PPH (week average)
   - Peak: 188.6 PPH (Tuesday high, observed 05.05)

These three scenarios replace the current single-PPH input with a probability-weighted range. The expected sort completion time is then displayed as a range (earliest/expected/latest) rather than a point estimate, consistent with the Monte Carlo forward simulation concept in v2.1 §21.

### 3.4 Zone Social Potential (Φz) Display Column

Per the tracker integration specification in v3.4 §10.9, the minimum viable implementation adds a Φz computed column to the area/zone view:

```
Φz = w_n × norm_score + w_v × volume_density + w_f × flow_indicator
     + w_s × supervisor_proximity + w_i × interaction_density + w_d × disruption_penalty
```

with prior weights: w_n=0.35, w_v=0.25, w_f=0.20, w_s=0.10, w_i=0.07, w_d=0.03.

The conditional formatting for Φz:

| Φz Range | Color | Label |
|---|---|---|
| ≥ 0.75 | Green | Flow |
| 0.60–0.74 | Yellow | Pre-flow |
| 0.45–0.59 | Amber | At risk |
| < 0.45 | Red | Intervention needed |

**ΔΦz** (change from previous snapshot) appears as a second column with directional arrow formatting (↑ green, ↓ red, → neutral).

Three diagnostic rules activate in the UI when:
- Φz < 0.45 AND avgPPH < 130 → **System design alert** (zone condition, not individual)
- ΔΦz < −0.15 in one snapshot window → **Backtrack trigger** (coordinator should review action)
- Φz ≥ 0.75 AND zone_volume_share > 0.311 + 0.02 → **Entrainment detected** (flow state — no-action policy optimal)

---

## 4. Dashboard Enhancements (v2.x)

### 4.1 Current State: Hub Ops Dashboard v2.9.x

The dashboard already shows hub-level KPIs, sort history charts, and comparative metrics. The CURE, SEAS, and LIB data streams open three new panel categories.

### 4.2 CURE Outbound Loading Panel

**Data source:** `CURE_Main_Data_MM.DD.YY.xlsx` — available next-day (reflects prior sort)

**Panel components:**
- Lane utilization heatmap: rows = destination lanes, color = utilization % using the same conditional bands as the PPH table (remapped to %)
- Critical threshold line at **92% utilization** (Theorem 16.1 in v2.1: ∂SQS/∂λ_sort < 0 above U_cap ≈ 0.92)
- Lanes at ≥ 92% shown in red with a dock-side intervention flag
- LL BEAN lane highlighted as a structural chronic overload case (88–98% across the five-sort week)
- Sort-over-sort trend for the top 5 most utilized lanes

**Operational interpretation rule:** When CURE shows U_d > 0.92 for any destination lane, the dashboard displays: *"Loading bottleneck detected. Coordinator intervention should target dock-side (trailer assignment, door priority) rather than floor-side (belt speed, staffing rebalance)."* This is Theorem 16.1's Corollary applied as a decision rule.

### 4.3 SEAS Quality Panel

**Data sources:** `SEAS_Twilight_MM.DD.YY.xlsx` and `SEAS_Employee_Data_MM.DD.YY.xlsx`

**Panel components:**

*Belt-level view:*
- Misload% by belt (bar chart, descending order)
- LIB count by belt (secondary bar)
- SMALLS SORT highlighted as structurally dominant in misload events (Proposition 17.1 in v2.1: 27.5% of LIB from 38.7% of scans)
- UNASSIGNED row explicitly excluded from the operational LIB metric and shown separately with a note: *"UNASSIGNED = packages scanned without SOR area assignment (SOR mismatch proxy, not operational LIB)"*

*Employee-level view:*
- Sortable table: employee ID, scans, misloads, misload%, LIB events
- Exclude test IDs (pattern `^0{5,}`)
- Employees with misload% > 2× zone average flagged in amber; > 3× in red

**Scan efficiency note:** The per-belt Scan% Eff column in the SEAS Twilight report shows 0% for individual PD belts — this is a display artifact of the SEAS report format. Only the HUB TOTAL row Scan% Eff (98.7% on 05.04) is meaningful. The dashboard should display only the hub-aggregate scan efficiency, not per-belt values.

### 4.4 LIB Analysis Panel

**Data source:** `LIB_MM.DD.YY.xlsx`

**Panel components:**
- LIB event count by type: HUB DERIVED LIB, HUB DERIVED CENTER LIB, HUB REPORTED LIB, HUB DNS RNP, HUB OPL, UNASSIGNED
- Operational LIB metric = HUB DERIVED LIB (excludes UNASSIGNED)
- Service Failure Rate (SFR) = HUB DERIVED LIB / V_PD × 10^4 (per 10K packages); displayed as the primary quality KPI alongside PPH
- Five-sort week SFR trend chart (from the empirical data in v2.1 §18):
  - 05.04: 1.34 / 10K
  - 05.05: 1.18 / 10K (improving)
  - 05.06: 0.98 / 10K
  - 05.07: 0.89 / 10K (LIB decay consistent with Theorem 18.1)
  - 05.08: lowest (volume-adjusted)
- Color bands: SFR < 1.0 = green; 1.0–1.5 = yellow; 1.5–2.0 = amber; > 2.0 = red

**Operational interpretation:** The LIB Decay Property (Theorem 18.1 in v2.1) establishes that HUB DERIVED LIB is monotone decreasing across the work week. A Wednesday or Thursday LIB spike is an anomaly signal (misload audit warranted). A structural floor of ~50–60 events exists even at 100K+ volume.

### 4.5 Weekly Volume Envelope Panel

**Data source:** MasterTrendAnalysis historical exports + current week iGate Hub Summary finals

**Panel components:**
- Weekly volume envelope chart: actual nightly PD volume vs. γ-curve prediction (V_Monday × γ^(d-1))
- Wednesday anomaly band: if Wednesday volume exceeds γ prediction by > 5%, flag as "Wednesday anomaly" (observed +6.5% on 05.06.26)
- Zone 9-12 volume share overlay: flat reference line at 31.1% ± 0.9%; deviations from this band indicate zone assignment changes or structural anomalies
- Day-of-week Φz prior table: display the calibrated prior for each day of the week (Monday ramp phase has lower initial Φz due to cold-start norm dynamics)

---

## 5. Sort-Snapshot Analytics — "Where Will We End Tonight"

### 5.1 The Core Concept

The numbered Hub Summary and Employee Summary snapshots create a discrete time series over the sort window. At any point during the sort, the system has:

- The sort start state (pre-sort or first snapshot)
- N completed snapshots with volume and PPH readings
- The planned volume (from SOR, corrected by iGate primacy)
- The current elapsed time and remaining time

From these, a sort-end forecast can be computed in real time.

### 5.2 The MDP Trajectory Model (oi_archive Integration)

The oi_archive JSON files capture the full coordinator view at a point in time. Three files were collected for the 05.04 week:
- `oi_archive_CHEMA_2026-05-06_2050.json` — intra-sort snapshot at 20:50
- `oi_archive_CHEMA_2026-05-06_2110.json` — intra-sort snapshot at 21:10
- `oi_archive_CHEMA_2026-05-07_2209.json` — end-of-sort export at 22:09

The oi_archive is the richest single data artifact: it contains all zone states, employee assignments, coordinator view, and current package counts simultaneously. Its structure maps directly to the MDP state vector s(t) = (V(t), E(t), c(t), λ(t), Φ(t)).

**Sort-snapshot module inputs:**
1. Latest Hub Summary snapshot → V(t), belt-level volumes
2. Latest Employee Summary snapshot → E(t), current workforce state and PPH
3. oi_archive (if available) → full coordinator view including zone assignments
4. SOR → planned hours and staffing (volume excluded per Prop 13.1)
5. Sort date → day-of-week γ prior for volume envelope

### 5.3 Sort-End Forecast Algorithm

```
Given:
  t_start = 18:00 (sort start)
  t_now = current time
  t_end = 22:00 (planned sort end; 4-hour sort window)
  V_now = GrossVolume from latest Hub Summary
  PPH_now = avgPPH from latest Employee Summary (all active employees)
  H_active = active employee-hours currently deployed (from SOR actual hours)
  V_plan = SOR planned volume (adjusted: use as lower bound, iGate as upper bound)

Step 1 — Remaining volume estimate:
  t_remaining = t_end - t_now (hours)
  V_remaining_expected = V_plan - V_now  (but V_plan is SOR; prefer gamma estimate)
  V_gamma = V_Monday × γ^(day_index)  (from structural constant)
  V_remaining = max(V_gamma, V_plan) - V_now  (conservative)

Step 2 — PPH scenarios:
  PPH_conservative = 155  (Wednesday empirical low)
  PPH_expected = PPH_now  (current observed rate)
  PPH_peak = 188.6  (Tuesday empirical high)

Step 3 — Sort-end volume forecast:
  V_sortend_conservative = V_now + PPH_conservative × H_active × t_remaining
  V_sortend_expected = V_now + PPH_expected × H_active × t_remaining
  V_sortend_peak = V_now + PPH_peak × H_active × t_remaining

Step 4 — Sort Quality Score (SQS) projection:
  fidelity_score = 1 - (UNASSIGNED_misloads / max(hub_total_scans, 1))
  misload_rate = hub_total_misloads / V_now  [from SEAS if available; else use weekly avg]
  lib_rate = HUB_DERIVED_LIB / V_now  [from LIB if available; else use SFR prior]
  pph_ratio = PPH_now / PPH_plan
  SQS = 0.35 × fidelity_score + 0.25 × min(pph_ratio, 1.0)
        + 0.25 × (1 - misload_rate × 1000) + 0.15 × (1 - lib_rate × 500)
  SQS = max(0, min(1, SQS))

Step 5 — Zone 9-12 sub-forecast:
  V_z912_expected = V_sortend_expected × 0.311  (structural constant)
  V_z912_range = [V_sortend_conservative × 0.302, V_sortend_peak × 0.320]
```

### 5.4 Sort-Snapshot UI Components

The sort-snapshot module appears as a new tab in the tracker (or as a live panel in the Hub tab sidebar) with:

**Top KPI row:**
- Current volume (V_now) vs. planned (V_plan)
- Current avgPPH vs. week average (175) vs. peak (188.6)
- Elapsed time / remaining time progress bar
- SQS score with color band (≥ 0.85 = green, 0.70–0.84 = yellow, < 0.70 = red)

**Sort-end forecast band:**
- Three-line chart or range display: conservative / expected / peak sort-end volume
- Zone 9-12 sub-forecast below
- Estimated sort completion time (when does V_now reach V_gamma?)

**Intra-sort timeline:**
- Scrollable timeline of all snapshots taken during the sort (Hub Summary N index)
- Each snapshot shown as a point: V(t), PPH at that snapshot, ΔV from previous
- HS(8) backtrack event marker (where the coordinator made a Type 1 decision)
- Entrainment window highlight: snapshots where Φz ≥ 0.75 AND PPH ≥ 175

**Active alert panel:**
- CURE dock utilization alerts (if CURE data available from prior sort)
- Zone Φz warnings
- Backtrack trigger log

### 5.5 Data Freshness Rules

Because SEAS, LIB, Misload, and CURE are available only next-day, the sort-snapshot module must operate in two modes:

**Live mode (intra-sort):** Uses only iGate Hub Summary, Employee Summary, SOR, and oi_archive. SEAS/LIB/Misload/CURE are unavailable. SQS uses prior-week averages for the quality components (misload_rate, lib_rate).

**Post-sort mode (next day):** Full SQS computation becomes available once SEAS, LIB, Misload are ingested. The tracker updates the previous sort's SQS retroactively and stores it in IDB for the History tab trend line.

The MasterTrendAnalysis module holds the sort-level SQS time series as each sort's data becomes complete. The DOP Calculator reads the last 30 sort SQS values to calibrate the PPH scenario weights.

---

## 6. Python–Tailscale Integration Pipeline

### 6.1 Phase Architecture (from v3.6 §13.9)

The four-phase Tailscale deployment brings the ingestion pipeline online during live operations:

**Phase 1 — SOR Watcher (available now; Tailscale ready)**

```python
# hub_watch.py — deploys on work laptop (Windows, Python 3.14)
import watchdog, openpyxl, requests, time, os

SORT_FOLDER = r"C:\Users\ptx2trl\...\Twilight_Sort\{today}"
TAILSCALE_ENDPOINT = "http://100.x.x.x:5000/ingest"  # companion server

def on_new_sor_file(path):
    data = ingest_sor(path)
    # Use SOR hours only, not volume
    payload = {"type": "sor", "data": data, "snapshot_num": extract_suffix(path)}
    requests.post(TAILSCALE_ENDPOINT, json=payload)

def on_new_hub_summary(path):
    n = extract_suffix(path)
    data = ingest_hub_snapshot(path, n)
    payload = {"type": "hub_summary", "snapshot_num": n, "data": data}
    requests.post(TAILSCALE_ENDPOINT, json=payload)

def on_new_employee_summary(path):
    n = extract_suffix(path)
    data = ingest_employee_snapshot(path, n)
    payload = {"type": "employee_summary", "snapshot_num": n, "data": data}
    requests.post(TAILSCALE_ENDPOINT, json=payload)
```

**Phase 2 — iGate Ingestion and Companion Server**

The companion.html (already in the Twilight 050426 folder) receives POST payloads from the watcher and updates its internal state. On receiving a new Hub Summary snapshot, it:
1. Updates the sort-end forecast
2. Recomputes Φz for each zone
3. Checks CURE dock utilization (from prior sort's CURE file, loaded at sort start)
4. Pushes updated state to any open tracker tabs via the Tailscale LAN

**Phase 3 — Coordinator Intervention Logger (D_op Builder)**

Every coordinator action (staffing move, belt speed change, borrow/loan request) is logged as a timestamped MDP action record. This builds the D_op dataset required for the expert iteration loop specified in v3.3.

```python
class InterventionLogger:
    def log_action(self, action_type: str, zone: str, magnitude: float,
                   snapshot_before: int, snapshot_after: int):
        # action_type: "staff_increase", "staff_decrease", "belt_speed_change",
        #              "borrow_request", "loan_request", "break_schedule"
        record = {
            "timestamp": datetime.now().isoformat(),
            "action_type": action_type,
            "zone": zone,
            "magnitude": magnitude,
            "pph_before": self.get_pph(zone, snapshot_before),
            "pph_after": self.get_pph(zone, snapshot_after),
            "phi_before": self.get_phi(zone, snapshot_before),
            "delta_phi": self.get_phi(zone, snapshot_after) - self.get_phi(zone, snapshot_before)
        }
        self.d_op.append(record)
```

**Phase 4 — MasterTrendAnalysis API and Prior Updater**

The MTA module (633-sort historical dataset in `MasterTrendAnalysis/`) serves as the calibration ground for the EKF priors. After each sort, the completed sort's data is appended to the MTA JSON and the following priors are updated:

- Day-of-week PPH priors (by day index)
- Zone-level volume share priors (Z9-12 structural constant validation)
- Φz prior weights (OLS recalibration from IDB sort history)
- Weekly γ decay parameter (rolling 8-week geometric mean)

This creates the weekly expert iteration loop: each completed sort improves the prior for the following sort.

### 6.2 Batch Mode (Pre-Tailscale Fallback)

Before Tailscale is fully operational in the sort environment, all ingestion runs in batch mode at sort-end:

```python
# run_sort_ingestion.py
import os, glob

def ingest_sort_folder(folder_path: str, sort_date: str) -> dict:
    # Discover all files
    hub_files = sorted(glob.glob(f"{folder_path}/Hub Summary*.xlsx"),
                       key=lambda x: extract_suffix(x))
    emp_files = sorted(glob.glob(f"{folder_path}/Employee Summary*.xlsx"),
                       key=lambda x: extract_suffix(x))
    sor_files = sorted(glob.glob(f"{folder_path}/SOR-US-CHEMA-*.xlsx"),
                       key=lambda x: extract_suffix(x))

    # Authoritative end-of-sort state = highest-suffix file
    hub_final = ingest_hub_snapshot(hub_files[-1], len(hub_files)-1)
    emp_final = ingest_employee_snapshot(emp_files[-1], len(emp_files)-1)
    sor_final = ingest_sor(sor_files[-1])

    # Full timeline for snapshot analytics
    hub_timeline = [ingest_hub_snapshot(f, i) for i, f in enumerate(hub_files)]
    emp_timeline = [ingest_employee_snapshot(f, i) for i, f in enumerate(emp_files)]

    # SEAS / LIB / Misload / CURE from next-day folder (if available)
    next_day_folder = find_next_day_folder(sort_date)
    seas = ingest_seas_twilight(find_seas_file(next_day_folder, sort_date)) if next_day_folder else None
    lib = ingest_lib(find_lib_file(next_day_folder, sort_date)) if next_day_folder else None
    misload = ingest_misload(find_misload_file(next_day_folder, sort_date)) if next_day_folder else None
    cure = ingest_cure(find_cure_file(next_day_folder, sort_date)) if next_day_folder else None

    # Compute SQS when all quality data available
    sqs = compute_sort_sqs(sort_date, hub_final, seas, sor_final, emp_final) if seas else None

    return {
        "sort_date": sort_date,
        "hub_final": hub_final,
        "emp_final": emp_final,
        "sor": sor_final,
        "hub_timeline": hub_timeline,
        "emp_timeline": emp_timeline,
        "seas": seas,
        "lib": lib,
        "misload": misload,
        "cure": cure,
        "sqs": sqs
    }
```

---

## 7. Implementation Priority and Sequencing

### Tier 1 — Fix and Align (No New Data Required)

These changes use only data already present in the tracker and require no new ingestion pipeline:

1. **Fix DOP Calculator tab** — audit JavaScript event handlers and DOM bindings; restore calculator functionality
2. **Area tab iGate Scan Metrics parity** — replicate Hub tab PPH table with identical conditional formatting into the Area tab; confirm no difference in column structure, color bands, or interaction behavior
3. **Add weekly γ and Zone 9-12 constant to DOP Calculator** — add day-of-week volume anchor and zone 9-12 structural prior as input parameters (hard-coded from empirical values pending dynamic ingestion)

### Tier 2 — Batch Ingestion (Sort-End Data Load)

These changes require the Python batch ingestion pipeline but not real-time connectivity:

4. **Sort folder drag-and-drop ingestion** — user selects the sort folder in the tracker UI; the tracker calls a local Python script (or web worker) to ingest all files and populate the IDB
5. **Post-sort SQS computation** — after SEAS/LIB/Misload are available next day, recompute and store SQS in IDB; display in History tab trend line
6. **CURE dock utilization panel** — load prior-sort CURE file at sort start; display lane utilization heatmap with 92% threshold alert
7. **SEAS quality panel** — load next-day SEAS data; display belt-level misload%, LIB count, and employee-level quality table

### Tier 3 — Live Sort-Snapshot Analytics (Tailscale Phase 1-2)

These changes require the Tailscale watcher to be running during the sort:

8. **Intra-sort snapshot timeline** — Hub Summary and Employee Summary are ingested as they appear; tracker updates sort-end forecast and PPH trend in real time
9. **Sort-end forecast band** — three-scenario (conservative/expected/peak) sort-end volume range displayed live; updates every time a new Hub Summary is ingested
10. **Φz computed column** — Φz and ΔΦz displayed for each zone; three diagnostic rules active

### Tier 4 — ML Pipeline and Expert Iteration (Tailscale Phase 3-4)

These changes require accumulated D_op data and are the v1.6+ full ML pipeline from v3.4 §10.9:

11. **Coordinator intervention logger** — log every staffing and belt action as an MDP action record; build D_op over successive sorts
12. **XGBoost-as-JSON recommendation panel** — trained on D_op; top-3 coordinator interventions ranked by predicted ΔΦz; weekly retraining cycle
13. **MTA prior updater** — automated post-sort update to MasterTrendAnalysis JSON; EKF priors refreshed after each sort

---

## 8. Data Flow Summary

```
Sort Window (18:00–22:00)
│
├── iGate Hub Summary (N snapshots) ──────────────────→ Sort-end forecast
│       ↓ highest-suffix = authoritative                 Zone 9-12 sub-forecast
│       ↓ timeline = intra-sort analytics                DOP Calculator input
│
├── iGate Employee Summary (N snapshots) ─────────────→ PPH trend
│       ↓ highest-suffix = authoritative                 Employee-level quality
│       ↓ exclude test IDs                               Φz computation
│
├── SOR (highest suffix per iGate primacy rule) ───────→ Hours and staffing ONLY
│       ↓ volume = context, not ground truth             DOP planned hours input
│
├── oi_archive (manual export) ────────────────────────→ Full MDP state vector
│       ↓ coordinator view at a point in time            Type 1 backtrack detection
│
Next Day (t+24h)
│
├── SEAS Twilight ─────────────────────────────────────→ Belt-level quality panel
│       ↓ Hub TOTAL scan eff = valid; per-belt = artifact SQS quality component
│
├── SEAS Employee ─────────────────────────────────────→ Employee quality table
│       ↓ exclude test IDs
│
├── LIB ───────────────────────────────────────────────→ SFR computation
│       ↓ HUB DERIVED LIB = operational metric          LIB decay monitoring
│       ↓ UNASSIGNED = SOR mismatch proxy (exclude)
│
├── Misload ────────────────────────────────────────────→ 7-type taxonomy table
│       ↓ AUTO MISLOAD > 20 = pre-sort structural alert
│
├── CURE ───────────────────────────────────────────────→ Dock utilization panel
│       ↓ U_d > 0.92 = dock intervention (not floor)    Loading bottleneck flag
│       ↓ note: filename date = prior sort date
│
└── All above → SQS retroactive computation → MasterTrendAnalysis → EKF prior update
```

---

## 9. Metrics That Don't Exist Yet (Open Instrumentation Gaps)

These are metrics the mathematical framework calls for but that cannot yet be computed from available data:

| Metric | Framework Symbol | Blocking Gap |
|--------|-----------------|--------------|
| Belt speed | λ_belt(t) | No telemetry access; coordinator must report manually |
| Feeder GPS ETA | τ_f (Theorem 3.1, v2.0) | UPS telematics API access required |
| Zone norm score | n_z (Φz component) | Requires employee-level scan location data, not available in current iGate |
| Fatigue curve | F_j(t) | No wearable/duration tracking; estimated from hours-worked proxy |
| Supervisor proximity | s_z (Φz component) | No location tracking; coordinator self-report or IDB entry |
| Coordinator action log | a(t) ∈ A | Not yet logged; Phase 3 of Tailscale deployment |
| Pre-sort ORION manifest | Q_pre (v1.5 future) | Requires ORION API integration; district approval |

The Φz computation in Tier 3 uses available observables as proxies for the unavailable variables, per the v3.4 §10.8 prior weights. The model degrades gracefully: the prior-weighted estimate is still better than no estimate, and it calibrates toward empirical OLS weights as sort history accumulates.

---

## 10. Document Relationships

This roadmap is the operational bridge document connecting the three whitepaper branches to the CHEMA tool suite:

| This section | Whitepaper source |
|---|---|
| §2.3 Python Ingestion Pipeline | v1.5 §22 |
| §2.4 iGate Data Primacy | v3.6 Proposition 13.1 |
| §3.2 Area Tab PPH Parity | v3.6 §13.9 (tracker parity requirement) |
| §3.3 DOP Calculator with γ and Z9-12 | v2.1 Definitions 20.1 and Theorem 19.1 |
| §3.4 Φz Display Column | v3.4 §10.9 (minimum viable implementation) |
| §4.2 CURE Panel | v2.1 §16, Theorem 16.1 |
| §4.3 SEAS Quality Panel | v2.1 §17, Proposition 17.1 |
| §4.4 LIB Panel | v2.1 §18, Theorem 18.1, Definition 18.1 |
| §4.5 Weekly Volume Envelope | v2.1 §20, Theorem 19.1, Definition 20.1 |
| §5.3 Sort-End Forecast Algorithm | v2.1 §21, v1.5 §22 (SQS pipeline) |
| §6 Tailscale Pipeline | v3.6 §13.9 (confirmed Phase 1–4 spec) |
| §7 Implementation Tiers | v3.4 §10.9 (Tier 1 = v1.5 MVB; Tier 4 = v1.6+) |
| §9 Instrumentation Gaps | v3.5 §12.9 (open questions); v2.0 §15 (migration path) |

---

*Rafael Almeida, Employee 6068314*  
*Hub Operations Coordinator — UPS Chelmsford Hub, Chelmsford MA*  
<!-- tracker_author: Rafael Almeida | employee_id: 6068314 | doc_type: integration_roadmap | version: 1.0 -->
