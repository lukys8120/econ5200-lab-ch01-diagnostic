# Data Quality Profiling — Big Mac Index

## Objective

Audit and remediate the data pipeline behind a Big Mac Index valuation analysis spanning 57 countries over 45 semi-annual periods (2,056 country-period observations), correcting a misspecified purchasing-power-parity ratio, quantifying the survivorship bias introduced by complete-panel filtering, and generalizing the diagnostics into a reusable DataFrame profiling utility.

## Methodology

- Reconstructed the PPP valuation calculation from first principles, defining the implied ratio as `r = implied_ppp / dollar_ex` and the over/undervaluation measure as `r - 1`.
- Traced an inverted-ratio defect in which the implementation evaluated `1/r - 1`. The error is reciprocal, not a sign flip, so it simultaneously reversed the cross-sectional ranking of currencies and inflated reported magnitudes beyond +100%.
- Patched the computation, re-based it on the July 2024 United States benchmark price of $5.69, and re-ran the full panel to confirm that valuation magnitudes returned to an economically admissible range.
- Isolated the sample-construction step that dropped any country lacking a complete observation history, and identified it as a source of survivorship bias.
- Quantified the bias by computing period-level averages under two sampling rules, complete-panel countries versus all available countries, and comparing level differences, percentage overstatement, and sign consistency across all 45 periods.
- Implemented `profile_dataframe()`, which classifies an arbitrary DataFrame as cross-sectional, time series, or panel, counts complete units, and reports per-column missingness.

## Key Findings

- **Inverted valuation ratio.** The original code returned the reciprocal of the intended ratio, ranking Taiwan (+149.4%), Indonesia (+131.3%) and Egypt (+130.4%) as the world's most overvalued currencies. Corrected, those three are among the most *under*valued and the ranking is headed by Switzerland at +41.8%, followed by Uruguay +24.3% and Norway +18.9%, against a median valuation of -20.7%. The defect therefore reversed the ordering and inflated the scale, having previously produced estimates beyond +100% — a currency more than twice overvalued, which is not an economically admissible result.
- **Survivorship bias is present and directional.** The complete-panel average ran $0.081 per period above the all-available average, a mean overstatement of +2.1%, and was the higher of the two series in 33 of 45 periods, including all six most recent periods.
- **The evidence rests on sign consistency, not any single cross-section.** The remaining 12 of 45 periods run in the opposite direction, and these cluster in the early data, where the two samples nearly coincide and the gap is not economically meaningful. The bias becomes both more consistent and more material as coverage widens over time.
- **Filtering to complete panels is not a neutral cleaning step.** The criterion is continuity of measurement, not income: Qatar, the highest-income country in the dataset, is dropped, as are Norway and Denmark, while Argentina is retained across all 45 periods. The retained sample is nonetheless materially richer and more expensive in aggregate — median GDP of 42,637 against 11,249, mean price of $4.56 against $4.31 in July 2024 — which is the mechanism behind the upward bias. Any statistic computed on the filtered sample should therefore be reported alongside its all-available counterpart.
- **Reusable diagnostics.** `profile_dataframe()` surfaces structure type, complete-unit counts, and column-level missingness before modeling, making the trade-off between panel completeness and sample representativeness explicit rather than implicit.
