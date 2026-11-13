# BlueShift: Mindanao priority-fish analysis

This report examines ocean conditions and quarterly production for the ten priority fish groups in the supplied BlueShift brief. The original CSV contains 66 reporting categories; this analysis selects ten using the explicit mapping below. They are not ranked by production.

The notebook contains reusable code, executed plots and short interpretations. The source CSV is preserved. Numerical tables below are snapshots of the current results and must be updated if the data or settings change.

## Read this first

- **For the main findings:** start with Section 1, then the species tables in Sections 4 and 7.
- **For the missing-value rule:** read Section 6. A fill always uses the same fish and province, with at least two recorded values.
- **For statistical detail:** read Section 5. Main tests use recorded values only; Section 8 separately checks the effect of estimated values.

| Term | Plain meaning | How to read it |
|---|---|---|
| Recorded yield | A measurement in the original file | Used for main EDA and tests. |
| Filled / imputed yield | An estimated value replacing a blank | Used only in the labelled completed-data comparison. |
| Coverage | Share of missing or hidden values that can be supported | High accuracy on a small supported subset is not enough. |
| Correlation (`rho`) | Whether two variables tend to move together | From −1 to +1; near zero means a weak monotonic relationship. |
| Adjusted slope (`beta`) | Estimated relationship while holding included predictors fixed | Positive and negative describe direction, not causation. |
| q-value (`q_BH`) | p-value adjusted for several tests | Below 0.05 meets the stated test-family cutoff; not a measure of effect size. |
| 95% confidence interval | Uncertainty around a model estimate | An interval spanning zero leaves the sign uncertain at that level. |
| MAE | Average absolute prediction error | In metric tons; lower is better. It is not a worst-case error. |
| RMSE | Error measure that emphasizes large misses | Also in metric tons; lower is better. |
| Within ±500 tons | Share of supported predictions meeting the error tolerance | Does not guarantee that every species or prediction meets the goal. |

**Three practical conclusions:** 2,063 gaps can be filled, 191 remain unsupported, and the ±500-ton goal is not met equally well across species. Environmental associations are exploratory; they do not establish that ocean change caused production change.

## 1. Findings at a glance

| Measure | Revised result | Meaning |
|---|---|---|
| Selected rows | 23,040 | Ten categories and the 24 province labels present in the file. |
| Recorded yields | 20,786 | Used for primary EDA and hypothesis tests. |
| Missing yields | 2,254 (9.78%) | Original blanks among the ten selected categories. |
| Local group-mean fills | 2,063 | Every estimate has at least two recorded values from the same species and province. |
| Left unresolved | 191 | Missing local history or insufficient support. |
| Validation MAE | 202.63 metric tons | Mean historical gap-filling error, not a maximum-error guarantee. |
| Within ±500 tons | 91.82% of supported validation cases | Does not establish Q4 2026 forecast performance. |

## 2. Exact mapping of the brief's priorities

| Priority name | Dataset species |
| --- | --- |
| Bangus | Milkfish |
| Tilapia | Tilapia |
| Galunggong | Roundscad |
| Tulingan | Frigate Tuna |
| Tamban | Bali Sardinella |
| Dalagang Bukid | Caesio |
| Lapu-Lapu | Grouper |
| Maya-Maya | Snapper |
| Alumahan | Indian Mackerel |
| Tanigue | Spanish Mackerel |

This operational mapping follows [PSA common-name reporting labels](https://rsso04a.psa.gov.ph/sites/default/files/attachment-dir/Annex%20A%20and%20Technical%20Note_0.pdf), not an assumption that the brief's mixed common and scientific names are exact equivalents.

- **Tulingan → Frigate Tuna:** PSA distinguishes this from Skipjack (Gulyasan) and Eastern Little Tuna. The brief mixes these labels; they are not silently merged.
- **Tamban → Bali Sardinella:** generic Sardines and Fimbriated Sardines are separate reporting categories. Their missingness must not be substituted for Tamban coverage.
- **Dalagang Bukid → Caesio:** a reporting proxy, not confirmation of the exact scientific species named in the brief.
- Grouper, Snapper and Spanish Mackerel are broad categories; exact biological species are not verified.
- Alumahan uses Indian Mackerel, not the separate Indo-Pacific Mackerel or Cavalla categories.

Excluded near-name categories, shown only to document the decision:

| Species | rows | recorded |
| --- | --- | --- |
| Eastern Little Tuna | 2304 | 1824 |
| Fimbriated Sardines | 2304 | 1809 |
| Sardines | 2304 | 51 |
| Skipjack | 2304 | 2115 |

Milkfish and Tilapia can involve inland, brackish or cultured production. The file has no fisheries-subsector field to distinguish those settings. Ocean measurements may be indirect proxies; do not interpret all ten categories as directly exposed wild marine stocks.

## 3. Data boundaries and workflow

| Section | Data used | Purpose |
|---|---|---|
| Coverage and missingness | Original selected rows, including blanks | Find reporting gaps by species and location. |
| Univariate EDA | Recorded yields; unique province-quarter environmental measurements | Inspect distributions and extremes. |
| Bivariate and adjusted models | Recorded yields only | Examine marginal and adjusted associations. |
| Per-species associations and trends | Recorded yields and original environmental records | Compare species-specific relationships and temporal patterns. |
| Imputation validation | Recorded yields temporarily hidden | Evaluate recovery without using hidden values in means. |
| After-imputation comparison | Recorded and completed yields, clearly labelled | Compare point estimates and partial totals. |

A recorded yield is present in the source; a missing yield is blank; an imputed yield is an estimate. A `series` is one species in one province followed over time. **Main EDA does not treat estimates as measurements.**

Coverage is 2002 Q3–2026 Q2, or 96 quarters. Boundary years have only two quarters. Environmental values repeat across species; summaries count each province-quarter once. The 24 labels in the file do not independently establish complete geographic coverage of Mindanao or stable historical administrative definitions.

## 4. Missing yield by priority species

| species | rows | recorded | missing | missing_pct | no_history_provinces | filled | unresolved |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Bali Sardinella | 2304 | 2142 | 162 | 7.031 | 0 | 162 | 0 |
| Caesio | 2304 | 2008 | 296 | 12.85 | 0 | 296 | 0 |
| Frigate Tuna | 2304 | 2177 | 127 | 5.512 | 0 | 127 | 0 |
| Grouper | 2304 | 2149 | 155 | 6.727 | 0 | 155 | 0 |
| Indian Mackerel | 2304 | 2152 | 152 | 6.597 | 0 | 152 | 0 |
| Milkfish | 2304 | 1848 | 456 | 19.79 | 0 | 361 | 95 |
| Roundscad | 2304 | 2175 | 129 | 5.599 | 0 | 129 | 0 |
| Snapper | 2304 | 2087 | 217 | 9.418 | 0 | 217 | 0 |
| Spanish Mackerel | 2304 | 2069 | 235 | 10.2 | 0 | 235 | 0 |
| Tilapia | 2304 | 1979 | 325 | 14.11 | 1 | 229 | 96 |

`missing_pct` describes original data; `filled` and `unresolved` describe the final output. Having only one recorded yield does not meet the two-observation rule.

Unresolved histories:

| Species | Province | yield_status | Rows |
| --- | --- | --- | --- |
| Milkfish | Sulu | unresolved_insufficient_history | 95 |
| Tilapia | Sulu | unresolved_no_local_history | 96 |

These are support decisions, not evidence that a species was absent. Missing values are never automatically converted to zero. The notebook's heatmap and chi-square screens describe coverage differences; repeated observations mean ordinary p-values cannot establish the reporting mechanism.

## 5. Summarized EDA findings

### Individual distributions

| Measure | Result | Interpretation |
|---|---|---|
| Yield median / mean | 65.06 / 380.75 metric tons | Right skew: a few large observations strongly influence averages. |
| Middle 50% | 12.49–308.88 metric tons | Wide variability in measured production. |
| 99th percentile / maximum | 4,322.42 / 56,521.69 metric tons | Upper-tail observations strongly influence raw-scale errors and totals. |
| Mean chlorophyll | 0.579 mg/m³ | Province-quarter proxy, not species-specific exposure. |
| Mean temperature | 29.73 °C | Satellite extraction footprints cannot be verified from this CSV. |

Log plots improve readability without making yields normal. Normality and rank tests are diagnostic screens in related observations. Equal quarterly row counts reflect dataset structure, not equal production.

### One-predictor models

| predictor | r2 | p | q_BH |
| --- | --- | --- | --- |
| chl | 0.0001428 | 0.7914 | 0.7914 |
| sst | 0.002653 | 0.06202 | 0.07442 |
| year | 0.006608 | 1.954e-06 | 3.909e-06 |
| province | 0.2118 | 3.144e-22 | 9.431e-22 |
| species | 0.2841 | 1.877e-22 | 9.431e-22 |
| quarter | 0.0002341 | 0.01114 | 0.01671 |

These allow observations within province-species combinations to be related but do not adjust for other predictors. Small q-values can coexist with very small explanatory power. Pooled scatter plots cannot isolate environmental effects from species, place and time differences.

### Pooled adjusted model

Model log(1 + yield) using chlorophyll, temperature, species, province, quarter and a linear year trend. Environmental units are one standard deviation; the year unit is one calendar year.

| index | beta | CI_low | CI_high | q_BH | p_province_cluster |
| --- | --- | --- | --- | --- | --- |
| chl_z | -0.01197 | -0.06416 | 0.04022 | 0.65 | 0.7313 |
| sst_z | 0.06197 | 0.01148 | 0.1125 | 0.02503 | 0.004885 |
| year_c | -0.01905 | -0.0304 | -0.007698 | 0.003693 | 0.0504 |

Joint-term tests:

| term | F | p | q_BH | status |
| --- | --- | --- | --- | --- |
| environment | 4.209 | 0.01772 | 0.02215 | estimable |
| year | 11.1 | 0.001231 | 0.002052 | estimable |
| quarter | 0.6249 | 0.6007 | 0.6007 | estimable |
| province | 18.46 | 1.237e-25 | 3.093e-25 | estimable |
| species | 37.69 | 1.443e-27 | 7.217e-27 | estimable |

R² = **0.508** in-sample, not forecast accuracy. In this selected scope, temperature has a positive adjusted slope, while chlorophyll lacks clear evidence of a common slope. The year estimate is negative, but its province-clustered sensitivity p = **0.0504**, showing dependence-model uncertainty.

Coefficients describe log(1 + yield), not arithmetic mean production. Primary uncertainty uses two-way clustering by province-species combination and time; sensitivity clusters by province. Slope q-values adjust three focal tests; omnibus q-values adjust five joint-term tests. Residual diagnostics show imperfect fit; clustering does not fix misspecification or missingness bias.

### Species-specific correlation scores and adjusted slopes

`rho` is the unadjusted Spearman correlation. `beta` is the adjusted environmental slope, controlling for province, quarter and year. Slope q-values adjust all 20 environmental tests; `joint_q` adjusts the ten species-level two-slope tests.

| species | rho_chl | rho_sst | beta_chl | beta_sst | q_BH_chl | q_BH_sst | joint_q |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Bali Sardinella | 0.2742 | -0.09059 | -0.03461 | -0.02006 | 0.7371 | 0.9328 | 0.6979 |
| Caesio | -0.008462 | 0.1334 | -0.04313 | 0.1041 | 0.5965 | 0.04083 | 0.07286 |
| Frigate Tuna | -0.03032 | -0.1357 | 0.05383 | 0.006194 | 0.628 | 0.9487 | 0.6979 |
| Grouper | 0.02012 | 0.105 | -0.0196 | 0.06392 | 0.9328 | 0.3456 | 0.4627 |
| Indian Mackerel | 0.1084 | -0.01042 | 0.01562 | 0.06393 | 0.9487 | 0.5522 | 0.6193 |
| Milkfish | 0.2127 | 0.2479 | 0.01694 | 0.1557 | 0.9328 | 0.1466 | 0.2812 |
| Roundscad | 0.02431 | -0.1265 | -0.007941 | 0.06876 | 0.9487 | 0.5522 | 0.6193 |
| Snapper | 0.02699 | 0.04779 | -0.06412 | 0.06714 | 0.628 | 0.3456 | 0.3126 |
| Spanish Mackerel | 0.003213 | 0.02228 | -0.06995 | -0.002441 | 0.3456 | 0.9682 | 0.4068 |
| Tilapia | 0.2038 | 0.2844 | 0.09171 | 0.121 | 0.04083 | 0.04083 | 0.07286 |

Some individual slope tests meet q < 0.05, but **none of the ten per-species joint tests meets q < 0.05**. These are different test families. The notebook includes confidence intervals; plotted 95% intervals are not multiplicity-adjusted.

Do not interpret association ranking as which fish is causally affected most. Location, fishing effort, culture and reporting can explain apparent patterns. Tilapia-ocean associations need particular habitat caution. Some nuisance-effect variances under two-way clustering are invalid; inference is restricted to supported environmental coefficients and valid joint tests.

### Ocean and production trends

| variable | slope_per_year | CI_low | CI_high | p | q_BH |
| --- | --- | --- | --- | --- | --- |
| chl | 0.003468 | 0.0009984 | 0.005938 | 0.007981 | 0.007981 |
| sst | 0.02508 | 0.02411 | 0.02604 | 1.098e-25 | 2.196e-25 |

Environmental trends adjust for quarter and province, with province-clustered uncertainty. Temperature and chlorophyll have positive fitted trends in the supplied measurements, but this does not validate the exposure footprint. Species production plots show recorded partial annual totals: reporting gaps and partial boundary years can resemble production changes. These are not stock-abundance estimates.

## 6. Local-only group-mean imputation

Use the first group with at least two recorded yields:

1. Same species, province and year: other quarters.
2. Same species, province and quarter: other years.
3. Same species and province: all dates.
4. Otherwise, leave the yield blank.

**No cross-species or cross-province borrowing.** Filled values never enter another mean. Historical records on either side of a gap are allowed, so this is retrospective filling, not future forecasting. For recorded quarterly yields of 10, 20 and 30, the missing-quarter estimate is 20 metric tons.

Actual source groups:

| imputation_group | Filled rows |
| --- | --- |
| same_quarter_other_years | 1627 |
| same_year | 255 |
| local_species_history | 181 |

Group mean is retained for simplicity at the user's request and revalidated on this scope. It is not proven universally optimal; previous all-species scores are not evidence of best performance for these ten categories.

## 7. Validation against the ±500-ton goal

Hide 20% of recorded values or four consecutive quarters per combination; repeat each pattern three times with fixed seeds. Recompute means after hiding. Accuracy excludes unsupported cases, while `coverage_pct` reports the supported fraction. Overlapping repeats are not independent experiments.

| scenario | coverage_pct | MAE | RMSE | within_500_pct |
| --- | --- | --- | --- | --- |
| block | 100 | 232.4 | 1,191 | 91.23 |
| scattered | 99.99 | 172.9 | 664.6 | 92.4 |

Per species:

| species | coverage_pct | MAE | RMSE | within_500_pct |
| --- | --- | --- | --- | --- |
| Bali Sardinella | 100 | 687 | 2,469 | 83.26 |
| Caesio | 100 | 31.77 | 85.23 | 99.02 |
| Frigate Tuna | 100 | 340.2 | 794.5 | 83.26 |
| Grouper | 100 | 32.27 | 64.87 | 99.77 |
| Indian Mackerel | 100 | 108.3 | 239 | 96.12 |
| Milkfish | 99.95 | 376.5 | 690.4 | 77.45 |
| Roundscad | 100 | 257.5 | 540.6 | 86.04 |
| Snapper | 100 | 26.85 | 50.88 | 99.92 |
| Spanish Mackerel | 100 | 31.17 | 67.79 | 99.66 |
| Tilapia | 100 | 139.1 | 389.5 | 92.57 |

**Bali Sardinella has average MAE above 500 tons.** Pooled MAE below 500 therefore does not mean every species meets the target. Milkfish has fewer predictions within ±500 than several low-volume categories. No result guarantees a maximum error or a Q4 2026 forecast.

MAE is average absolute error; RMSE emphasizes large errors. RMSLE in the notebook is a log-scale evaluation metric, not log imputation. Total bias is also reported. Performance on known yields may not represent yields missing for different reporting reasons.

## 8. After-imputation sensitivity

| scenario | n | chl_z | sst_z | year_c |
| --- | --- | --- | --- | --- |
| observed_only | 20786 | -0.01197 | 0.06197 | -0.01905 |
| completed | 22849 | -0.007343 | 0.05818 | -0.01877 |
| imputed_minus25 | 22849 | -0.008951 | 0.05784 | -0.01768 |
| imputed_plus25 | 22849 | -0.006084 | 0.05845 | -0.01964 |

These are coefficient comparisons, not new significance tests. The ±25% changes to filled yields are assumed scenarios, not confidence intervals. Completed annual totals remain partial; adding estimates is not evidence of measured production growth.

## 9. Files, execution and output columns

| File | Purpose |
|---|---|
| `blueshift_dataset.csv` | Original 66-category input; unchanged. |
| `blueshift_analysis.ipynb` | Revised executed analysis. |
| `analysis_outputs/blueshift_yield_completed.csv` | **Selected ten categories only**, replacing the previous broad-scope export. |
| `requirements.txt` | Python dependencies. |
| `README.md` | Scope, methods, results and limitations. |

```powershell
python -m pip install -r requirements.txt
jupyter notebook blueshift_analysis.ipynb
```

Keep the CSV beside the notebook and run all cells in order. The explicit mapping and settings are at the top. Only the completed CSV is written; plots and tables stay inside the notebook. No helper scripts or standalone images are required. Rerunning replaces the output CSV.

| Added column | Meaning |
|---|---|
| `yield_completed` | Recorded value or supported local estimate; otherwise blank. |
| `yield_was_imputed` | True only for filled rows. |
| `yield_status` | `recorded`, `imputed`, `unresolved_no_local_history`, or `unresolved_insufficient_history`. |
| `imputation_method` | `group_mean` for filled rows. |
| `imputation_group` | First qualifying group used. |
| `imputation_support_n` | Recorded observations supporting the estimate; at least two for every fill. |

Original measurement columns are retained. There is no unsupported suggestion column in this revision. Use original yield for primary tests; use completed yield only for the documented descriptive comparisons.

## 10. Food-security and deployment limits

- The project brief supplies motivation, not additional observations. There are no import, export, consumption, price, effort or subsector fields, so food-security and import-dependence impacts cannot be quantified.
- Province measurements are analysis proxies, not claims of provincial ocean ownership. The file does not verify satellite matching or geographic completeness.
- Broad reporting labels, changing administrative definitions and uneven recording constrain interpretation. Milkfish and Tilapia are not automatically marine capture fisheries.
- Single imputation does not fully capture uncertainty. Main tests use recorded yields only; few geographic clusters and imperfect covariance still warrant caution.
- No Q4 2026 forecast is produced. Future environmental inputs or justified scenarios and forward-in-time backtesting are needed. Historical imputation is not forecast validation.

Sources: supplied BlueShift brief; [PSA category-name table](https://rsso04a.psa.gov.ph/sites/default/files/attachment-dir/Annex%20A%20and%20Technical%20Note_0.pdf); [PSA fisheries statistics](https://psa.gov.ph/system/files/main-publication/Fisheries%2520Statistics%2520of%2520the%2520Philippines%252C%25202019-2021_0.pdf?vcode=GW2I). Numerical findings come from the supplied CSV and revised notebook, not those external tables.
