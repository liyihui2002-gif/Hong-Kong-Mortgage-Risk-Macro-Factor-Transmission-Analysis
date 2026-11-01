# The Impact of the Interest-Rate Hiking Cycle on Hong Kong Residential Mortgage Risks

**DSC5002 · Group 31 · Monthly exploratory analysis, 2018–2025**

## 1. Executive Summary

This project integrates HKMA mortgage and negative-equity surveys, HIBOR, Land Registry residential transactions, and Census & Statistics Department macro indicators into a 96-month analytical dataset.

During the hiking window, average monthly mortgage applications were 25.3% lower than before the cycle. Approvals fell 17.8% and residential transactions fell 24.3%. Three-month HIBOR has correlations of -0.70 with applications, -0.67 with approvals and -0.46 with transactions.

Applications and transactions peak at zero lag (r=0.67). Reversing the pair is a mirror of the same cross-correlation, so it is not an independent robustness check; the zero-lag maximum supports a largely synchronous market cycle rather than a stable one-way transmission. Negative equity behaves as a lagged risk: within the pre-set ±12-month search window, the strongest HIBOR relationship appears at the positive boundary (+12 months, r=0.85), but this is a search-window limit, not an identified transmission period, and the result is sensitive to the window and to the transformation.

The equal-weight Housing Mortgage Stress Index peaks at 58.7/100 in July 2024. The rate-dominant variant peaks at 60.7 in July 2024 and the borrower-dominant variant at 59.9 in September 2024, so the three schemes broadly agree on a high-stress window but not on an identical peak month.

## 2. Research Questions

1. How do mortgage applications and approvals change during a rising-rate cycle?
2. Do residential transactions lead or lag mortgage activity?
3. How is negative equity related to rates, unemployment and transaction activity?
4. Can a transparent composite index summarize monthly housing mortgage stress?

## 3. Data

The sample covers January 2018 through December 2025.

| Domain | Source | Variables | Frequency |
| --- | --- | --- | --- |
| Mortgage activity | HKMA T0307 | Applications, approvals, LTV, approved value | Monthly |
| Negative equity | HKMA T0316 | Cases, borrower share, value, unsecured portion | Quarterly |
| Rates | HKMA T060302 | 3-month HIBOR | Monthly average |
| Property market | Land Registry | Residential ASP cases and value | Monthly |
| Labour market | Census & Statistics Department | Unemployment rate | Rolling three-month |
| Inflation | Census & Statistics Department | Composite CPI YoY | Monthly |

GDP is not used in the monthly index because it is quarterly and is not one of the four proposed index components. CPI is retained as contextual evidence.

## 4. Method

### Comparison Windows

- Pre-hiking: January 2018 to February 2022
- Hiking cycle: March 2022 to August 2023
- Post-hiking: September 2023 to December 2025

The dates are analyst-defined from the sustained 3-month HIBOR rise and subsequent plateau. They are comparison windows rather than estimated structural breaks. August 2023 is a researcher-chosen transition point, not the sample's HIBOR peak: the monthly-average 3-month HIBOR actually reached its maximum of 5.438% in November 2023.

### Stress Index

Four sample-standardized components are combined:

1. 3-month HIBOR
2. Negative-equity cases
3. Negative three-month transaction change
4. Unemployment rate

```text
stress_index = clip(50 + 10 × Σ weight_i × z_i, 0, 100)
```

Three weighting schemes are tested: equal, rate-dominant and borrower-dominant. The output is a relative-risk gauge, not a default probability.

### Lead-Lag Diagnostics

Pearson correlations are calculated from -12 to +12 months using `corr(driver.shift(lag), outcome)`. Positive lag means the first variable leads. The maximum absolute correlation is exploratory and is not a causal or Granger test.

## 5. Results

| Metric | Pre-hiking | Hiking | Post-hiking |
| --- | ---: | ---: | ---: |
| Average 3M HIBOR | 1.25% | 3.28% | 4.01% |
| Monthly applications | 12,544 | 9,370 | 7,301 |
| Monthly approvals | 8,957 | 7,365 | 5,395 |
| Residential transactions | 5,168 | 3,911 | 4,514 |
| Approval/application ratio | 72.3% | 78.6% | 74.8% |
| Unemployment rate | 4.15% | 3.80% | 3.23% |
| Stress index | 47.6 | 50.2 | 54.5 |

### Mortgage Activity

Rates and mortgage activity move in opposite directions. The strongest contemporaneous relationships are with applications and approvals. HIBOR changes lead transaction declines by three months at the best tested lag, but the correlation is modest at -0.29.

### Transactions and Mortgage Activity

Applications and transactions peak at zero lag (r=0.67). Reversing the pair simply mirrors the same cross-correlation and therefore does not add independent evidence. Approvals and transactions are most correlated at a one-month transaction lead (r=0.61): `corr(approvals.shift(-1), transactions)` is the highest-|r| lag, meaning transactions lead approvals by one month. The reverse construction yields the mirror lag with the same coefficient, so the data support synchronicity, not a stable directional lead.

### Negative Equity

Negative-equity cases increase from 533 in 2022 Q3 to 12,164 in 2022 Q4, 32,073 in 2024 Q1 and 40,741 in 2025 Q1, before falling to 21,304 in 2025 Q4. In the pre-set ±12-month search window, HIBOR and negative equity are most correlated at the +12-month boundary (r=0.85). This boundary is a search limit, not an identified transmission lag: extending the search to +15 months raises the level correlation to 0.86, and a quarterly-only alignment (quarter-end HIBOR versus quarter-end negative equity) gives r=0.86 at a four-quarter lead but only r=0.27 on first differences. The delayed-collateral-stress interpretation is therefore plausible but not robustly supported, and none of this is causal evidence.

### Stress Index

The equal-weight index peaks at 58.7 in July 2024. The rate-dominant variant peaks at 60.7 in July 2024 and the borrower-dominant variant at 59.9 in September 2024. All three place the highest stress in the late-2023 to late-2024 window, but the peak month is not identical across specifications. The index eases in 2025 as rates fall and transactions recover, but risk remains above some early low-stress periods.

## 6. Conclusions

1. The hiking cycle coincides with weaker mortgage applications, approvals and transactions.
2. Transactions and mortgage activity are primarily synchronous.
3. Negative equity accumulates with a delay after rate and valuation shocks.
4. The composite index provides an interpretable common scale and is moderately robust to weighting choices.
5. Recovery in transaction volume does not imply immediate recovery in household leverage risk.

## 7. Limitations

- The sample includes COVID-19 and major policy changes.
- Negative equity is quarterly and forward-filled only for continuity.
- Approvals/applications is a ratio of separate flows, not a cohort approval rate.
- Correlation and lag selection can be affected by common trends and omitted variables.
- Aggregate monthly data cannot measure borrower-level default risk.
- The analysis is educational and is not an investment, underwriting or regulatory model.

## 8. Robustness Appendix

### A. Lead-lag sensitivity

Pearson correlations use `corr(driver.shift(lag), outcome)`, so a positive lag means the first variable leads. The headline lag window is −12 to +12 months, which is a pre-set search range, not an identified cycle length.

| Pair | ±12-month best lag | r | n | Extended search |
| --- | ---: | ---: | ---: | --- |
| 3M HIBOR → negative equity (levels) | +12 | 0.8464 | 84 | +15 months: r=0.8605, n=81 |
| 3M HIBOR → negative equity (quarterly, levels) | +4 quarters | 0.8572 | 28 | +5 quarters: r=0.8568, n=27 |
| 3M HIBOR → negative equity (quarterly, first differences) | +4 quarters | 0.2657 | 27 | +5 quarters: r=0.5916, n=26 |
| Approvals → transactions (levels) | −1 | 0.6077 | 95 | transactions lead approvals by 1 month |
| Applications → transactions (levels) | 0 | 0.6652 | 96 | synchronous |

The quarterly comparisons align quarter-end HIBOR with quarter-end negative equity and use fewer, different observations, so their sample sizes are not directly comparable to the monthly results. The first-difference correlation is much weaker than the level correlation, indicating the level results partly reflect common trends. These checks show the "12-month" result is a boundary of the search window and is sensitive to the window and transformation, not a robustly identified one-year transmission lag.

### B. Seasonal and negative-equity change

A dedicated seasonal adjustment and a negative-equity growth-rate analysis were proposed but are not yet implemented in the pipeline or reports. Negative-equity is a quarterly stock; its month-to-month change within a quarter is zero by construction after forward-filling and therefore not analysed as a monthly growth series. This is a scope reduction relative to the proposal and should be stated explicitly in the final report.

## 9. References

- Hong Kong Monetary Authority, Monthly Statistical Bulletin, Tables 3.7, 3.16 and 6.3.2.
- Hong Kong Land Registry, Monthly Statistical Data.
- Census and Statistics Department, Hong Kong, Table 210-06103 and Table 510-60001A.
- Project Proposal for DSC5002, Group 31.
