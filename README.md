# Forecast-Driven Inventory Optimization

A supply chain analytics project that connects **demand forecasting** with **inventory policy decisions**. Using retail sales data from Favorita stores (Ecuador), the project forecasts weekly product-family demand and translates those forecasts into concrete replenishment parameters: safety stock, reorder point, and economic order quantity.

The project is built around a practical planning question:

> How can a demand forecast be turned into inventory decisions that keep service levels high while controlling cost?

---

## Why this project

Most forecasting projects stop at "predict demand." This one goes one step further — into the decision that actually creates business value: **how much to order, when to reorder, and how much safety stock to hold.** That is the core of demand planning and procurement work.

---

## Dataset

Retail sales history from the **Store Sales - Favorita** dataset (Kaggle competition). The analysis focuses on the **BEVERAGES** product family, aggregated to weekly demand across all stores.

Dataset: https://www.kaggle.com/competitions/store-sales-time-series-forecasting/data

*(The raw `train.csv` is not included in this repo — download it from Kaggle and place it in `data/raw/`.)*

---

## Approach

The workflow has three stages, one notebook each:

| Notebook | Stage | What it does |
|---|---|---|
| `01_data_preparation` | Data prep | Load 3M+ rows, filter product family, aggregate daily store-level sales into clean weekly demand |
| `02_forecasting` | Forecasting | Holt-Winters exponential smoothing with trend + seasonality, evaluated on a 12-week hold-out |
| `03_inventory_optimization` | Inventory policy | Convert forecast uncertainty into safety stock, reorder point, and EOQ |

---

## Key results (BEVERAGES)

**Forecasting** — Holt-Winters with weekly seasonality (52-week cycle):

| Metric | Value |
|---|---:|
| MAPE | 11.25% |
| Average weekly demand | ~898,800 units |

A data-quality issue was identified and fixed during analysis: the final week of the dataset was truncated (incomplete 7-day week), which initially inflated MAPE to 30.6%. Removing it improved forecast accuracy to 11.25%.

**Inventory policy** — continuous review (s, Q), 95% target service level, 1-week lead time:

| Parameter | Value |
|---|---:|
| Forecast-error sigma | ~112,400 |
| Safety stock | ~184,900 units |
| Reorder point (s) | ~1,083,700 units |
| Economic order quantity (Q) | ~176,500 units |

**Business insight:** because the forecast achieved ~11% MAPE, safety stock stays relatively lean. A weaker forecast would require a thicker buffer — directly increasing tied-up working capital. Forecast quality translates straight into inventory cost.

---

## Methodology notes

- **Forecast-error-based safety stock.** Safety stock is sized from the standard deviation of *forecast error* (not raw demand), using the classic `z · sigma · sqrt(lead time)` formula. This ties the buffer directly to forecast reliability.
- **Chronological train/test split.** The series is never shuffled — the model learns from the past and is tested on the most recent 12 weeks, mirroring how forecasting works in practice.
- **Stated assumptions.** Ordering cost, holding cost (15%/year), and lead time (1 week) are not in the dataset, so they are set as documented assumptions, following standard retail planning conventions.

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
- Monte Carlo simulation to optimize safety stock under stochastic demand
- Sensitivity analysis on lead time and service level
- Extend to multiple product families

---

## Skills demonstrated

Time-series forecasting · demand planning · inventory theory (EOQ, safety stock, reorder point) · data cleaning · Python (pandas, statsmodels, scipy) · translating analysis into business decisions
