# Forecast-Driven Inventory Optimization

A supply chain analytics project that connects **demand forecasting** with **inventory policy decisions**. Using retail sales data from Favorita stores (Ecuador), the project forecasts weekly product-family demand and translates those forecasts into concrete replenishment parameters: safety stock, reorder point, and economic order quantity.

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

Three notebooks, one stage each:

| Notebook | Stage | What it does |
|---|---|---|
| `01_data_preparation` | Data prep | Load 3M+ rows, filter product families, aggregate daily store-level sales into clean weekly demand |
| `02_forecasting` | Forecasting | Holt-Winters exponential smoothing (trend + 52-week seasonality), evaluated on a 12-week hold-out with MAE / RMSE / MAPE / Bias |
| `03_inventory_optimization` | Inventory policy | Convert forecast-error uncertainty into safety stock, reorder point, and EOQ |

The forecasting and inventory logic are wrapped in reusable functions, so any product family can be evaluated with a single call.

---

## Key results

### Forecast accuracy (12-week hold-out)

| Family | Avg weekly demand | MAPE | Bias |
|---|---:|---:|---:|
| GROCERY I | ~1,423,000 | **5.52%** | +24,143 |
| BEVERAGES | ~899,000 | **11.25%** | +136,368 |

GROCERY I is markedly easier to forecast — staple groceries have stable, repeatable weekly demand, whereas beverages are more seasonal and volatile.

A data-quality issue was identified and fixed during analysis: the final week of the dataset was truncated (incomplete 7-day week), which initially inflated BEVERAGES MAPE to 30.6%. Removing it improved accuracy to 11.25%.

Both models showed a **positive bias** (over-forecasting). For BEVERAGES the bias accounts for most of the total error, signalling a systematic over-prediction that would inflate inventory if left uncorrected.

### Inventory policy (continuous review, 95% service level, 1-week lead time)

| Family | Safety stock | Reorder point | EOQ | Safety stock as % of weekly demand |
|---|---:|---:|---:|---:|
| BEVERAGES | ~184,900 | ~1,083,700 | ~176,500 | ~20.6% |
| GROCERY I | ~196,700 | ~1,620,000 | ~222,100 | ~13.8% |

**Core insight:** despite selling ~60% more, GROCERY I needs a *proportionally lighter* safety buffer. Because its demand is more predictable, its forecast error is smaller, so less buffer stock is required. **Better forecast accuracy translates directly into lower safety stock — and therefore less working capital tied up in inventory.** Forecast quality is a financial lever, not just a technical metric.

---

## Methodology notes

- **Forecast-error-based safety stock.** Safety stock is sized from the standard deviation of *forecast error* (not raw demand), using `z · sigma · sqrt(lead time)`. This ties the buffer directly to forecast reliability.
- **Chronological train/test split.** The series is never shuffled — the model learns from the past and is tested on the most recent 12 weeks, mirroring real forecasting.
- **Stated assumptions.** Ordering cost, holding cost (15%/year), and lead time (1 week) are not in the dataset, so they are set as documented assumptions following standard retail planning conventions.

---

## Repository structure

```
inventory-optimization/
├── data/
│   ├── raw/          # train.csv (download from Kaggle — not committed)
│   └── processed/    # weekly demand, forecasts, inventory policy
├── notebooks/
│   ├── 01_data_preparation.ipynb
│   ├── 02_forecasting.ipynb
│   └── 03_inventory_optimization.ipynb
├── visuals/          # exported charts
├── requirements.txt
└── README.md
```

---

## How to run

```bash
pip install -r requirements.txt
```

Download `train.csv` from the Kaggle link above into `data/raw/`, then run the notebooks in order (01 → 02 → 03).

---

## Possible extensions

- Add SARIMAX / XGBoost to capture short-term demand variation Holt-Winters misses
- Bias correction on the forecast before sizing inventory
- Monte Carlo simulation to optimize safety stock under stochastic demand
- Sensitivity analysis on lead time and service level

---

## Skills demonstrated

Time-series forecasting · demand planning · inventory theory (EOQ, safety stock, reorder point) · forecast error analysis (MAPE, bias) · data cleaning · Python (pandas, statsmodels, scipy) · translating analysis into business decisions