# E-Waste Collection Demand Forecasting and Zone Optimization Using R

## 1. Project Overview

E-waste is one of the fastest-growing waste streams and requires efficient collection, recycling, and resource planning.

This project develops a data-driven approach for:

1. Analyzing historical e-waste generation and recycling.
2. Forecasting future e-waste generation.
3. Comparing ARIMA and Random Forest forecasting models.
4. Evaluating models using MAE, RMSE, and MAPE.
5. Analyzing ward-level population and household data.
6. Grouping wards into collection zones using K-Means clustering.
7. Supporting data-driven e-waste collection planning.

The project is implemented using **R** and uses real-world government datasets.

---

## 2. Objectives

The main objectives of the project are:

- Analyze historical e-waste generation and recycling trends.
- Calculate the percentage of e-waste recycled.
- Forecast future e-waste generation using ARIMA.
- Develop a Random Forest regression model for comparison.
- Compare ARIMA and Random Forest using:
  - Mean Absolute Error (MAE)
  - Root Mean Squared Error (RMSE)
  - Mean Absolute Percentage Error (MAPE)
- Analyze population and household distribution across wards.
- Group wards into three collection zones using K-Means clustering.
- Provide a data-driven approach for e-waste collection planning.

---

## 3. Project Workflow

```text
                    REAL-WORLD DATA
                          |
             +------------+------------+
             |                         |
             v                         v
      E-WASTE DATA               CENSUS DATA
             |                         |
             v                         v
       DATA CLEANING            WARD DATA EXTRACTION
             |                         |
             v                         v
     E-WASTE ANALYSIS        POPULATION + HOUSEHOLDS
             |                         |
             v                         v
       TIME SERIES              DATA STANDARDIZATION
             |                         |
             v                         v
       +-----------+              K-MEANS
       |           |                 |
       v           v                 v
     ARIMA    RANDOM FOREST     3 COLLECTION ZONES
       |           |
       +-----+-----+
             |
             v
      MODEL EVALUATION
             |
      MAE / RMSE / MAPE
             |
             v
       MODEL COMPARISON
             |
             v
      COLLECTION PLANNING
