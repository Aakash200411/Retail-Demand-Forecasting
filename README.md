# Retail Demand Forecasting & Inventory Planning

An end-to-end data science project using the **M5 Forecasting dataset** to analyse historical retail demand, compare forecasting approaches, generate a 28-day demand forecast, and translate forecast uncertainty into inventory-planning scenarios.

> **Scope note:** The saved run in this repository uses one store (`CA_1`) and six selected item-store series (two products from each of the three product categories). It is a focused, runtime-conscious demonstration, not a full M5 competition solution. The selected products are chosen using sales history available before the first backtest window.

## Project objectives

- Inspect and validate the supplied sales, calendar, price and submission data.
- Explore demand patterns across products and categories.
- Engineer historical demand features without using future sales information.
- Compare statistical, baseline and machine-learning forecasting methods using rolling-origin validation.
- Tune a small set of XGBoost configurations using time-aware validation.
- Produce a 28-day forecast and inventory-planning scenarios, including safety stock and reorder points.
- Export metrics, predictions, forecasts and inventory outputs for further analysis.

## Dataset

The project uses the five M5 CSV files:

| File | Purpose |
|---|---|
| `sales_train_validation.csv` | Historical daily sales used for model development and backtesting |
| `sales_train_evaluation.csv` | Extended sales history used for the final forecast period |
| `calendar.csv` | Calendar dates, weekdays, events and SNAP indicators |
| `sell_prices.csv` | Weekly item-store selling prices |
| `sample_submission.csv` | Reference submission format |

The M5 dataset is large. **Do not commit the raw CSV files to GitHub by default**; follow the dataset's terms and use the official source/download instructions. Place the five files in the project root when running locally, or upload them to the Colab runtime.

## Methodology

1. **Data validation:** check source-file availability, duplicate identifiers, calendar-key uniqueness, price-key uniqueness and consistency between validation/evaluation series.
2. **Leakage-aware selection:** select the top two products per category for store `CA_1` using only the earliest backtest training window.
3. **Exploratory analysis:** inspect demand over time, category/product contributions, weekday and event patterns, zero-demand behaviour, prices and SNAP indicators.
4. **Feature engineering:** create lagged and rolling demand features using past observations, alongside available calendar and price covariates.
5. **Rolling-origin evaluation:** compare models over four consecutive, non-overlapping 28-day validation windows. Each forecast origin uses only information available before its test period.
6. **Model comparison:** evaluate Naive, Seasonal Naive, Moving Average, Exponential Smoothing, Random Forest and XGBoost. Tune three bounded XGBoost configurations.
7. **Forecasting:** generate a 28-day forecast for the selected item-store series using the best mean rolling-origin model.
8. **Inventory planning:** estimate lead-time demand, safety stock, reorder points and suggested order quantities under explicit assumptions; test alternate lead times and service levels.

## Results from the saved notebook run

### Forecast model comparison

Mean WAPE across the four rolling-origin windows:

| Model | Mean WAPE | Std. dev. of WAPE |
|---|---:|---:|
| Exponential Smoothing | **53.18%** | 2.84 percentage points |
| Moving Average | 56.00% | 4.50 percentage points |
| Tuned XGBoost (`xgb_depth4_lr003`) | 57.61% | 1.62 percentage points |
| Random Forest | 61.81% | 1.90 percentage points |
| Original XGBoost | 62.43% | 5.01 percentage points |
| Seasonal Naive | 67.44% | 3.65 percentage points |
| Naive | 95.72% | 30.51 percentage points |

**Interpretation:** Exponential Smoothing had the lowest mean rolling-origin WAPE in this experiment. XGBoost tuning improved on the original XGBoost baseline, but did not outperform Exponential Smoothing. WAPE is an aggregate error measure; lower is better.

### Forecast summary

The final 28-day forecast contains **140 rows** (5 item-store series with complete evaluation history × 28 days) and totals approximately **1,020.7 units**.

| Category | Forecast units |
|---|---:|
| FOODS | 97.2 |
| HOBBIES | 520.9 |
| HOUSEHOLD | 402.6 |

The selected validation scope contains six series; one series did not have complete evaluation history and was excluded from the final forecast. Forecasts were non-negative and had no missing values in the saved run.

### Inventory planning assumptions

The example inventory plan uses a **7-day lead time**, **7-day review period**, **95% service level**, and **zero on-hand inventory**. These are scenario assumptions, not values supplied by the M5 dataset. Suggested order quantities should therefore be interpreted as illustrative outputs, not operational purchase recommendations.

## Repository structure

```text
retail-demand-forecasting/
├── notebooks/
│   └── Retail_Demand_Forecasting.ipynb
├── outputs/
│   ├── overall_model_metrics.csv
│   ├── rolling_origin_metrics.csv
│   ├── model_stability_summary.csv
│   ├── xgboost_tuning_comparison.csv
│   ├── backtest_predictions.csv
│   ├── final_28_day_forecast.csv
│   ├── forecast_summary_by_store_category.csv
│   ├── inventory_scenario_plan.csv
│   └── inventory_sensitivity.csv
├── README.md
├── requirements.txt
└── .gitignore
```

The `outputs/` directory is where the notebook writes its generated CSV files. Download the **latest run's** output folder from Colab and place those CSVs here before committing. Do not use CSVs from an older notebook run because their schemas, selected products and metrics may differ.

## Run the project

### Option A — Google Colab

1. Open `notebooks/Retail_Demand_Forecasting.ipynb` in Google Colab.
2. Upload the five M5 CSV files to the Colab runtime (or mount a Drive folder and update `DATA_DIR`).
3. Run the notebook from top to bottom.
4. Confirm that the final quality-control cell reports all checks as `PASS`.
5. Download the generated `retail_forecasting_outputs` folder and copy its CSV files into this repository's `outputs/` directory.

To download all generated CSVs as one ZIP from Colab, run:

```python
import shutil
from google.colab import files

shutil.make_archive(
    "/content/retail_forecasting_outputs",
    "zip",
    root_dir="/content",
    base_dir="retail_forecasting_outputs"
)
files.download("/content/retail_forecasting_outputs.zip")
```

If your `OUTPUT_DIR` is different, update `root_dir`, `base_dir` and the ZIP path accordingly.

### Option B — VS Code / local Jupyter

1. Clone or download this repository.
2. Download the five M5 source CSVs separately and place them in the repository root.
3. Create and activate a virtual environment.
4. Install dependencies:

```bash
python -m venv .venv

# Windows PowerShell
.venv\Scripts\Activate.ps1

# macOS / Linux
source .venv/bin/activate

pip install -r requirements.txt
```

5. Open `notebooks/Retail_Demand_Forecasting.ipynb` in VS Code and run all cells.

The notebook detects whether it is running in Colab or locally and writes generated files to `retail_forecasting_outputs/` by default. If you want local outputs directly in `outputs/`, change `OUTPUT_DIR` in the configuration cell to `Path("outputs")`.

## Output files

| Output | Description |
|---|---|
| `overall_model_metrics.csv` | Pooled evaluation metrics for each model |
| `rolling_origin_metrics.csv` | Metrics for each model and validation origin |
| `model_stability_summary.csv` | Mean and standard deviation of WAPE, MAE and RMSE across origins |
| `xgboost_tuning_comparison.csv` | Candidate XGBoost tuning results, if exported by the notebook |
| `backtest_predictions.csv` | Actual and predicted demand for each backtest observation |
| `final_28_day_forecast.csv` | Daily forecast values and calendar information |
| `forecast_summary_by_store_category.csv` | Forecast totals and average daily demand by store/category |
| `inventory_scenario_plan.csv` | Lead-time demand, safety stock, reorder point and suggested order quantity |
| `inventory_sensitivity.csv` | Inventory results under alternative lead times and service levels |

## Limitations and responsible interpretation

- Results represent a small selected sample from one store, not the entire M5 dataset or all retail stores.
- The tuned XGBoost candidate was selected using the same four rolling-origin windows reported for tuning. Its score is therefore a model-selection result, not an untouched final test score.
- Exponential Smoothing is the best model in the saved comparison; the notebook's final forecast uses the selected best model rather than assuming XGBoost must win.
- Inventory outputs depend on illustrative lead-time, service-level, review-period and on-hand assumptions. Real deployment requires validated operational inputs and business review.
- Historical associations with events, prices or SNAP indicators should not be interpreted as causal effects.

## Tools and technologies

Python, Pandas, NumPy, Matplotlib, Seaborn, scikit-learn, XGBoost, Statsmodels, Jupyter Notebook / Google Colab.

## License and data attribution

This repository contains project code and analysis. Check the original M5 dataset terms and attribution requirements before redistributing source data. Add a project license only if you intend to specify reuse terms for your own code.
