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

## Main figures

### Figure 1. Daily German electricity demand with rolling median

This plot shows daily German electricity demand from 2015 to 2020. The orange line is the 30-day rolling median, which makes the long-term movement easier to see. The blue band shows the normal variation range. The repeated peaks and dips show that demand has a strong seasonal pattern.

### Figure 2. Average hourly demand: weekday vs weekend

This figure compares the average hourly electricity demand on weekdays and weekends. Weekday demand is clearly higher, especially during working hours. This shows why hourly demand forecasting needs to consider daily routines and working-day effects.

### Figure 3. Monthly seasonal index

This bar chart shows how average demand changes by month compared with the overall average. Values above 1 mean demand is higher than usual. The result shows stronger demand in winter months and weaker demand in some spring/summer months.

### Figure 4. Seasonal decomposition of weekly demand

This decomposition separates weekly electricity demand into trend, seasonal and residual components. The seasonal part confirms a repeating annual pattern, while the trend shows changes in the overall demand level over time.

### Figure 5. ACF and PACF of differenced weekly load

The ACF and PACF plots help identify remaining autocorrelation after differencing. The visible spike around lag 52 supports the use of a yearly seasonal period in the SARIMA model.

### Figure 6. Benchmark forecasts

This figure compares the benchmark forecasts with actual weekly demand over the test period. Seasonal naive follows the yearly pattern better than the mean, naive and drift models, which is why it becomes the main baseline.

### Figure 7. Benchmark RMSE comparison

This bar chart ranks the benchmark models by RMSE. Seasonal naive has the lowest error, showing that repeating the same week from the previous year is a strong simple method for this dataset.

### Figure 8. SARIMA forecast

This figure compares the SARIMA forecast with actual weekly demand and shows the forecast interval. The model captures some seasonal movement, but it does not follow the 2020 demand drop very well.

### Figure 9. SARIMA residual diagnostics

The SARIMA residual plots show the remaining errors after fitting the model. Most residuals are close to zero, but a few large errors remain, suggesting that the SARIMA model does not fully capture unusual demand changes.

### Figure 10. Temperature and electricity demand

This scatter plot shows the relationship between Berlin weekly temperature and German weekly electricity demand. Demand is generally higher at lower temperatures, showing why temperature can be a useful explanatory variable.

### Figure 11. SARIMAX forecast with external covariates

This figure shows the SARIMAX forecast after adding temperature and holiday variables. The external variables add useful context, but the forecast still does not fully beat the stronger benchmark or Random Forest model.

### Figure 12. Random Forest forecast

This plot compares the Random Forest forecast with actual weekly demand. The model follows the general movement of weekly demand better than SARIMA/SARIMAX in this version, especially because it uses lag and calendar-based features.

### Figure 13. Random Forest feature importance

This chart shows the most important input features in the Random Forest model. The 52-week load lag is the strongest feature, confirming that last year’s demand is very important for forecasting this year’s weekly demand.

### Figure 14. Random Forest feature group importance

This grouped importance chart shows that lag features are much more important than weather, holiday and rolling features. This supports the idea that yearly demand memory is the main driver of the weekly forecast.

### Figure 15. LSTM training and validation loss

This figure shows how the LSTM training and validation loss changed over epochs. Both losses decrease, which suggests that the model learned useful hourly demand patterns without obvious divergence during training.

### Figure 16. LSTM forecast compared with actual hourly demand

This plot shows a ten-day sample of the LSTM forecast against actual hourly demand. The forecast follows the daily peaks and troughs closely, showing that LSTM is useful for short-term hourly forecasting.

### Figure 17. Weekly aggregated LSTM forecast

This figure aggregates the hourly LSTM predictions into weekly averages. The forecast line is very close to the actual weekly load, but this result should be interpreted carefully because it comes from a rolling hourly setup.

### Figure 18. Final model comparison

This bar chart compares the main models using RMSE. LSTM has the lowest error overall, but it uses hourly data. For weekly operational forecasting, Random Forest is the strongest practical model in this version.

### Figure 19. Weekly forecasts compared against actual demand

This figure compares the weekly forecasts from Seasonal Naive, SARIMA, SARIMAX and Random Forest against actual weekly demand. It shows how the models behave differently during the same two-year test period.

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

If the local CSV file is not available, the OPSD source link below can be used to download the original time-series file again.

## Notes on forecast interpretation

- The final two years are used as the test period.
- Weekly models and hourly LSTM models are reported separately because they use different data frequencies.
- Holiday features are safe for operational forecasting because holiday dates are known in advance.
- Temperature variables are only fully operational if they come from weather forecasts available at the forecast origin.
- The recommended weekly operational model in this version is Random Forest, with seasonal naive kept as a simple backup benchmark.
## Answers for the questions in the assignment
## Answers were reported in the report in the designated paragrapghs 5,6,etc

## What the Model Comparison Revealed

Seasonal Naive provided a demanding weekly benchmark, with RMSE 3006.76, MAE 2318.52 and MAPE 4.41%. Random Forest delivered the only meaningful weekly improvement, reducing RMSE to 2505.60 and MAPE to 3.51%. This represents an RMSE reduction of approximately 16.7%, showing that lagged demand, calendar information and engineered features added predictive value beyond repeating the corresponding week from the previous year.

SARIMA produced RMSE 4765.23 and MAPE 7.37%, while temperature-and-holiday SARIMAX returned RMSE 5633.88 and MAPE 8.92%. Neither statistical model surpassed Seasonal Naive during the two-year test period. The LSTM achieved RMSE 1690.02 and MAPE 2.40%, but these values were calculated from hourly observations. Its performance should not be ranked directly against the weekly models until both forecasts and observations are evaluated at the same frequency.

## Why an Annual Seasonal Structure Was Retained

The forecasting series was weekly, meaning that approximately 52 observations represented one annual cycle. This agreed with the recurring winter–summer demand pattern and the seasonal dependence visible in the exploratory and autocorrelation results. The final SARIMA specification was `(4,2,6)(1,1,1,52)`, using non-seasonal differencing `d=2`, seasonal differencing `D=1` and a seasonal period of `52`.

These differencing orders emerged from the parameter-search process. However, the ADF and KPSS evidence suggested that the original weekly series did not require especially aggressive non-seasonal differencing. The model’s weak hold-out performance supports the possibility that differencing twice removed useful level information. A future implementation should compare `d=0`, `d=1` and `d=2` using rolling validation, while reserving the final two years for one unbiased evaluation.

## What Was Known at the Forecast Date

The information boundary was protected when the feature table was constructed. Load lags and rolling summaries were calculated only after shifting the load series, ensuring that the target week could not become one of its own predictors. The feature `temperature_lag1` was also created by shifting temperature by one week, making it historical and available at the forecasting date.

The models nevertheless retained observed same-week temperature and a temperature-change feature derived from current and previous temperature. These variables do not leak the target electricity load, but their observed future values would not be available during a real forecast. They would need to be supplied by an external weather forecast. Consequently, load lags, rolling features, calendar variables, holidays and lagged temperature are operationally available, whereas results involving observed same-week temperature must be interpreted as conditional.

## Whether Weather and Holidays Added Forecasting Value

Temperature and holiday covariates did not improve SARIMAX accuracy in this experiment. Temperature-and-holiday SARIMAX produced RMSE 5633.88 and MAPE 8.92%, compared with RMSE 3006.76 and MAPE 4.41% for Seasonal Naive. This does not demonstrate that weather and holidays are irrelevant. Instead, it suggests that the selected SARIMAX structure and a single Berlin temperature series did not represent national electricity demand sufficiently well.

Holiday dates are known in advance and can therefore be used safely for operational forecasting. One-week lagged temperature is also known. Future measured temperature is different because it is unavailable at the forecast origin. Unless a genuine weather forecast is supplied, SARIMAX results based on contemporaneous temperature should be described as explanatory or conditional forecasts.

## Interpretability and Modelling Complexity

SARIMAX was the easiest advanced model to explain because its autoregressive, moving-average, differencing, seasonal and exogenous components had identifiable statistical roles. It also generated prediction intervals that could be used to communicate uncertainty. Its main weakness in this study was forecast accuracy.

Random Forest occupied the middle of the interpretability spectrum. An individual prediction was less transparent than a SARIMAX equation, but feature-importance values identified which inputs contributed most strongly. It could also represent non-linear relationships without requiring the same stationarity assumptions as SARIMA or SARIMAX.

LSTM was the most complex and least transparent model. It could learn detailed hourly and longer-range temporal patterns, but required scaling, sequence construction, architectural tuning and greater computational monitoring. Its hidden-state representation also made individual forecasts harder to explain to a non-technical user.

## Operational Recommendation

Random Forest provides the most practical model for weekly operational forecasting in this version. It produced the best weekly accuracy, improved substantially over Seasonal Naive, remained easier to retrain and maintain than the LSTM, and offered feature-importance evidence that could be communicated to users.

Its principal limitation is uncertainty because it does not automatically produce SARIMAX-style confidence intervals. Bootstrap, quantile or conformal prediction intervals should therefore be added before deployment. If contemporaneous temperature remains an input, it must be supplied by a genuine weather forecast. Otherwise, the operational version should rely on lagged demand, rolling history, calendar information, known holidays and lagged weather.

Seasonal Naive should remain available as a monitoring benchmark and emergency fallback. It is inexpensive, stable and simple to update, making it useful for detecting whether the deployed model continues to provide meaningful forecasting value.

The report discusses the modelling choices and literature in more detail. Key references include standard time-series forecasting material, SARIMA/SARIMAX methods, Random Forest/feature-based forecasting ideas and LSTM literature for sequential demand forecasting.

Useful source links:

- Open Power System Data time-series file: https://data.open-power-system-data.org/time_series/2020-10-06/time_series_60min_singleindex.csv
- Open-Meteo historical weather API: https://archive-api.open-meteo.com/v1/archive
- Assignment report: `Trinath Chandra Bose Bolla.pdf`
