# Time Series Forecasting & Backtesting

> **Two forecasting case studies that judge models the way a risk team would: against a naive benchmark, across multiple out-of-sample windows, and with the bias of each model on the record.**

![Python](https://img.shields.io/badge/python-3.11+-blue) ![Course](https://img.shields.io/badge/Time%20Series%20Forecasting-C41230) ![Status](https://img.shields.io/badge/status-completed-green)

---

## TL;DR

| | |
|---|---|
| **Question** | Does a forecasting model actually beat the simplest alternative, and does it keep winning on periods it has never seen? |
| **Case 1 · Bank stock** | For JPMorgan Chase monthly prices, the "best" smoothing model turned out to be the naive forecast (α = 1.0). Only a combination of models beat it (MASE 3.19 vs. 3.63) |
| **Case 1 · Macro signals** | Unemployment, CPI and the fed funds rate look strongly correlated with JPM on price levels, but the link disappears once trends are removed |
| **Case 2 · Backtesting** | Across 4 rolling 12-month windows, multiplicative Triple ETS ranked **1st in every window** (avg RMSE 1.53, MAPE 6.65%); models without seasonality were ~2.5× worse |
| **Takeaway** | A benchmark and a multi-window backtest caught what single-split accuracy would have hidden |

---

## 📋 Project Overview

**Why this matters:** in risk work, a forecast is only useful if it beats doing nothing clever, and a model validated on one test period can simply have been lucky. These studies apply the same checks used in model validation and VaR backtesting: compare against a naive benchmark, test over several out-of-sample windows, and look at the sign of the errors, not just their size.

---

## 📊 Datasets

| | Case 1 | Case 2 |
|---|---|---|
| **Series** | JPMorgan Chase (JPM) monthly median price | Monthly anti-diabetic drug sales, Australia |
| **Period** | Jan 2014 onward (153 monthly observations) | Jul 1991 – Jun 2008 (204 monthly observations) |
| **Extra data** | Unemployment rate, CPI, effective fed funds rate (FRED) | — |
| **Source** | Yahoo Finance, Federal Reserve Bank of St. Louis | Course dataset |
| **Shape** | Strong trend, no meaningful seasonality | Strong trend and a 12-month seasonal cycle |

---

## 🏗️ Project Structure

```
time-series-forecasting/
│
├── Predict Median Monthly Stock Price.ipynb          # Case 1: JPM, macro indicators, ETS models
├── Back Test ETS and ARIMA Time Series Models.ipynb  # Case 2: ETS vs. ARIMA, 4-window backtest
└── README.md
```

---

## 🔍 Key Findings from EDA

- **JPM has trend but no seasonality:** once prices are converted to monthly returns, all twelve months look alike, and the seasonal component is under 3% of price.
- **Correlation on levels is misleading:** two independent random series reach a correlation of ~0.89 once the same trend is added to both. JPM, CPI and rates all trended upward, which inflated their apparent relationship.
- **Macro data arrives too late to help:** indicators are published one to two months after the reference period, and markets price in expectations before release.
- **Drug sales are non-stationary with growing seasonal peaks:** the ADF test confirms non-stationarity, and the January peaks get larger as the level rises, which points to multiplicative seasonality.

---

## 🚀 Methodology

### Case 1 — JPM monthly price ([notebook](Predict%20Median%20Monthly%20Stock%20Price.ipynb))
- Tested whether three macro indicators carry predictive signal, on levels, on differences, and with lags.
- Fitted Simple, Double and Triple exponential smoothing (additive and multiplicative), held out the last 12 months.
- Scored with RMSE, MAE, MAPE, sMAPE, and **MASE against a naive last-value forecast**.

### Case 2 — Drug sales backtest ([notebook](Back%20Test%20ETS%20and%20ARIMA%20Time%20Series%20Models.ipynb))
- Stationarity checks (ADF), first and seasonal differencing, ACF/PACF for model orders.
- 8 candidates: Simple, Double and Triple ETS (additive and multiplicative), manual ARIMA(2,1,1), auto ARIMA, auto seasonal ARIMA, and an ensemble average.
- **Rolling backtest:** 4 consecutive 12-month test windows (Jul 2004 – Jun 2008), each model refit only on data before the window.
- Ranked by average error, average rank across windows, and mean error (bias).

---

## 📈 Results

**Case 1 — JPM, 12 held-out months:**

| Model | RMSE | MAE | MAPE | **MASE** |
|---|---|---|---|---|
| **Average (SES + two Double ETS)** | **19.62** | **15.06** | **4.95%** | **3.19** |
| SES, optimised α (= naive) | 25.47 | 17.16 | 5.10% | 3.63 |
| Naive benchmark | 25.47 | 17.16 | 5.10% | 3.63 |
| Double ETS additive | 28.92 | 24.85 | 8.03% | 5.26 |
| Triple ETS additive | 37.32 | 34.11 | 10.93% | 7.21 |

> The optimiser chose α = 1.0, which turns simple smoothing into the naive forecast: no weighting of history beat the most recent price. The trend model lost because it extrapolated the 2024–2025 rally into a flat 2026, and all twelve of its errors had the same sign. Only the combination forecast beat the benchmark.

**Case 2 — Drug sales, average over 4 backtest windows:**

| Model | RMSE | MAE | MAPE | Mean error | Avg rank |
|---|---|---|---|---|---|
| **ETS Triple (multiplicative)** | **1.53** | **1.31** | **6.65%** | **0.57** | **1.0** |
| ETS Triple (additive) | 1.69 | 1.49 | 7.63% | 0.61 | 2.0 |
| Auto SARIMA (seasonal) | 1.81 | 1.57 | 8.06% | 0.77 | 3.0 |
| Ensemble average | 2.59 | 2.16 | 10.60% | 1.65 | 4.0 |
| Non-seasonal models | ~3.2–3.9 | — | — | ~2.7 | 5–8 |

> Multiplicative Triple ETS was the most accurate model in every window, not just on average, and had the smallest bias. Every model under-forecast (positive mean error) because of the persistent upward trend; the gap was 0.57–0.77 for seasonal models against ~2.7 for non-seasonal ones. Unlike Case 1, the ensemble did not help here: averaging in the weak non-seasonal models dragged it down.

**Limitations:** Case 1 tests one 12-month window, so its ranking is less robust than Case 2's; MASE here compares a 12-step forecast error with a 1-step naive error, so values above 1 are expected and only the relative comparison with the naive row is meaningful; neither study models external shocks.

---

## 💡 Risk Analyst Perspective

> These are the same questions asked when validating any model in production: does it beat a trivial benchmark, does it hold up across periods it was not fitted on, and is it biased in one direction? The multi-window backtest in Case 2 follows the same logic as VaR backtesting, and the benchmark comparison in Case 1 is what kept a naive forecast from being mistaken for a smart one.

---

## 🛠️ Tech Stack

- **Language:** Python 3.11+
- **Forecasting:** statsmodels (ETS, ARIMA, ADF test), pmdarima (auto ARIMA)
- **Data:** pandas, NumPy, yfinance, pandas-datareader (FRED)
- **Visualization:** matplotlib, seaborn

---

## 🏃 Getting Started

```bash
git clone https://github.com/Olivia-Yoob/time-series-forecasting.git
cd time-series-forecasting
jupyter lab
```

Both notebooks also open directly in Google Colab.

---

## 📬 Contact

**Olivia Kim (Yoobin Kim)**
- LinkedIn: [linkedin.com/in/olivia-yoobin-kim](https://www.linkedin.com/in/olivia-yoobin-kim/)
- GitHub: [@Olivia-Yoob](https://github.com/Olivia-Yoob)
- Email: yoobink@andrew.cmu.edu

---

**Project Status:** Completed September 2026
