# BlueBlikeML
Model to predict the hourly demand/usage of the Boston Blue Bikes
# Blue Bikes Hourly Demand Forecasting

A complete walkthrough of how I built, evaluated, and tuned a machine-learning pipeline to predict hourly bike-share demand for Blue Bikes.

---

## Overview

The objective of this project was to forecast the number of trips taken on Blue Bikes in each hour of the day, using historical trip data. Accurate hourly forecasts can inform bike rebalancing, staffing, and maintenance schedules.

---

## Data Preparation

1. **Raw Data Ingestion**  
   - Loaded a CSV containing every individual trip record (start time, end time, rideable type, stations). This repo does not provide you with the data necessary to train your own model. Please visit [this link](https://s3.amazonaws.com/hubway-data/index.html) to find more Blue Bikes data.
2. **Feature Engineering**  
   - Extracted calendar features:  
     - **Date**  
     - **Hour of day**  
     - **Day of week**  
     - **Weekend flag**  
     - **Holiday flag**  
   - Aggregated at the (date, hour) level to compute `total_trips`.  
   
   _Figure 1: Time-series plot of hourly trip counts over the full dataset._

3. **Train/Test Split**  
   - Sorted data chronologically and reserved the last 30 days as a hold-out test set.  
   - All preceding hours formed the training set.

---

## Baseline Model: Linear Regression

- **Approach:**  
  Fitted a simple linear regression using only the four calendar features (`hour`, `dayofweek`, `is_weekend`, `is_holiday`).  
- **Result:**  
  - MAE and RMSE were high, and the model failed to capture clear morning/evening peaks or weekend behavior.  
- **Insight:**  
  A straight-line model was too rigid to reflect the non-linear patterns in bike usage.

_Figure 2: Scatterplot of actual vs. predicted trips under the linear regression baseline._

---

## Model Exploration & Leaderboard

To identify a stronger algorithm, I automatically evaluated every regressor in scikit-learn using time-series cross-validation:

- **3-fold TimeSeriesSplit**  
- **Evaluation metrics:** MAE and RMSE  

This produced a ranked “leaderboard” of the top 10 models:

| Rank | Model                         | MAE    | RMSE   |
|:----:|:-----------------------------:|-------:|-------:|
| 1    | RadiusNeighborsRegressor      | 208.21 | 334.59 |
| 2    | KNeighborsRegressor           | 215.33 | 346.70 |
| 3    | GaussianProcessRegressor      | 230.50 | 384.38 |
| …    | …                             | …      | …      |
| 10   | GradientBoostingRegressor     | 240.59 | 367.05 |

_**Winner:**_ **RadiusNeighborsRegressor** achieved the lowest MAE by averaging demand among “nearby” feature-space neighbors.

_Table 1: Top 10 regressors by cross-validated MAE._

---

## Hyperparameter Tuning

Focused on the Radius Neighbors model to squeeze out further gains:

- Wrapped in a pipeline with feature scaling.  
- Grid-searched over:  
  - **radius** (0.5 → 10.0)  
  - **weights** (uniform vs. distance)  
  - **distance metric** (Euclidean vs. Manhattan)  
  - **leaf_size** (20, 30, 40)  

**Best parameters found:**  
- radius = 5.0  
- weights = distance  
- metric = manhattan  
- leaf_size = 20  

_Figure 3: Heatmap of MAE across radius and metric combinations._

---

## Final Evaluation

Applied the tuned model to the 30-day test set:

- **Test MAE:** 184.76 trips/hour  
- **Test RMSE:** _(insert RMSE)_  

_Figure 4: Time-series overlay of actual vs. predicted demand on the test set._  
_Figure 5: Scatterplot of actual vs. predicted trips with best-fit and y=x reference._

---

## Next Steps

- **Feature Enrichment:** Incorporate weather, special events, and station-specific metadata.  
- **Advanced Models:** Explore ARIMA/Prophet hybrids or deep sequence models (LSTM, TCN, Transformers).  
- **Productionization:** Package the model into a Flask API or Docker container for real-time forecasting.

---

**Conclusion:**  
By iterating from a simple linear baseline to automated model selection and careful hyperparameter tuning, I reduced MAE and produced a reliable hourly demand forecast that can directly support Blue Bikes operations.
