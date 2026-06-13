# GridLock Traffic Demand Prediction

## Overview

This repository contains the solution developed for the **GridLock Traffic Demand Prediction Challenge**.

The objective is to predict the **traffic demand** for each row in `test.csv` and generate a submission file matching the required format:

```text
Index,demand
```

The final solution relies exclusively on the official competition datasets and combines extensive feature engineering, target encoding, train-only lag features, hyperparameter optimization, and model blending.

---

## Dataset

### Files Used

| File                    | Purpose                          |
| ----------------------- | -------------------------------- |
| `train.csv`             | Primary labeled training dataset |
| `test.csv`              | Prediction dataset               |
| `sample_submission.csv` | Submission format reference      |

### Important Note

The auxiliary `training.csv` dataset was **not used** in the final modeling pipeline.

---

## Data Cleaning

### Categorical Standardization

Standardized categorical values for:

* geohash
* timestamp
* RoadType
* LargeVehicles
* Landmarks
* Weather

### Missing Value Handling

#### Categorical Features

Missing values replaced with:

```python
"Missing"
```

#### Numerical Features

* Median imputation
* Explicit missing indicators

Example:

```text
Temperature_missing
```

### Prediction Constraints

Final predictions are clipped to:

```text
[0, 1]
```

to satisfy demand bounds.

---

## Feature Engineering

### Time Features

Derived from timestamp:

* timestamp_minutes
* hour
* minute
* 15-minute slot index
* is_peak_commute
* is_late_night
* is_business_hour

### Cyclic Time Encoding

* time_sin
* time_cos
* slot_sin
* slot_cos

These help models understand daily periodicity.

---

### Day Features

Generated from day information:

* day
* day_mod7
* is_day49

---

### Temperature Features

* Temperature
* Temperature_missing
* Temperature_clip
* Temperature_sq
* Temperature_x_hour

Used to capture nonlinear weather effects.

---

### Road & Traffic Features

* RoadType
* NumberofLanes
* LargeVehicles
* Landmarks
* Weather

Interaction features:

* large_allowed
* has_landmark
* lanes_x_hour
* lanes_x_peak
* road_lanes
* weather_road

---

### Missingness Features

Separate indicators for:

* Temperature_missing
* RoadType_missing
* LargeVehicles_missing
* Landmarks_missing
* Weather_missing

---

### Geohash Features

Location was one of the strongest predictive signals.

Generated features:

* geohash_prefix3
* geohash_prefix4
* geohash_prefix5
* geohash character encodings
* geohash integer representation
* decoded latitude
* decoded longitude
* latitude error
* longitude error
* geo_lat_x_lon

---

### Frequency Encoding

Applied to:

* geohash
* geohash prefixes
* timestamp
* RoadType
* Weather
* road_lanes

Frequency encoding provides information about category prevalence.

---

### Target Encoding

Out-of-fold target encoding with smoothing was applied to:

* geohash
* geohash prefixes
* timestamp
* hour
* slot
* RoadType
* Weather
* LargeVehicles
* Landmarks
* road_lanes
* weather_road
* geo_time_key
* prefix5_time_key
* geo_hour_key
* road_weather_time_key

---

### Train-Only Lag Features

Historical demand features were created using only information available in `train.csv`.

Examples:

* Previous-day same geohash demand
* Previous-day geohash mean demand
* Previous-day timestamp mean demand
* Previous-day geohash-hour demand
* Previous-day prefix5-time demand

These features capture recurring traffic patterns while avoiding future leakage.

---

## Validation Strategy

A forward-looking validation scheme was used.

### Training Split

* Day 48
* Early Day 49 observations before 01:30

### Validation Split

* Day 49
* Timestamps:

  * 01:30
  * 01:45
  * 02:00

### Evaluation Metrics

Primary:

* RMSE

Secondary:

* MAE
* R²

---

## Models Evaluated

### Tree-Based Models

* LightGBM Regressor
* XGBoost Regressor
* HistGradientBoosting Regressor
* ExtraTrees Regressor
* RandomForest Regressor

### Linear Models

* Ridge Regression

### Ensemble

* Optuna-optimized weighted blend

---

## Hyperparameter Optimization

All major models were tuned using **Optuna**.

Optimized parameters included:

### LightGBM

* n_estimators
* learning_rate
* num_leaves
* max_depth
* min_child_samples
* subsample
* colsample_bytree
* reg_alpha
* reg_lambda

### XGBoost

* n_estimators
* learning_rate
* max_depth
* min_child_weight
* subsample
* colsample_bytree
* reg_alpha
* reg_lambda

### Tree Ensembles

* max_depth
* min_samples_leaf
* min_samples_split
* max_features
* n_estimators

### Ridge Regression

* alpha
* fit_intercept

---

## Validation Results

| Model                 | RMSE     | MAE      | R²       |
| --------------------- | -------- | -------- | -------- |
| Optuna Weighted Blend | 0.035319 | 0.023369 | 0.939642 |
| XGBoost               | 0.036184 | 0.023969 | 0.936652 |
| LightGBM              | 0.036251 | 0.023456 | 0.936417 |
| ExtraTrees            | 0.038303 | 0.024516 | 0.929014 |
| HistGradientBoosting  | 0.038931 | 0.024288 | 0.926665 |
| RandomForest          | 0.039089 | 0.024238 | 0.926070 |
| Ridge Regression      | 0.042913 | 0.029738 | 0.910899 |

---

## Final Model

### Optuna Weighted Blend

Weights:

| Model                | Weight   |
| -------------------- | -------- |
| XGBoost              | 0.479918 |
| LightGBM             | 0.391556 |
| Ridge                | 0.091101 |
| HistGradientBoosting | 0.016789 |
| ExtraTrees           | 0.011049 |
| RandomForest         | 0.009587 |

This ensemble achieved the best validation performance and was selected for final submission.

---

## Tech Stack

### Core Libraries

* pandas
* numpy
* scikit-learn
* Optuna
* LightGBM
* XGBoost
* joblib

### Development Environment

* Python
* Jupyter Notebook

---

## Submission

Final output file:

```text
predictions.csv
```

Submission format:

```csv
Index,demand
```

Validation checks:

* Correct column names
* Correct row count
* Matching test index order
* No missing predictions
* Predictions clipped to valid range

---

## Project Structure

```text
├── train.csv
├── test.csv
├── sample_submission.csv
├── GridLock_train_only_submission.ipynb
├── predictions.csv
├── README.md
```

---

## Results

The solution demonstrates that strong feature engineering, careful train-only validation, target encoding, lag feature construction, and model blending can significantly improve traffic demand forecasting performance.

Final selected approach:

✅ Advanced feature engineering
✅ Train-only lag features
✅ Out-of-fold target encoding
✅ Optuna hyperparameter tuning
✅ Ensemble blending
