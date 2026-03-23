# 📊 Sales Forecasting — Amazon Dataset

Comparison of four time-series models (SARIMA, Holt-Winters, LSTM, XGBoost) on 4 years of Amazon sales data. The best-performing model is used to project monthly revenue 12 months ahead.

---

## Dataset

| Property | Value |
|---|---|
| Source | Amazon Sales Dataset (provided as `Amazon 2_Raw.xlsx`) |
| Period | January 2011 – December 2014 |
| Records | 3 203 orders |
| Features | Order Date, Ship Date, Sales, Quantity, Profit, Category, Geography |
| Target | Monthly aggregated Sales ($) |

The raw transaction data is resampled to monthly frequency (48 periods). The train/test split is 80/20 — 38 months for training, 10 months (Mar–Dec 2014) for evaluation.

---

## Models

| # | Model | Description |
|---|---|---|
| 1 | **SARIMA(1,1,1)(1,1,0,12)** | Classical statistical model with seasonal differencing. Manually parameterised for stability on a short series. |
| 2 | **Holt-Winters** | Triple exponential smoothing (additive trend + additive seasonality). Parameters α, β, γ estimated by MLE. |
| 3 | **LSTM** | Two-layer recurrent neural network with 12-month lookback window, Dropout regularisation, Early Stopping. |
| 4 | **XGBoost** | Gradient-boosted trees with 12 lag features + 3-month rolling statistics. Hyperparameters tuned with `TimeSeriesSplit` GridSearchCV. |

### Results (test set: Mar–Dec 2014)


| Model | MAE ($) | RMSE ($) | MAPE (%) |
|---|---|---|---|
| SARIMA | 4181.74 | 4840.41 | 17.17 |
| Holt-Winters | 3473.91 | 4252.19 | 14.52 |
| LSTM | 5118.19 | 5810.27 | 24.70 |
| XGBoost | 5821.08 | 6889.67 | 24.65 |



## Project Structure

```
salesforecasting/
├── salesforecasting.ipynb         # Main notebook (19 sections)
├── Amazon 2_Raw.xlsx              # Raw dataset (required)
├── requirements.txt               # Python dependencies
├── .gitignore                     # Git ignore rules
├── README.md                      # This file
├── correlation_matrix.png         # Generated on run
└── residuals_analysis.png         # Generated on run
```

---

## How to Run

### 1. Clone and set up environment

```bash
git clone https://github.com/YOUR_USERNAME/salesforecasting.git
cd salesforecasting
```

#### Option A — pip (virtualenv recommended)

```bash
python -m venv .venv
# Windows:
.venv\Scripts\activate
# macOS/Linux:
source .venv/bin/activate

pip install -r requirements.txt
```

#### Option B — conda (recommended on Windows)

```bash
conda create -n forecasting python=3.10
conda activate forecasting
conda install -c conda-forge --file requirements.txt
```

### 2. Add the dataset

Place `Amazon 2_Raw.xlsx` in the project root (same folder as the notebook).

### 3. Launch

```bash
jupyter notebook salesforecasting.ipynb
```

Then **Run All** (`Kernel → Restart & Run All`).

---

## Key Findings

- **Monthly sales exhibit strong Q4 seasonality** with a clear upward trend across 2011–2014.
- **Holt-Winters** achieved the best performance overall (MAE $3,474 / 14.52% MAPE), effectively capturing both trend and additive seasonality on this short dataset.
- **SARIMA** was the second-best model (MAE $4,182 / 17.17% MAPE), providing a solid statistical baseline with seasonal differencing.
- **LSTM and XGBoost underperformed** the statistical models (~24–25% MAPE), likely due to the limited training data (only 38 months), which is insufficient for these data-hungry approaches to generalise well.
- SHAP analysis on the XGBoost model revealed that `Lag_1` (previous month's sales) is the most influential feature, followed by `Rolling_Mean_3M`.

---

## Dependencies

See `requirements.txt`. Main packages:

- `pandas`, `numpy`, `matplotlib`, `seaborn`
- `statsmodels` — SARIMA, Holt-Winters, ADF test
- `tensorflow` / `keras` — LSTM
- `xgboost`, `shap` — XGBoost + interpretability
- `scikit-learn` — preprocessing, metrics, cross-validation
- `pmdarima` — ACF/PACF utilities
- `openpyxl` — reading the `.xlsx` dataset

---

## Possible Improvements

- Integrate external signals (macroeconomic indicators, promotional calendars)
- Try more advanced architectures: N-BEATS, Temporal Fusion Transformer
- Automate monthly retraining with a pipeline (Airflow / GitHub Actions)
- Deploy the best model as a REST API for real-time forecasting

---

*Python 3.10+ · Last updated March 2026*
