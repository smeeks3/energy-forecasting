# Energy Load Forecasting

A time series forecasting project for American Electric Power's Appalachian Power Territory (AEPAPT), comparing Exponential Smoothing (ESM) and Seasonal ARIMA (SARIMA) models to predict total daily energy demand. The full business write-up is available in `reports/Report.pdf`.

## Overview

This project develops and evaluates two forecasting approaches for daily energy load (MW) in a deregulated energy market. The SARIMA model outperformed ESM on all accuracy metrics, producing forecasts within approximately 7% of actual daily demand, providing a strong foundation for operational planning and peak-demand management.

## Methodology

- **Data** — Hourly MW load data from PJM's Data Miner (AEPAPT region), aggregated to daily totals; 1,857 training observations (Aug 2018–Aug 2023) and 366 validation observations (Sep 2023–Aug 2024)
- **ESM model selection** — Six candidate models compared via AICc; automated ETS search identified the best-fit model with multiplicative error, no trend, and additive seasonality — ETS(M,N,A)
- **SARIMA model selection** — Dual seasonality (weekly and yearly) addressed with Fourier terms (K=3 weekly, K=2 yearly), selected via grid search and AICc elbow plot; one non-seasonal difference applied based on KPSS testing; AR/MA terms identified from ACF/PACF analysis and validated with a Ljung-Box white noise test
- **Final SARIMA model** — SARIMAX(3,1,1)(2,0,2) with weekly (K=3) and yearly (K=2) Fourier terms

## Model Performance

| Model | MAE | MAPE |
|---|---|---|
| ESM — ETS(M,N,A) | 11,332.69 MW | 10.23% |
| SARIMA — (3,1,1)(2,0,2) | 7,296.90 MW | 6.81% |

The SARIMA model captures both weekly fluctuations and the broader yearly seasonal pattern, including winter peak demand, where the ESM model falls short.

## Repository Structure

```
energy-load-forecasting/
├── energy_2025.csv
├── Energy_Forecasting.Rmd
└── Report.pdf
```

## Getting Started

```r
# Required packages
install.packages(c("tidyverse", "fpp3", "forecast", "fable", "fabletools"))

# Open and knit the R Markdown file
rmarkdown::render("Energy_Forecasting.Rmd")
```

## Tools & Libraries

![R](https://img.shields.io/badge/R-4.0+-blue)
![fable](https://img.shields.io/badge/fable-ESM%20%7C%20ARIMA-orange)
![fpp3](https://img.shields.io/badge/fpp3-time%20series-green)
