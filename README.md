# TIME-SERIES-CASE-STUDY
German Electricity Demand Forecasting - Time Series Case Study

This repository contains my work for the electricity demand forecasting case study. I compare simple benchmark forecasts, SARIMA/SARIMAX models, a feature-based Random Forest model and an hourly LSTM model using German electricity demand data.

The aim is not just to chase the lowest error number. I also wanted to understand why each model behaved the way it did, and whether the model would actually be sensible to maintain in a real forecasting setting.

## Project files

| File | Purpose |
|---|---|
| `Trinath Chandra Bose Bolla.ipynb` | Main notebook containing the full data preparation, plots, modelling and evaluation. |
| `Trinath Chandra Bose Bolla.pdf` | Final report submitted for the assignment. |
| `time_series_60min_singleindex_filtered.csv` | Filtered German electricity demand dataset used by the notebook. |
| `README.md` | Short guide to the project, files, methods and results. |

## Data used

The analysis uses German electricity demand from the Open Power System Data time-series dataset. The original observations are hourly. For most models, the load values are aggregated to weekly demand because the assignment focuses on a two-year weekly forecast horizon.

Original electricity data source:

https://data.open-power-system-data.org/time_series/2020-10-06/time_series_60min_singleindex.csv

The German demand column used in the work is `DE_load_actual_entsoe_transparency`.

Berlin temperature data is used as a representative weather input. This was taken from the Open-Meteo historical weather API:

https://archive-api.open-meteo.com/v1/archive

Public holiday indicators are also added because holidays can affect electricity demand and are known before the forecast date.

## What the notebook does

The notebook is written as one complete workflow. In simple terms, it does the following:

1. Loads and prepares German electricity demand data.
2. Builds hourly and weekly time series.
3. Explores demand patterns using time plots, seasonal views and autocorrelation checks.
4. Tests stationarity using ADF/KPSS-style analysis and differencing checks.
5. Creates benchmark forecasts: mean, naive, seasonal naive and drift.
6. Fits SARIMA models using AIC-based parameter search.
7. Extends the model to SARIMAX using temperature and holiday variables.
8. Builds a Random Forest model using lag, rolling, weather and calendar features.
9. Trains and compares LSTM designs on hourly demand.
10. Compares the models and discusses which one is most suitable for operational use.

## Main modelling approach

### Benchmarks

The benchmark models give a simple comparison point. Seasonal naive is especially important here because German electricity demand has a strong yearly repeating pattern. If a more complicated model cannot beat this, then the extra complexity is not automatically justified.

### SARIMA and SARIMAX

SARIMA is used because the weekly demand series shows seasonality. The notebook searches across candidate non-seasonal orders and keeps a 52-week seasonal structure. SARIMAX then adds temperature and holiday variables.

The weather-based SARIMAX results are interpreted carefully. Observed future temperature is not normally known at the forecast origin, so temperature-based forecasts are conditional unless future weather forecasts are supplied.

### Random Forest

The Random Forest model is the main feature-based model in this version. It uses previous demand values, calendar patterns, weather variables, holiday indicators and rolling summaries. Lag features are created from earlier observations so that the model does not accidentally use the target week as an input.

### LSTM

The LSTM is trained on hourly demand rather than weekly demand. It is useful for short-horizon hourly forecasting, but it is more complex and less interpretable than the weekly statistical and feature-based models. Because of this, I discuss the LSTM result separately instead of treating it as a direct replacement for the weekly models.

## Result snapshot

The main results from the notebook are:

| Model | Frequency | RMSE | MAE | MAPE |
|---|---|---:|---:|---:|
| LSTM | Hourly | 1690.02 | 1282.10 | 2.40% |
| Random Forest | Weekly | 2505.60 | 1826.49 | 3.51% |
| Seasonal Naive | Weekly | 3006.76 | 2318.52 | 4.41% |
| SARIMA | Weekly | 4765.23 | 3883.62 | 7.37% |
| SARIMAX + Temperature + Holidays | Weekly | 5633.88 | 4712.47 | 8.92% |

The Random Forest model gives the best weekly forecasting result in this version. The LSTM has the lowest error overall, but it is evaluated on hourly data, so it should not be treated as a direct like-for-like comparison with the weekly models.

## Key findings

- Seasonal naive is a strong baseline because German electricity demand repeats annually.
- Random Forest improves the weekly forecast by using lagged load and calendar/weather features.
- SARIMA and SARIMAX are useful for interpretation, but their forecast accuracy is weaker in this run.
- Temperature and holiday variables are meaningful, but they do not automatically guarantee better SARIMAX performance.
- LSTM performs well for hourly forecasting, but it is harder to explain and maintain.

## How to run the notebook

1. Keep the notebook and dataset in the same project folder.
2. Open `Trinath Chandra Bose Bolla.ipynb` in Jupyter Notebook or Google Colab.
3. Run the cells from top to bottom.
4. Check that the generated plots and tables match the report.

If the local CSV file is not available, the OPSD source link above can be used to download the original time-series file again.

If running in a fresh Python environment, install the main packages first:

```bash
pip install pandas numpy matplotlib seaborn statsmodels scikit-learn tensorflow holidays requests
```

## Notes on forecast interpretation

- The final two years are used as the test period.
- Weekly models and hourly LSTM models are reported separately because they use different data frequencies.
- Holiday features are safe for operational forecasting because holiday dates are known in advance.
- Temperature variables are only fully operational if they come from weather forecasts available at the forecast origin.
- The recommended weekly operational model in this version is Random Forest, with seasonal naive kept as a simple backup benchmark.

## References

The report discusses the modelling choices and literature in more detail. Key references include standard time-series forecasting material, SARIMA/SARIMAX methods, Random Forest/feature-based forecasting ideas and LSTM literature for sequential demand forecasting.

Useful source links:

- Open Power System Data time-series file: https://data.open-power-system-data.org/time_series/2020-10-06/time_series_60min_singleindex.csv
- Open-Meteo historical weather API: https://archive-api.open-meteo.com/v1/archive
- Assignment report: `Trinath Chandra Bose Bolla.pdf`
