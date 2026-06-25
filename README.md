# Forecast-Driven Inventory Optimization

A supply chain analytics project that connects **demand forecasting** with **inventory policy decisions**. Using retail sales data from Favorita stores (Ecuador), the project forecasts weekly product-family demand, compares forecasting models, and translates the forecasts into concrete replenishment parameters: safety stock, reorder point, and economic order quantity.

The project is built around a practical planning question:

> How can a demand forecast be turned into inventory decisions that keep service levels high while controlling cost?

---

## Why this project

Most forecasting projects stop at "predict demand." This one goes one step further — into the decision that actually creates business value: **how much to order, when to reorder, and how much safety stock to hold.** That is the core of demand planning and procurement work.

---

## Dataset

Retail sales history from the **Store Sales - Favorita** dataset (Kaggle competition), covering 2013–2017 across 54 stores. The analysis focuses on two high-volume product families — **BEVERAGES** and **GROCERY I** — aggregated to weekly demand across all stores.

Dataset: https://www.kaggle.com/competitions/store-sales-time-series-forecasting/data

*(The raw `train.csv` is not included in this repo — download it from Kaggle and place it in `data/raw/`.)*

---

## Approach

Four notebooks:

| Notebook | Stage | What it does |
|---|---|---|
| `01_data_preparation` | Data prep | Load 3M+ rows, filter product families, aggregate daily store-level sales into clean weekly demand |
| `02_forecasting` | Forecasting | Holt-Winters baseline (trend + 52-week seasonality), evaluated with MAE / RMSE / MAPE / Bias |
| `03_inventory_optimization` | Inventory policy | Convert forecast-error uncertainty into safety stock, reorder point, and EOQ |
| `04_sarima_comparison` | Model selection | Compare Holt-Winters against SARIMA (auto-tuned with `auto_arima`) across both families |

Forecasting and inventory logic are wrapped in reusable functions, so any product family can be evaluated with a single call.

---

## Key results

### Model comparison (12-week hold-out)

| Family | Model | MAPE | Bias |
|---|---|---:|---:|
| BEVERAGES | Holt-Winters | 11.25% | +136,368 |
| BEVERAGES | **SARIMA** | **6.17%** | −77,941 |
| GROCERY I | Holt-Winters | 5.52% | +24,143 |
| GROCERY I | **SARIMA** | **4.01%** | +10,442 |

SARIMA (parameters auto-tuned via `auto_arima`) outperformed the Holt-Winters baseline on both families, improving accuracy and reducing bias.

**Key insight — match model complexity to demand difficulty.** The volatile family (BEVERAGES) benefited most from the more complex model (MAPE 11.25% → 6.17%), while the stable, predictable family (GROCERY I) improved only marginally (5.52% → 4.01%) — the simpler model was already nearly sufficient. In practice, model complexity should be matched to how hard each item is to forecast, rather than applied uniformly.

A data-quality issue was also identified and fixed during analysis: the final week of the dataset was truncated, which initially inflated BEVERAGES MAPE to 30.6%.

### Inventory policy (continuous review, 95% service level, 1-week lead time)

| Family | Safety stock | Reorder point | EOQ | Safety stock as % of weekly demand |
|---|---:|---:|---:|---:|
| BEVERAGES | ~184,900 | ~1,083,700 | ~176,500 | ~20.6% |
| GROCERY I | ~196,700 | ~1,620,000 | ~222,100 | ~13.8% |

**Core insight:** despite selling ~60% more, GROCERY I needs a *proportionally lighter* safety buffer. Because its demand is more predictable, its forecast error is smaller, so less buffer stock is required. **Better forecast accuracy translates directly into lower safety stock — and therefore less working capital tied up in inventory.**

---

## Methodology notes

- **Model selection by measurement.** Two forecasting approaches were tested and compared on a common hold-out, rather than committing to one model upfront.
- **Forecast-error-based safety stock.** Safety stock is sized from the standard deviation of *forecast error* (not raw demand), using `z · sigma · sqrt(lead time)`.
- **Chronological train/test split.** The series is never shuffled — the model learns from the past and is tested on the most recent 12 weeks.
- **Stated assumptions.** Ordering cost, holding cost (15%/year), and lead time (1 week) are documented assumptions following standard retail planning conventions.

---

## Repository structure

```
inventory-optimization/
├── data/
│   ├── raw/          # train.csv (download from Kaggle — not committed)
│   └── processed/    # weekly demand, forecasts, inventory policy, model comparison
├── notebooks/
│   ├── 01_data_preparation.ipynb
│   ├── 02_forecasting.ipynb
│   ├── 03_inventory_optimization.ipynb
│   └── 04_sarima_comparison.ipynb
├── visuals/          # exported charts
├── requirements.txt
└── README.md
```

---

## How to run

```bash
pip install -r requirements.txt
```

Download `train.csv` from the Kaggle link above into `data/raw/`, then run the notebooks in order (01 → 02 → 03 → 04).

---

## Possible extensions

- Add XGBoost with calendar/holiday feature engineering
- Bias correction on the forecast before sizing inventory
- Monte Carlo simulation to optimize safety stock under stochastic demand
- Sensitivity analysis on lead time and service level

---

## Skills demonstrated

Time-series forecasting (Holt-Winters, SARIMA) · model selection · demand planning · inventory theory (EOQ, safety stock, reorder point) · forecast error analysis (MAPE, bias) · data cleaning · Python (pandas, statsmodels, pmdarima, scipy) · translating analysis into business decisions