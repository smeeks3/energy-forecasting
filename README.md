# Energy Load Forecasting

Forecasts total daily electricity demand (MW) for the AEP load area (RFC/WEST market region) using seven years of hourly PJM grid data. Two model families are compared — Exponential Smoothing (ETS) and Seasonal ARIMA with Fourier terms (SARIMAX) — with the best SARIMAX model achieving a validation MAPE of **6.81%** and MAE of **7,296 MW**, outperforming the best ETS model (MAPE 10.23%, MAE 11,333 MW) by roughly 33%.

## Overview

Grid operators and energy traders need reliable short-horizon load forecasts to balance supply and demand, schedule generation assets, and manage costs. This project builds and compares classical time series models on real hourly load data pulled from the PJM energy market, aggregated to daily totals, and evaluated on a held-out validation year.

The dataset (`energy_2025.csv`) contains 62,113 hourly observations covering **August 2018 through August 2025** for the AEPAPT load area. The full analysis and narrative are documented in `Report.pdf`.

## Data

| Field | Detail |
|---|---|
| Source | PJM energy market (NERC region: RFC, market region: WEST, zone: AEP) |
| Granularity | Hourly MW readings → aggregated to daily total MW |
| Raw rows | 62,113 |
| Date range | Aug 1, 2018 – Aug 31, 2025 |
| Target variable | `daily_total` (sum of hourly `mw` per day) |

### Train / Validation / Test Split

| Split | Period | Purpose |
|---|---|---|
| Train | Aug 2018 – Aug 2023 | Model fitting |
| Validation | Sep 2023 – Aug 2024 (366 days) | Model selection |
| Test | Sep 2024 – Jan 2025 | Final held-out evaluation |

## Models

### 1. Exponential Smoothing (ETS)

Six ETS variants were fit and ranked by AIC, then evaluated on the validation set by MAPE:

| Model | Components |
|---|---|
| SES | Error(A), Trend(N), Season(N) |
| Linear | Error(A), Trend(A), Season(N) |
| Linear Damped | Error(A), Trend(Ad), Season(N) |
| Holt-Winters Additive | Error(A), Trend(A), Season(A) |
| Holt-Winters Multiplicative | Error(M), Trend(A), Season(M) |
| **Auto-Search (best)** | **ETS(M,N,A)** — Multiplicative error, No trend, Additive seasonality |

**Validation performance (best ETS model):**
- MAPE: **10.23%**
- MAE: **11,332.69 MW**

### 2. Seasonal ARIMA with Fourier Terms (SARIMAX)

Daily energy load exhibits two seasonal cycles (weekly and yearly). Fourier terms were used to handle both because standard seasonal ARIMA cannot accommodate periods as large as 365.

**Fourier term selection:**
- Weekly Fourier K: grid-searched over K ∈ {1, 2, 3} → **K = 3** selected
- Yearly Fourier K: elbow plot of AICc over K ∈ {1, …, 13} → **K = 2** selected (balances fit vs. complexity)

**Differencing:** KPSS and `unitroot_ndiffs` tests required **d = 1** first difference to achieve stationarity.

**AR/MA order selection:** ACF and PACF plots on differenced residuals guided the grid search:
- Non-seasonal: p ∈ {3, 4}, q ∈ {1, 2}
- Seasonal: P ∈ {0, 1, 2}, Q ∈ {0, 1, 2}

**Top model AICc scores from grid search:**

| Model | AICc |
|---|---|
| SARIMA(3,1,1)(2,0,2) | 37,795.14 |
| SARIMA(4,1,1)(2,0,2) | 37,796.97 |
| SARIMA(3,1,2)(2,0,2) | 37,797.34 |
| SARIMA(3,1,1)(1,0,0) | 37,797.37 |
| SARIMA(3,1,1)(0,0,1) | 37,797.43 |

**Best model selected:** `SARIMAX(3,1,1)(2,0,2)` with Fourier terms (Kw=3, Ky=2)
- Passed Ljung-Box white noise test (lag = 21, dof = 8)
- The auto-search ARIMA did **not** produce white noise residuals and was excluded

**Validation performance:**
- MAPE: **6.81%**
- MAE: **7,296.02 MW**

## Results Summary

| Model | Validation MAPE | Validation MAE |
|---|---|---|
| ETS(M,N,A) | 10.23% | 11,332.69 MW |
| **SARIMAX(3,1,1)(2,0,2) + Fourier** | **6.81%** | **7,296.02 MW** |

SARIMAX reduces MAE by ~**3,937 MW** (~35%) over the best ETS model.

## Repository Structure

```
Energy-Load-Forecasting/
├── Energy_Forecasting.Rmd   # Full analysis: data prep, ETS, ARIMA modeling, plots
├── energy_2025.csv          # Raw hourly PJM load data (Aug 2018 – Aug 2025)
└── Report.pdf               # Written report with results, diagnostics, and discussion
```

## Dependencies

R packages used:

```r
tidyverse, fpp3, forecast, fable, fabletools, ggplot2
```

## Usage

1. Open `Energy_Forecasting.Rmd` in RStudio.
2. The data is loaded directly from the public GitHub raw URL — no local path change needed unless working offline (update the `read_csv()` path on line 23 to point to `energy_2025.csv`).
3. Knit to HTML to reproduce all plots and model outputs.

## Author

Steven Meeks — November 2025
