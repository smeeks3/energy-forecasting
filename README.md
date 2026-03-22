# Energy Load Forecasting

Forecasts total daily electricity demand (MW) for the PJM AEP load area using seven years of hourly grid data. Two classical time series families — Exponential Smoothing (ETS) and Seasonal ARIMA with Fourier terms (SARIMAX) — are built and compared on a held-out validation year. The best model, SARIMAX(3,1,1)(2,0,2) with weekly and yearly Fourier terms, achieves a validation MAPE of **6.81%** and MAE of **7,296 MW**, outperforming the best ETS model by ~35%.

## Overview

Grid operators and energy traders need reliable short-horizon load forecasts to balance supply and demand, schedule generation assets, and manage costs. This project applies classical time series methods to real hourly PJM load data, aggregated to daily totals, with model selection driven by a held-out validation year (Sep 2023 – Aug 2024). The dataset (`energy_2025.csv`) covers **August 2018 through August 2025** across 62,113 hourly observations for the AEPAPT load zone (NERC region: RFC, market region: WEST). Full methodology, diagnostics, and discussion are in `Report.pdf`.

## Data

| Field | Detail |
|---|---|
| Source | PJM energy market |
| Granularity | Hourly MW → aggregated to daily total MW |
| Raw rows | 62,113 |
| Date range | Aug 1, 2018 – Aug 31, 2025 |
| Target | `daily_total` (sum of hourly `mw` per day) |

**Train / Validation / Test Split**

| Split | Period |
|---|---|
| Train | Aug 2018 – Aug 2023 |
| Validation | Sep 2023 – Aug 2024 |
| Test | Sep 2024 – Jan 2025 |

## Results

| Model | Validation MAPE | Validation MAE |
|---|---|---|
| ETS(M,N,A) | 10.23% | 11,333 MW |
| **SARIMAX(3,1,1)(2,0,2) + Fourier (Kw=3, Ky=2)** | **6.81%** | **7,296 MW** |

The SARIMAX model passed the Ljung-Box white noise test; the auto-search ARIMA did not and was excluded. See `Report.pdf` for full model diagnostics, Fourier term selection, and residual analysis.

## Repository Structure

```
Energy-Load-Forecasting/
├── Energy_Forecasting.Rmd   # Full analysis: data prep, ETS, ARIMA modeling, plots
├── energy_2025.csv          # Raw hourly PJM load data (Aug 2018 – Aug 2025)
└── Report.pdf               # Written report with results, diagnostics, and discussion
```

## Dependencies

```r
tidyverse, fpp3, forecast, fable, fabletools, ggplot2
```

## Usage

1. Open `Energy_Forecasting.Rmd` in RStudio.
2. Data loads from the public GitHub raw URL by default. To work offline, update the `read_csv()` path on line 23 to point to `energy_2025.csv`.
3. Knit to HTML to reproduce all plots and model outputs.

## Author

Steven Meeks — November 2025
