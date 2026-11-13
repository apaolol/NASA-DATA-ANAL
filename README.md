# BlueShift: what we found and how we handled missing yield

BlueShift asks: **are ocean temperature and chlorophyll related to fish production in the supplied Mindanao data?**

This project studies the ten fish groups in the project brief. The notebook contains the code, statistical tests and plots. This README explains the results in everyday language.

## 1. The short answer

| Question | Answer |
|---|---|
| How much data did we analyse? | 23,040 rows for ten fish groups across the 24 province labels in the file. |
| What does one row represent? | One fish group, province, year and quarter. Yield is measured in metric tons. |
| How many yields were originally recorded? | 20,786. |
| How many yields were missing? | 2,254, or 9.78% of the selected rows. |
| How many blanks did we fill? | 2,063, using averages from the same fish group and province. |
| How many remain blank? | 191, because there is not enough local recorded data. |
| Did ocean conditions explain all production changes? | No. Some relationships appear, but the data do not prove that ocean changes caused production changes. |
| Did we forecast 2026 Q4? | No. This notebook analyses past records and fills historical gaps. |

**Important:** a recorded yield is a value already in the original file. A filled yield is an estimate. A blank is unknown—it does not automatically mean zero catch.

## 2. Which fish groups are included?

The original file contains 66 reporting categories. We selected the following ten to match the project brief. These are the project's priorities, not the ten biggest producers.

| Name in the brief | Category used in the dataset |
|---|---|
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

The mapping follows [PSA reporting names](https://rsso04a.psa.gov.ph/sites/default/files/attachment-dir/Annex%20A%20and%20Technical%20Note_0.pdf). Some names in the brief are mixed or broader than one biological species.

In particular, **Tamban uses Bali Sardinella, not the separate generic Sardines category**. Tulingan uses Frigate Tuna; Skipjack and Eastern Little Tuna remain separate and are excluded. Caesio, Grouper, Snapper and Spanish Mackerel are reporting categories, not confirmation of the exact scientific species in every record.

## 3. How did we fill missing yields?

We used **group-mean imputation**. This means replacing a blank with the average of suitable recorded yields.

### A simple example

Suppose sardine yield in one province was recorded as follows:

| Quarter in the same year | Yield, metric tons |
|---|---:|
| Q1 | 10 |
| Q2 | 20 |
| Q3 | Missing |
| Q4 | 30 |

We estimate Q3 as **(10 + 20 + 30) ÷ 3 = 20 metric tons**.

### What if the same year does not have enough recorded values?

Try the following steps in order. Stop at the first step with **at least two recorded values**.

| Step | Values used for the average | Example |
|---|---|---|
| 1 | Same fish, province and year | Other quarters of 2004 for that fish in that province. |
| 2 | Same fish, province and quarter | Q3 records from other years for that fish in that province. |
| 3 | Same fish and province, across all dates | All recorded quarters for that fish in that province. |
| 4 | None of these has enough values | Leave the yield blank. |

**We never borrow a different fish's yield or another province's yield.** We calculate every average from original recorded values, not from values we already filled.

Here is how the 2,063 fills were obtained:

| Average used | Values filled |
|---|---:|
| Other quarters in the same year | 255 |
| Same quarter in other years | 1,627 |
| All dates for the same fish and province | 181 |
| **Total** | **2,063** |

Most fills therefore use other years, not just the other quarters of the same year. Because later records can help fill earlier gaps, this is a method for completing historical data—not forecasting the future.

## 4. Which missing yields were filled?

Each fish category has 2,304 rows before analysis.

| Fish category | Originally missing | Filled | Still blank |
|---|---:|---:|---:|
| Bali Sardinella | 162 | 162 | 0 |
| Caesio | 296 | 296 | 0 |
| Frigate Tuna | 127 | 127 | 0 |
| Grouper | 155 | 155 | 0 |
| Indian Mackerel | 152 | 152 | 0 |
| Milkfish | 456 | 361 | 95 |
| Roundscad | 129 | 129 | 0 |
| Snapper | 217 | 217 | 0 |
| Spanish Mackerel | 235 | 235 | 0 |
| Tilapia | 325 | 229 | 96 |
| **Total** | **2,254** | **2,063** | **191** |

The remaining blanks are all in Sulu:

- **Milkfish:** only one yield is recorded for the entire local history. That is not enough to calculate a supported estimate under our two-value rule, so 95 blanks remain.
- **Tilapia:** no yield is recorded at any date for that province, so all 96 blanks remain.

Leaving these blank does not mean the fish was absent. The file cannot tell us whether the reason was absence, nonreporting or something else.

## 5. Did the filling method work well?

We checked it using yields whose true values we already knew:

1. Temporarily hide some recorded yields.
2. Calculate group means without looking at the hidden values.
3. Estimate the hidden yields.
4. Compare the estimates with the real values.

We tried scattered missing values and four consecutive missing quarters, repeating each check three times.

| Check | What it tells us | Result |
|---|---|---|
| Average absolute error, called MAE | How far estimates were from real yields, on average | 202.63 metric tons. |
| Estimates within ±500 metric tons | How often an estimate met the brief's error tolerance | 91.82% of cases where enough data remained to estimate. |
| Ability to produce an estimate | Whether the hidden value still had enough recorded local history | Almost all hidden values; a few scattered cases were unsupported. |

**About 92 out of 100 supported validation estimates were within 500 tons. This does not guarantee the same accuracy for every missing value or every fish.**

### Accuracy differs by fish

Lower average error is better. The last column reports the share of supported estimates within the ±500-ton tolerance.

| Fish category | Average error, metric tons | Within ±500 tons |
|---|---:|---:|
| Bali Sardinella | 686.97 | 83.26% |
| Caesio | 31.77 | 99.02% |
| Frigate Tuna | 340.17 | 83.26% |
| Grouper | 32.27 | 99.77% |
| Indian Mackerel | 108.31 | 96.12% |
| Milkfish | 376.54 | 77.45% |
| Roundscad | 257.50 | 86.04% |
| Snapper | 26.85 | 99.92% |
| Spanish Mackerel | 31.17 | 99.66% |
| Tilapia | 139.08 | 92.57% |

**Bali Sardinella's average error exceeds 500 tons.** Milkfish also has a noticeably lower within-tolerance percentage than several other categories. The overall average hides these differences. Lower-volume fish can have smaller absolute errors simply because their yields are smaller.

These checks measure historical filling accuracy. They do not test a 2026 Q4 forecast, and they cannot reveal the true yields of combinations with no recorded history.

## 6. What did EDA find?

EDA means **exploratory data analysis**: looking at distributions, relationships and patterns before drawing conclusions.

**All main EDA and statistical tests use recorded yields only.** We do not count our own estimates as new measurements. A separate section checks what changes after filling.

### A. One variable at a time

| Finding | Result | What it means |
|---|---|---|
| A typical recorded yield is much smaller than the average | Median 65.06 versus mean 380.75 metric tons | A few very large records pull the average upward. |
| Yields vary widely | Middle half: 12.49–308.88 metric tons | Production differs substantially between records. |
| Some records are extremely large | Maximum 56,521.69 metric tons | Large values strongly affect totals and error measures. |
| Average recorded ocean temperature | 29.73 °C | Describes conditions in the supplied measurements, not a temperature preferred by every fish. |
| Average chlorophyll concentration | 0.579 mg/m³ | Describes an ocean-productivity indicator, not fish production itself. |

The yield histograms have a long right tail. Log-scale plots compress large values so smaller differences become visible. **Using a log plot does not mean we used log imputation.** Missing yields were filled using ordinary averages.

### B. Looking at one relationship at a time

| Predictor | Result from the repeated-measurement test | Plain-language takeaway |
|---|---|---|
| Chlorophyll | Adjusted q-value 0.791 | No clear evidence of a linear relationship when considered alone. |
| Temperature | q = 0.074 | Does not meet the 0.05 cutoff when considered alone. |
| Year | q < 0.001 | A time relationship is detectable. |
| Province | q < 0.001 | Recorded yields differ across provinces. |
| Fish category | q < 0.001 | Recorded yields differ across fish categories. |
| Quarter | q = 0.0167 | A difference is detectable alone, but it explains very little variation. |

These tests examine one predictor at a time. Results can change when we account for other variables together. A detectable relationship does not necessarily explain much variation.

### C. Considering several predictors together

The model considers temperature, chlorophyll, province, fish category, quarter and year together. Its outcome is log(1 + recorded yield), which helps handle the wide yield range.

| Finding | Statistical result | Plain-language takeaway |
|---|---|---|
| Temperature has a positive adjusted relationship | q = 0.025 | Higher temperature is associated with higher recorded log yield in this model, after accounting for the included variables. This is not proof of a beneficial warming effect. |
| Chlorophyll has no clear common adjusted relationship | q = 0.650 | The model does not provide enough evidence for one shared linear chlorophyll relationship across all ten fish groups. |
| Environmental predictors considered together | q = 0.022 | The pooled model detects an environmental association, but it does not identify a causal mechanism. |
| The adjusted year relationship is negative | q = 0.0037 in the main model | The result is sensitive to how related observations are handled; an alternative calculation gives p = 0.0504. |
| Quarterly differences are not clear after adjustment | q = 0.601 | The earlier one-predictor quarter result does not remain clear in the full model. |
| Model explains about half the recorded log-yield variation | R² = 0.508 | This describes fit to the available records, not future forecast accuracy. |

A **q-value** is a p-value adjusted for checking multiple relationships. We use 0.05 as the cutoff within the stated group of tests. It is not the probability that a conclusion is true, an effect size or evidence of causation. Full coefficients, uncertainty intervals and test details are in the notebook.

### D. Are the relationships the same for every fish?

No. Simple correlation scores and adjusted model slopes differ by species.

- Some individual environmental slopes meet the adjusted 0.05 cutoff, including temperature for Caesio and both environmental predictors for Tilapia.
- However, **none of the ten tests that considers both environmental predictors together for an individual fish meets the adjusted 0.05 cutoff**.
- These are different questions and different groups of tests. An individual significant slope is not a general claim that a fish is most affected by ocean change.
- Milkfish and Tilapia may include cultured or inland production. Ocean measurements may not describe the water conditions where all of that production occurred.

The notebook provides the per-fish correlation scores, model slopes and confidence intervals. A bar crossing zero in a coefficient plot means the estimated direction is uncertain at that confidence level.

### E. What do the trend plots show?

Temperature and chlorophyll have positive fitted trends in the supplied environmental records. Temperature's fitted increase is about **0.025 °C per year**, after accounting for province and quarter.

Fish production plots show recorded annual totals, not complete production. A drop may reflect missing reports. The first year, 2002, includes only Q3–Q4; 2026 includes only Q1–Q2. Their totals should not be compared directly with full years.

## 7. What changes after filling?

The notebook compares the model using recorded yields with the model after adding supported estimates. It also tests scenarios where only filled values are reduced or increased by 25%.

The main coefficient directions remain the same in those scenarios: temperature is positive, chlorophyll negative and year negative. Their sizes change. **These comparisons do not add new significance claims.**

The completed annual totals are still incomplete because 191 yields remain blank. A higher total after filling means estimates were added; it does not show measured production growth.

## 8. Which files should I use?

| File | Use it for |
|---|---|
| `blueshift_dataset.csv` | Original data. It has not been overwritten. |
| `blueshift_analysis.ipynb` | Full analysis, plots, tests and explanations. |
| `analysis_outputs/blueshift_yield_completed.csv` | The ten selected fish categories with supported missing yields filled. |
| `requirements.txt` | Installing Python packages. |
| `README.md` | Understanding the workflow and main findings. |

### Important columns in the completed CSV

| Column | What it means |
|---|---|
| Original yield column | Original recorded values and blanks, preserved. |
| `yield_completed` | Recorded yield where available; otherwise a supported estimate or a remaining blank. |
| `yield_was_imputed` | `True` if that row was filled. |
| `yield_status` | Whether the yield is recorded, estimated, or still blank because history is missing or insufficient. |
| `imputation_method` | `group_mean` for filled rows. |
| `imputation_group` | Which averaging rule supplied the estimate. |
| `imputation_support_n` | How many original recorded values supported the estimate. Every fill has at least two. |

For main statistical tests, use the original recorded yield. For the documented completed-data summaries, use `yield_completed`. Keep the flags so estimates remain distinguishable from measurements.

## 9. How to run the notebook

Keep the original CSV beside the notebook. Then run:

```powershell
python -m pip install -r requirements.txt
jupyter notebook blueshift_analysis.ipynb
```

Open the notebook and run all cells from top to bottom. The settings and fish-name mapping are near the top. Only the completed CSV is saved; plots and tables stay inside the notebook. Rerunning replaces that output CSV.

The README tables summarize the current executed results. Update them if the data or settings change.

## 10. What we cannot conclude

- We cannot say that ocean conditions caused production changes. Fishing effort, reporting, location and production methods may also matter.
- We cannot quantify food-security or import-dependence impacts because the file has no consumption, trade, price or fishing-effort data.
- We cannot assume the 24 province labels represent complete coverage of Mindanao or verify the satellite measurement locations from this CSV alone.
- We cannot treat a filled yield as certain. Averages can smooth unusual events and large changes.
- We cannot claim future forecast accuracy from historical gap-filling tests. Forecasting 2026 Q4 needs future environmental inputs or clear scenarios and a time-ordered forecast test.

The full statistical methods, uncertainty checks and sources are documented in the notebook. The numerical results above come from the supplied data, not from the external fish-name reference.
