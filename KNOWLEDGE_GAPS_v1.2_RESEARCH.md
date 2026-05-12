# Knowledge Gaps Identified for v1.2 Research

**Gaps to investigate & fill:**

## Data Science / ML Algorithms

1. **KDE Silverman Bandwidth (§14.2)** — Verify formula $h = 1.06 \sigma n^{-0.2}$ is current best practice, or if updated variants (Sheather-Jones, etc.) are now standard

2. **DTW Distance Normalization (§14.2)** — Verify we're using the standard Euclidean formulation; check if Sakoe-Chiba band constraint is relevant for our use case

3. **Kalman Filter Measurement Noise (§14.4)** — Research: is $R(t) \propto (1 - t/T)^2$ the right functional form, or should we use other decay curves (exponential, linear)?

4. **Kalman Filter Initialization (§14.4)** — Research: cold-start initialization strategies for Kalman filters when no prior history exists

5. **MLR Feature Interaction Terms (§14.3)** — Check best practices for interaction terms in OLS when features are on different scales (DOW is categorical, vol/head is continuous)

## Operational Statistics

6. **Simpson's Paradox in Sort Data (§18.2)** — Does stratification by era actually reverse any major trend directions in real 620-sort corpus?

7. **IQR-Based Classification (§14.1)** — Verify Tukey fence method ($Q_3 + 1.5 \times IQR$) is being applied correctly for SS/IDS status flags

8. **CUSUM Control Charts (§6.4)** — Research: what are industry-standard control limits for manufacturing/logistics operations similar to package sorting?

9. **Phase-Dependent Distributions (§11.2)** — Quantify: what % of Ramp-phase low PPH alarms are "expected" vs "concerning"?

## Information Theory

10. **Channel Capacity Estimation (§5.2)** — Research: practical methods to estimate channel capacity when the full transition matrix is available

11. **Fisher Information for Small n (§19.2)** — Verify: at what sample size does the asymptotic Gaussian approximation break down for Bernoulli trials?

## Systems & Architecture

12. **MapReduce / Monoid Properties (§7.2)** — Verify: are there edge cases where the monoid property breaks for real IndexedDB aggregations?

13. **Pushout Diagram for Overlay (§8.2)** — Verify: is a pushout the correct categorical structure, or is coproduct more accurate?

14. **Label-Based Extraction Robustness (§13.2)** — Research: how to handle tracker versions with field name changes (not just layout changes)?

## Domain-Specific (UPS Operations)

15. **Door Turn Rate Variance (§15.1)** — Check: is 800–1,200 packages/hour per door group still accurate (2026 update)?

16. **PD-04 Twilight Sort (§16.1)** — Verify: are WORMA N / PRORI N still the relevant wrap-time indicators?

17. **Jam-Breaker Overhead (§23.3)** — Confirm: is "exactly 4 people for 4 hours = 16 person-hours" still the operational rule?

18. **Borrow/Loan Economics (§11.3)** — Research: what % of sorts require cross-zone moves, and what's the typical ROI (PPH impact)?

---

## Research Plan for v1.2

### Web Search Tasks

- [ ] KDE bandwidth: latest research (Silverman vs others)
- [ ] Kalman filter initialization strategies
- [ ] CUSUM control limits for logistics/manufacturing
- [ ] MLR with categorical + continuous features (scaling, interaction terms)
- [ ] Channel capacity estimation methods

### Computational Verification

- [ ] Check 620-sort corpus for Simpson's Paradox instances
- [ ] Quantify phase-dependent PPH baseline expectations
- [ ] Verify IQR/Tukey fence application
- [ ] Test label-based extraction on tracker version changes

### Domain Expert Consultation

- [ ] Verify door turn rate (800–1,200/hr still current?)
- [ ] Confirm jam-breaker overhead model
- [ ] Check PD-04 wrap indicators

---

**Priority for v1.2:**
1. High: KDE bandwidth, Kalman filter initialization, CUSUM limits
2. Medium: MLR interactions, Simpson's Paradox quantification, Fisher info small-n
3. Low: Domain specifics (can defer to future session if UPS ops unchanged)
