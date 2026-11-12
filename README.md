# Blueshift analysis

Open `blueshift_analysis.ipynb` and run all cells with Python 3. Install the analysis dependencies with:

```powershell
python -m pip install -r requirements.txt
```

Keep `blueshift_dataset.csv` beside the notebook, or update `DATA_PATH` in the first code cell. Run all cells in order. Tables, plots, tests, validation results, and brief interpretations are contained in the notebook. The only exported file is `analysis_outputs/blueshift_yield_completed.csv`. The source CSV is preserved.

The report follows this order: setup, data checks, missingness, recorded-data EDA, imputation, after-imputation comparison, and findings. The EDA section contains univariate, bivariate and multivariate analysis. Every results table and plot includes a short interpretation. Main EDA and hypothesis tests use only recorded yields; the separate after-imputation section uses filled values to check how results change.

Imputation uses only the selected group-mean method. It first averages other recorded quarters for the same province, species and year. When fewer than two values are available, it follows the documented fallback groups. Means use recorded yields, never other imputed values.

Validation hides scattered values and consecutive quarterly gaps, with three repeats per pattern. MAE, RMSE, RMSLE, total bias and distribution spread are reported. RMSLE is an evaluation metric; imputation uses ordinary means without a log transformation. Alternative imputation code has been removed. Regression models in the EDA remain statistical analyses of recorded yields.

The group-mean hierarchy was selected from the earlier candidate comparison. Only 3,648 of the 42,170 eligible fills have enough observations for a same-year quarterly mean; the remaining fills use the documented fallback groups. The notebook reports this distinction explicitly.

The completed CSV keeps the original measurement columns and adds `yield_completed`, `yield_was_imputed`, `yield_status`, `imputation_method`, and `yield_model_suggestion`. Province–species combinations with no recorded yield anywhere in the dataset remain blank by default. Suggestions for those combinations are estimates without recorded yields for that combination to guide them; they are kept separate from completed yields.

Primary hypothesis tests use recorded yields. Single-imputed results are used only for descriptive sensitivity. All tests are exploratory and observational; the notebook documents dependence, multiplicity, missingness, and model limitations beside the relevant results.

The notebook is self-contained and includes executed results. No helper scripts are required. Package versions are displayed in the first code cell. To open it locally after installing the requirements, run `jupyter notebook blueshift_analysis.ipynb`.
