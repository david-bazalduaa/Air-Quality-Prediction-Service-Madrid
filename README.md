# Air Quality Prediction System - Madrid, Spain
### ID2223 Scalable Machine Learning - Lab 1

This project implements a scalable, serverless machine learning pipeline to predict air quality ($PM_{2.5}$) for the whole city of Madrid, consisting of 7 different sensors. The system is fully automated using **Hopsworks Feature Store**, **GitHub Actions**, and **XGBoost**.

🔗 **[Click here to view the Live Dashboard](INSERT_YOUR_GITHUB_PAGES_URL_HERE)**

---

## 📋 Project Overview

The goal of this project is to predict $PM_{2.5}$ levels for the next 6-7 days. It uses a feature store architecture to manage data, trains an XGBoost model on historical data, then test the model on recent data and finally performs batch inference daily. Addresses the **Grade A** requirement by scaling predictions to a cluster of sensors (Madrid city level) and implementing advanced time-series feature engineering.

**Key Technical Features:**
* [cite_start]**Multi-Sensor Architecture:** Aggregates and processes data from 7 distinct locations in Madrid[cite: 427].
* [cite_start]**Advanced Feature Engineering:** Implements sliding windows and lag features (`pm25_lag1`, `pm25_lag2`, `pm25_lag3`, `pm25_roll3`) to capture temporal dependencies[cite: 423].
* **Recursive Forecasting:** The inference pipeline uses a row-by-row prediction strategy to dynamically update lag features for future days.
* **Automated Pipelines:** Daily runs via GitHub Actions for continuous operation.

---

## 🛠 Pipeline Architecture

The system is structured into 4 independent Python notebooks, managed through the Hopsworks Feature Store.

### 1. Feature Backfill (`1_air_quality_feature_backfill.ipynb`)
* **Data Ingestion:** Downloads historical data for Air Quality ($PM_{2.5}$) from .csv files retrieved from [AQICN](https://aqicn.org) and Weather from [Open-Meteo](https://open-meteo.com) for all 7 Madrid sensors.
* **Feature Engineering:**
    * **Air Quality Base Features:** ($PM_{2.5}$) value, sensor ID, city, country and street.
    * * **Weather Base Features:** Temperature, precipitation, wind speed, wind direction, cloud cover, relative humidity and dew point.
    * **Lags:** Calculated `pm25_lag1`, `pm25_lag2`, `pm25_lag3` to use previous days' pollution as predictors.
    * **Rolling Window:** Calculated a rolling mean for trend smoothing.
    * **Temporal:** Added a binary `is_weekend` feature, apart from the date.
* **Storage:** All air quality data is cleaned, engineered, and aggregated by sensor into a single Pandas DataFrame before being registered as a **Feature Group in Hopsworks**. We create a second **Feature Group in Hopsworks** with the weather data. 

### 2. Daily Feature Pipeline (`2_air_quality_feature_pipeline.ipynb`)
* **Automation:** Scheduled to run daily via **GitHub Actions**.
* **Logic:** Retrieves the most recent daily data, applies the same feature engineering transformations (lags, rolling means, etc.), and updates the Feature Groups in Hopsworks with the new observations.

### 3. Training Pipeline (`3_air_quality_training_pipeline.ipynb`)
* **Dataset:** Creates a **Feature View** joining air quality features with weather data.
* **Model:** Trains and evaluates a **XGBoost Regressor** to predict the future ($PM_{2.5}$) values.
* **Features Immportance:** Calculate the most influential features for the model based on F score.
* **Registry:** Evaluates the model and registers the best-performing version to the **Hopsworks Model Registry**.

### 4. Batch Inference Pipeline (`4_air_quality_batch_inference.ipynb`)
* **Automation:** Scheduled to run daily via **GitHub Actions**.
* **Forecasting Logic:**
    1.  Retrieves the trained model and weather forecasts for the next 6-7 days.
    2.  **Recursive Prediction:** Since the model depends on `lag` features (e.g., yesterday's value), predictions are made **row-by-row** in a loop.
    3.  After predicting day $t$, the result is fed back as the `lag_1` input for day $t+1$, ensuring consistent time-series forecasting.
* **Visualization:** Generates the plots for the dashboard and pushes them to the repository for GitHub Pages.

---

## 📊 Dashboard

The predictions are visualized on a public dashboard hosted on GitHub Pages. It includes:
1.  **Hindcast Graphs:** Comparing past predictions against actual measurements to monitor accuracy.
2.  **7-Day Forecast:** Predicted Air Quality levels for all 7 sensors in Madrid.

---

## 🚀 How to Run

1.  **Clone the repository:**
    ```bash
    git clone [https://github.com/YOUR_USERNAME/YOUR_REPO_NAME.git](https://github.com/YOUR_USERNAME/YOUR_REPO_NAME.git)
    ```
2.  **Install dependencies:**
    ```bash
    pip install -r requirements.txt
    ```
3.  **Set up Secrets:**
    Ensure you have your `HOPSWORKS_API_KEY` configured in your environment or GitHub Secrets.

---
