# E-Waste Collection Demand Forecasting and Zone Optimization Using R

> **Project documentation based on the uploaded notebook:** `E_Waste_Forecsting_(2).ipynb`  
> **Language:** R   |  **Environment used by the notebook:** Google Colab-style `/content` paths   |  **Main techniques:** exploratory data analysis, time-series forecasting, Random Forest regression, and K-Means clustering

## 1. Project introduction

Electronic waste (e-waste) is discarded electrical and electronic equipment, such as computers, mobile phones, televisions, chargers, and household appliances. When e-waste is not collected and processed properly, useful recoverable materials may be lost and hazardous components can be handled unsafely. A collection system therefore needs to understand how much e-waste is being generated, how much is being recycled, and which areas may require more collection attention.

This project, **E-Waste Collection Demand Forecasting and Zone Optimization Using R**, analyses historical e-waste generation and recycling figures for India and explores how demographic information can be used to group Hyderabad municipal wards into collection-planning zones.

The notebook has two connected parts:

1. **National-level trend and forecasting analysis:** examines recorded e-waste generation and recycling quantities, calculates recycling percentages, forecasts future generation using ARIMA, builds a Random Forest model for comparison, and evaluates predictions using MAE, RMSE, and MAPE.
2. **Ward-level collection-zone analysis:** reads a census workbook, extracts ward population and household counts, and applies K-Means clustering to group wards with similar demographic characteristics.

The output is an analytical prototype. It gives a preliminary view of national e-waste trends and demographic collection zones; it does **not** directly measure the e-waste produced by each ward, calculate collection routes, or prove that the resulting groups are operationally optimal.

## 2. Problem statement

E-waste generation changes over time, while collection resources such as vehicles, staff, bins, and collection centres are limited. If planners do not monitor generation and recycling trends, or do not account for differences in the population and households of local areas, collection capacity may not be distributed efficiently.

The project addresses the following questions:

- How did recorded e-waste generation and recycling change across the financial years included in the main dataset?
- What percentage of recorded generated e-waste was recorded as recycled each year?
- What do ARIMA and Random Forest predict, and which performs better under the notebook's test setup?
- How can wards be grouped using population and household counts as preliminary proxies for collection demand?
- What tables and plots can help communicate the findings to a reviewer or a collection-planning team?

## 3. Aim

To analyse recorded e-waste generation and recycling trends, compare two forecasting approaches, and explore demographic clustering of Hyderabad wards to support preliminary e-waste collection planning.

## 4. Objectives

1. Load and clean the main e-waste dataset.
2. Compare annual generation with annual recycling quantities.
3. Calculate annual recycling percentages.
4. Visualize yearly e-waste generation and convert the data into an R time-series object.
5. Fit an ARIMA model and forecast two subsequent annual observations.
6. Fit a Random Forest regression model and compare test predictions with ARIMA predictions.
7. Evaluate predictions with Mean Absolute Error (MAE), Root Mean Squared Error (RMSE), and Mean Absolute Percentage Error (MAPE).
8. Plot an additional e-waste dataset that includes a partial 2025-26 year entry.
9. Extract ward population and household counts from the census workbook.
10. Use K-Means with three clusters to group wards by standardized population and household counts.
11. Export plots and result tables into an output directory.

## 5. Overall workflow

```text
Start
  →
Install R packages and load libraries
  →
Check uploaded files and create output folders
  →
Load main e-waste dataset (generation + recycling)
  →
Rename columns, convert numeric fields, remove missing generation rows
  →
Explore generation, recycling, and recycling percentage
  →
Create annual trend and time-series object
  →
Fit ARIMA → forecast next 2 observations
  →
Split historical data → train Random Forest and create test predictions
  →
Evaluate models using MAE, RMSE, MAPE
  →
Plot and inspect the additional e-waste dataset
  →
Load census workbook → filter ward records → clean population/households
  →
Plot ward population → standardize demographic variables
  →
Run K-Means (3 clusters) → summarize and visualize zones
  →
Print final results and save outputs
  →
End
```

## 6. Datasets used by the notebook

The file paths below are the paths used in the notebook. The notebook itself does not document full source URLs or detailed provenance for these uploaded files, so their original source should be verified separately before the results are used in a formal report.

| File | Role in the notebook | Data visible in the recorded output |
|---|---|---|
| `RS_Session_269_AU_525_A.csv` | Main generation and recycling dataset | 5 financial-year rows, with generation and recycling quantities |
| `RS_Session_270_AS_140_A_i.csv` | Additional recent generation trend | 3 rows: 2023-24, 2024-25, and 2025-26 (so far) |
| `DDW_PCA2805_2011_MDDS with UI.xlsx` | Demographic/census data; sheet `EB-2805` | 191 rows and 94 columns before filtering; 108 ward records selected |
| `RS_Session_269_AU_525_1.csv` | Visible in the file listing | Listed by the notebook, but not used by the shown analysis cells |

### 6.1 Main e-waste dataset

The notebook reads `RS_Session_269_AU_525_A.csv` and identifies three columns:

- `Financial Year (FY)`
- `E-Waste Generation - Metric Ton (MT)`
- `E-Waste Recycled (MT)`

The data covers five financial years, from 2020-21 through 2024-25. The notebook uses this five-row dataset for the main generation/recycling analysis and for the forecasting experiment.

### 6.2 Additional e-waste dataset

The additional CSV contains:

- `Year`
- `E-Waste Generation (Metric Tonnes)`
- `Percentage increase of e-waste generation over previous year`

It includes an entry marked **2025-26 (so far)**. This is a partial-period record according to its label, not a completed financial-year total. The notebook visualizes this file separately; it does not combine it into the five-observation time series used to fit and evaluate the models.

### 6.3 Census/ward dataset

The workbook `DDW_PCA2805_2011_MDDS with UI.xlsx` is read from the `EB-2805` sheet. Its filename indicates a 2011 census dataset. The notebook reports 191 rows and 94 columns, with administrative levels comprising:

| Level | Number of records |
|---|---:|
| DISTRICT | 3 |
| SUB-DISTRICT | 48 |
| TOWN | 32 |
| WARD | 108 |

The analysis filters records where `Level == "WARD"`, then selects identifying fields and household/population-related columns. The final clustering uses only **Population** and **Households**.

## 7. Tools and R packages

The notebook installs and loads the following packages:

| Package | Purpose in this notebook |
|---|---|
| `readr` | Read CSV files with `read_csv()` |
| `readxl` | Read the Excel census workbook with `read_excel()` |
| `dplyr` | Filter, select, rename/derive fields, and summarize data |
| `tidyr` | Reshape the generation/recycling data for plotting |
| `ggplot2` | Create charts and save figures |
| `forecast` | Fit ARIMA models and generate forecasts |
| `randomForest` | Train the Random Forest regression model |
| `cluster` | Loaded by the notebook; the shown clustering call uses base R's `kmeans()` |
| `scales` | Loaded by the notebook; not central to the visible calculations |

### Setup code

```r
install.packages(c(
  "readr", "readxl", "dplyr", "tidyr", "ggplot2",
  "forecast", "randomForest", "cluster", "scales"
), repos = "https://cloud.r-project.org")

library(readr)
library(readxl)
library(dplyr)
library(tidyr)
library(ggplot2)
library(forecast)
library(randomForest)
library(cluster)
library(scales)

cat("All libraries loaded successfully!\n")
```

**Recorded output:** `All libraries loaded successfully!` (along with ordinary package attachment messages).

The notebook then checks the files in `/content` and creates these folders:

```r
list.files("/content")

dir.create("/content/EWaste_Case_Study", showWarnings = FALSE)
dir.create("/content/EWaste_Case_Study/outputs", showWarnings = FALSE)
dir.create("/content/EWaste_Case_Study/data", showWarnings = FALSE)
```

**Recorded file listing:** the three CSV/XLSX data files named above plus a `sample_data` directory. The notebook prints `Project folders created.` after creating the folders.

## 8. Main e-waste analysis: loading and cleaning

### 8.1 Read the CSV

```r
ewaste_data <- read_csv(
  "/content/RS_Session_269_AU_525_A.csv",
  show_col_types = FALSE
)

print(names(ewaste_data))
print(ewaste_data)
```

The notebook prints the original column names and all five data rows.

### 8.2 Rename columns and clean numeric fields

```r
ewaste_clean <- ewaste_data

names(ewaste_clean)[1:3] <- c("Year", "Generated", "Recycled")
ewaste_clean$Generated <- as.numeric(ewaste_clean$Generated)
ewaste_clean$Recycled <- as.numeric(ewaste_clean$Recycled)

ewaste_clean <- ewaste_clean %>%
  filter(!is.na(Generated))

print(ewaste_clean)
cat("\nRows:", nrow(ewaste_clean), "\n")
cat("Columns:", ncol(ewaste_clean), "\n")
```

**Why this step is used:**

- Shorter field names make the later R expressions easier to read.
- `as.numeric()` ensures the generation and recycling values are numeric for calculations and plotting.
- `filter(!is.na(Generated))` removes rows without a generation value.
- `nrow()` and `ncol()` confirm the size of the cleaned table.

**Recorded output:** the cleaned table contains **5 rows and 3 columns**. No row was removed by the missing-generation filter in the recorded output.

### 8.3 Cleaned main dataset output

| Financial year (`Year`) | Generated (MT) | Recycled (MT) |
|---|---:|---:|
| 2020-21 | 1,346,496 | 354,541 |
| 2021-22 | 1,601,155 | 527,132 |
| 2022-23 | 1,609,117 | 539,256 |
| 2023-24 | 1,254,287 | 778,205 |
| 2024-25 | 1,397,956 | 1,159,228 |

The most important visible trend is that generation rose between 2020-21 and 2022-23, declined in 2023-24, and increased again in 2024-25. The recorded recycling quantity increased across the five rows.

## 9. Generation versus recycling visualization

The notebook reshapes `Generated` and `Recycled` into a long-format table so that both quantities can be shown as side-by-side bars for each financial year.

```r
ewaste_long <- ewaste_clean %>%
  pivot_longer(
    cols = c(Generated, Recycled),
    names_to = "Category",
    values_to = "Quantity"
  )

p1 <- ggplot(ewaste_long,
             aes(x = Year, y = Quantity, fill = Category)) +
  geom_col(position = "dodge") +
  labs(
    title = "E-Waste Generation vs Recycling in India",
    x = "Financial Year",
    y = "Quantity (Metric Tonnes)",
    fill = "Category"
  ) +
  theme_minimal() +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))

print(p1)
ggsave("/content/EWaste_Case_Study/outputs/01_ewaste_generation_vs_recycling.png",
       p1, width = 9, height = 6, dpi = 300)
```

`pivot_longer()` converts two quantity columns into a category column and one value column. `geom_col(position = "dodge")` places the generated and recycled bars beside each other. The x-axis labels are rotated to improve readability.

**Recorded plot:** `01_ewaste_generation_vs_recycling.png`.

![E-waste generation versus recycling plot](E_Waste_Project_Guide_assets/01_ewaste_generation_vs_recycling.png)

**Interpretation:** The bars make the gap between generated and recycled quantities visible. In the last year included in the main dataset, the recorded recycled quantity is much closer to the generated quantity than it was in 2020-21. This chart compares the two reported values; it does not establish what happened to any unaccounted-for waste.

## 10. Recycling percentage calculation

The notebook calculates recycling percentage as:

\[
\text{Recycling Percentage} = \frac{\text{Recycled}}{\text{Generated}} \times 100
\]

The corresponding R code is:

```r
ewaste_percentage <- ewaste_clean %>%
  mutate(Recycling_Percentage = (Recycled / Generated) * 100)

print(ewaste_percentage %>%
        select(Year, Generated, Recycled, Recycling_Percentage))
```

### Recorded output

| Financial year | Generated (MT) | Recycled (MT) | Recycling percentage |
|---|---:|---:|---:|
| 2020-21 | 1,346,496 | 354,541 | 26.3% |
| 2021-22 | 1,601,155 | 527,132 | 32.9% |
| 2022-23 | 1,609,117 | 539,256 | 33.5% |
| 2023-24 | 1,254,287 | 778,205 | 62.0% |
| 2024-25 | 1,397,956 | 1,159,228 | 82.9% |

The percentage rises from **26.3%** in 2020-21 to **82.9%** in 2024-25 in the supplied data. This is the ratio of the two columns as supplied, rather than a separately verified national recycling rate.

### Recycling percentage plot

The notebook plots one line and one point per year, with the calculated percentage on the y-axis.

![E-waste recycling percentage plot](E_Waste_Project_Guide_assets/02_ewaste_recycling_percentage.png)

**Recorded plot file:** `02_ewaste_recycling_percentage.png`.

## 11. Annual trend and time-series preparation

The code extracts the first four characters of the financial-year label to create a numeric year value. For example, `2020-21` becomes `2020`.

```r
annual_data <- ewaste_clean %>%
  mutate(Year_Num = as.numeric(substr(as.character(Year), 1, 4))) %>%
  arrange(Year_Num)

p3 <- ggplot(annual_data, aes(x = Year_Num, y = Generated)) +
  geom_line(linewidth = 1) +
  geom_point(size = 3) +
  labs(
    title = "Annual E-Waste Generation Trend",
    x = "Year",
    y = "E-Waste Generated (MT)"
  ) +
  theme_minimal()

print(p3)
ggsave("/content/EWaste_Case_Study/outputs/03_annual_ewaste_trend.png",
       p3, width = 9, height = 6, dpi = 300)
```

**Recorded output:** `annual_data` has five rows and four columns: `Year`, `Generated`, `Recycled`, and `Year_Num` (2020 through 2024).

![Annual e-waste generation trend](E_Waste_Project_Guide_assets/03_annual_ewaste_trend.png)

The data is then converted into a time-series object:

```r
ewaste_ts <- ts(
  annual_data$Generated,
  start = min(annual_data$Year_Num),
  frequency = 1
)

print(ewaste_ts)
```

**Recorded time-series output:**

```text
Time Series:
Start = 2020
End = 2024
Frequency = 1
[1] 1346496 1601155 1609117 1254287 1397956
```

`frequency = 1` represents one observation per year. The notebook also saves a basic time-series plot as `04_ewaste_time_series.png`.

![Annual e-waste time-series plot](E_Waste_Project_Guide_assets/04_ewaste_time_series.png)

## 12. ARIMA forecasting

### 12.1 What ARIMA does here

ARIMA stands for **AutoRegressive Integrated Moving Average**. It is a time-series forecasting method. In this notebook, `auto.arima()` selects a model from the five annual generation observations, and `forecast()` estimates the next two observations.

### 12.2 Fitting ARIMA and forecasting two steps ahead

```r
arima_model <- auto.arima(ewaste_ts, seasonal = FALSE)
print(arima_model)

arima_forecast <- forecast(arima_model, h = 2)
print(arima_forecast)

p5 <- autoplot(arima_forecast) +
  labs(
    title = "ARIMA Forecast of E-Waste Generation",
    x = "Year",
    y = "E-Waste Generation (MT)"
  ) +
  theme_minimal()

print(p5)
ggsave("/content/EWaste_Case_Study/outputs/05_arima_forecast.png",
       p5, width = 9, height = 6, dpi = 300)
```

`seasonal = FALSE` tells the model not to fit seasonal ARIMA components. `h = 2` requests two future observations.

### 12.3 Recorded ARIMA model output

```text
Series: ewaste_ts
ARIMA(0,0,0) with non-zero mean

Coefficients:
           mean
      1441802.2
s.e.    63105.9

sigma^2 = 2.489e+10:  log likelihood = -66.38
AIC=136.76   AICc=142.76   BIC=135.98
```

The selected model is **ARIMA(0,0,0) with a non-zero mean**. In this fitted model, the point forecast is a constant mean of approximately **1,441,802 metric tonnes** for each future step. This does not mean the model has learned a detailed rising or falling trend; with only five observations, `auto.arima()` selected a very simple model.

### 12.4 Recorded forecast output

| Forecast year shown by R | Point forecast (MT) | 80% lower | 80% upper | 95% lower | 95% upper |
|---|---:|---:|---:|---:|---:|
| 2025 | 1,441,802 | 1,239,622 | 1,643,982 | 1,132,594 | 1,751,010 |
| 2026 | 1,441,802 | 1,239,622 | 1,643,982 | 1,132,594 | 1,751,010 |

The notebook creates a data frame with the forecast year, forecast point estimate, and 80% prediction interval, then saves it to `arima_forecast_values.csv`:

```r
forecast_values <- data.frame(
  Year = time(arima_forecast$mean),
  Forecast = as.numeric(arima_forecast$mean),
  Lower_80 = as.numeric(arima_forecast$lower[, "80%"]),
  Upper_80 = as.numeric(arima_forecast$upper[, "80%"])
)

print(forecast_values)
write.csv(forecast_values,
          "/content/EWaste_Case_Study/outputs/arima_forecast_values.csv",
          row.names = FALSE)
```

**Recorded saved-table values:**

| Year | Forecast | Lower_80 | Upper_80 |
|---:|---:|---:|---:|
| 2025 | 1,441,802 | 1,239,622 | 1,643,982 |
| 2026 | 1,441,802 | 1,239,622 | 1,643,982 |

![ARIMA forecast plot](E_Waste_Project_Guide_assets/05_arima_forecast.png)

**Important interpretation note:** These forecast years are the numeric time points generated from the time-series object. The input `Year` labels are financial years such as `2024-25`, but the code extracts the starting year only. A production version should explicitly map forecast years to financial-year labels and ensure that the forecast target matches the reporting period.

## 13. Random Forest forecasting experiment

### 13.1 Prepare the input table

The notebook selects the numeric year and generation amount:

```r
rf_data <- annual_data %>%
  select(Year_Num, Generated)

print(rf_data)
```

**Recorded data:** five rows, with years 2020-24 and their generation values.

### 13.2 Train/test split

The notebook uses the first three rows as training data and the last two as test data:

```r
n <- nrow(rf_data)

train_data <- rf_data[1:(n - 2), ]
test_data <- rf_data[(n - 1):n, ]

print(train_data)
print(test_data)
```

This results in:

- **Training years:** 2020, 2021, 2022 (3 observations)
- **Testing years:** 2023, 2024 (2 observations)

### 13.3 Fit Random Forest and predict the test rows

```r
set.seed(123)

rf_model <- randomForest(
  Generated ~ Year_Num,
  data = train_data,
  ntree = 500
)

print(rf_model)

rf_predictions <- predict(rf_model, newdata = test_data)
print(rf_predictions)
```

`ntree = 500` asks the model to build 500 trees. `set.seed(123)` makes the model's random choices reproducible in the same environment and with the same data/package behavior.

### 13.4 Recorded Random Forest output

The notebook prints this warning:

```text
The response has five or fewer unique values. Are you sure you want to do regression?
```

The warning is relevant because the training data contains only three response values. The recorded model summary is:

```text
Type of random forest: regression
Number of trees: 500
No. of variables tried at each split: 1
Mean of squared residuals: 39278226580
% Var explained: -164.04

Predictions:
1593749 1593749
```

Both test predictions are approximately **1,593,749 MT**. The negative training variance-explained result is a warning sign that this fit is not performing well. Using 500 trees does not compensate for having only three training examples.

## 14. ARIMA test predictions and model evaluation

### 14.1 ARIMA predictions on the same held-out years

To make a fairer comparison within this small experiment, the notebook fits a separate ARIMA model on the three training observations, then forecasts two steps for the 2023 and 2024 test years.

```r
arima_train_ts <- ts(
  train_data$Generated,
  start = min(train_data$Year_Num),
  frequency = 1
)

arima_test_model <- auto.arima(arima_train_ts, seasonal = FALSE)
arima_test_forecast <- forecast(arima_test_model, h = nrow(test_data))
arima_predictions <- as.numeric(arima_test_forecast$mean)

print(arima_predictions)
```

**Recorded ARIMA test predictions:**

```text
[1] 1518923 1518923
```

### 14.2 Evaluation metrics

The notebook calculates three common regression metrics.

**Mean Absolute Error (MAE)**

\[
\mathrm{MAE}=\frac{1}{n}\sum_{i=1}^{n}|y_i-\hat{y}_i|
\]

MAE is the average absolute difference between actual and predicted values. It is measured in the same unit as the target: metric tonnes.

**Root Mean Squared Error (RMSE)**

\[
\mathrm{RMSE}=\sqrt{\frac{1}{n}\sum_{i=1}^{n}(y_i-\hat{y}_i)^2}
\]

RMSE penalizes larger errors more heavily because the errors are squared before averaging. It is also measured in metric tonnes.

**Mean Absolute Percentage Error (MAPE)**

\[
\mathrm{MAPE}=\frac{100}{n}\sum_{i=1}^{n}\left|\frac{y_i-\hat{y}_i}{y_i}\right|
\]

MAPE expresses average absolute error as a percentage. The code excludes actual values equal to zero to avoid division by zero.

The functions implemented by the notebook are:

```r
calculate_mae <- function(actual, predicted) {
  mean(abs(actual - predicted))
}

calculate_rmse <- function(actual, predicted) {
  sqrt(mean((actual - predicted)^2))
}

calculate_mape <- function(actual, predicted) {
  valid <- actual != 0
  mean(abs((actual[valid] - predicted[valid]) / actual[valid])) * 100
}
```

### 14.3 Recorded model metrics

| Model | MAE (MT) | RMSE (MT) | MAPE |
|---|---:|---:|---:|
| **ARIMA** | **192,801.8** | **205,749.2** | **14.87586%** |
| Random Forest | 267,627.4 | 277,100.4 | 20.53490% |

Lower values indicate smaller errors under these metrics. In the notebook's two-row test set, **ARIMA performs better than Random Forest on MAE, RMSE, and MAPE**. The final project output names ARIMA as the best model based on RMSE.

This comparison is preliminary: the scores are calculated from only two test observations, so they should not be treated as reliable evidence that ARIMA will outperform Random Forest on new data generally.

### 14.4 Model comparison table and chart

The notebook stores the model results in `model_comparison` and writes `model_comparison.csv`. It then reshapes MAE, RMSE, and MAPE into a long format and uses facets with separate y-axis scales to compare them visually.

![ARIMA versus Random Forest comparison](E_Waste_Project_Guide_assets/06_model_comparison.png)

**Recorded plot file:** `06_model_comparison.png`.

### 14.5 Actual versus predicted values

The comparison table contains the held-out observations and both sets of predictions:

| Year | Actual generation (MT) | ARIMA prediction (MT) | Random Forest prediction (MT) |
|---:|---:|---:|---:|
| 2023 | 1,254,287 | 1,518,923 | 1,593,749 |
| 2024 | 1,397,956 | 1,518,923 | 1,593,749 |

The notebook saves this table as `prediction_comparison.csv` and plots all three series.

![Actual versus predicted generation](E_Waste_Project_Guide_assets/07_actual_vs_predicted.png)

The two models produce constant predictions across both test years in this run, while actual values differ between 2023 and 2024. The actual/predicted chart makes those deviations visible.

## 15. Additional recent e-waste dataset

The notebook loads `RS_Session_270_AS_140_A_i.csv`, prints its fields, renames them to `Year`, `Generated`, and `Percentage_Increase`, and converts the numeric columns to numeric types.

```r
additional_data <- read_csv(
  "/content/RS_Session_270_AS_140_A_i.csv",
  show_col_types = FALSE
)

additional_clean <- additional_data
names(additional_clean)[1:3] <- c(
  "Year", "Generated", "Percentage_Increase"
)

additional_clean$Generated <- as.numeric(additional_clean$Generated)
additional_clean$Percentage_Increase <- as.numeric(
  additional_clean$Percentage_Increase
)

print(additional_clean)
```

### Recorded additional dataset output

| Year | Generated (MT) | Percentage increase over previous year |
|---|---:|---:|
| 2023-24 | 1,254,287 | `NA` |
| 2024-25 | 1,397,956 | 11.4% |
| 2025-26 (so far) | 1,506,710 | 7.78% |

The first increase value is missing in this file and is shown as `NA`. The plot shows a rising value across these three entries.

![Additional e-waste generation trend](E_Waste_Project_Guide_assets/08_additional_ewaste_trend.png)

**Recorded plot file:** `08_additional_ewaste_trend.png`.

**Important:** The partial 2025-26 value is not used to retrain the ARIMA model or Random Forest model in the notebook. Because that record is marked “so far,” it should not be compared directly with completed financial-year totals without confirming the reporting period and data definitions.

## 16. Load and inspect the census workbook

The notebook reads the `EB-2805` worksheet:

```r
census_data <- read_excel(
  "/content/DDW_PCA2805_2011_MDDS with UI.xlsx",
  sheet = "EB-2805"
)

print(dim(census_data))
print(names(census_data))
print(table(census_data$Level, useNA = "ifany"))
```

**Recorded dimensions:** `191 94` (191 rows by 94 columns).

The code prints a list of census fields and the following counts by administrative level: 3 districts, 48 sub-districts, 32 towns, and 108 wards.

## 17. Extract and clean ward-level data

### 17.1 Filter ward records

```r
ward_data <- census_data %>%
  filter(Level == "WARD") %>%
  select(
    State, District, Subdistt, `Town/Village`, Ward, Name,
    No_HH, TOT_P, TOT_M, TOT_F
  )

print(dim(ward_data))
print(head(ward_data))
```

This selects identifying information, household count (`No_HH`), total population (`TOT_P`), and male/female population columns. The recorded dimensions are **108 rows by 10 columns**.

### 17.2 Convert population and household counts

```r
ward_data <- ward_data %>%
  mutate(
    Population = as.numeric(TOT_P),
    Households = as.numeric(No_HH)
  ) %>%
  filter(!is.na(Population), !is.na(Households))

summary(ward_data[c("Population", "Households")])
print(head(ward_data))
```

The two new fields are:

- `Population`: numeric version of total ward population.
- `Households`: numeric version of the number of households.

The notebook filters out wards where either value is missing and prints descriptive statistics.

### 17.3 Recorded ward summary

| Statistic | Population | Households |
|---|---:|---:|
| Minimum | 6,762 | 627 |
| 1st quartile | 28,674 | 6,304 |
| Median | 34,662 | 7,388 |
| Mean | 36,512 | 7,862 |
| 3rd quartile | 41,904 | 9,017 |
| Maximum | 69,177 | 15,740 |

The cleaned table is shown with **108 wards** and **12 columns** (the original selected fields plus `Population` and `Households`).

## 18. Ward population visualization

The notebook sorts ward names by population for the bar chart, flips the coordinates so the ward labels can be read more easily, and saves `09_ward_population.png`.

```r
p9 <- ggplot(
  ward_data,
  aes(x = reorder(Name, Population), y = Population)
) +
  geom_col() +
  coord_flip() +
  labs(
    title = "Population Distribution Across Hyderabad Wards",
    x = "Ward",
    y = "Population"
  ) +
  theme_minimal()

print(p9)
ggsave("/content/EWaste_Case_Study/outputs/09_ward_population.png",
       p9, width = 10, height = 8, dpi = 300)
```

![Population distribution across wards](E_Waste_Project_Guide_assets/09_ward_population.png)

This plot highlights that ward populations differ. Population and households can be useful **proxy indicators** when exploring where collection service may be needed, but they are not direct measurements of e-waste generation.

## 19. K-Means clustering for collection zones

### 19.1 What K-Means does in this project

K-Means is an unsupervised clustering algorithm. It groups records by similarity in the numerical features supplied to it. Here, the notebook uses two variables:

- `Population`
- `Households`

The algorithm is not given e-waste quantities per ward, map coordinates, road distances, or collection-centre locations. Consequently, its output should be understood as **demographic groupings for preliminary collection planning**, not an optimized geographic route plan.

### 19.2 Select features, standardize, and fit three clusters

```r
zone_features <- ward_data %>%
  select(Population, Households)

zone_scaled <- scale(zone_features)

set.seed(123)
kmeans_model <- kmeans(
  zone_scaled,
  centers = 3,
  nstart = 25
)

ward_data$Zone <- factor(
  kmeans_model$cluster,
  labels = c("Zone 1", "Zone 2", "Zone 3")
)

print(ward_data %>%
        select(Name, Population, Households, Zone))
```

**What the important lines mean:**

- `select(Population, Households)` supplies only the two demographic variables to the algorithm.
- `scale()` standardizes the variables so that differences in their numerical scales do not dominate the clustering.
- `centers = 3` requests three groups.
- `nstart = 25` tries multiple starting configurations and keeps the best result found by the implementation.
- `set.seed(123)` helps make the clustering repeatable.
- `factor()` gives readable labels to the numeric cluster IDs. The labels Zone 1, Zone 2, and Zone 3 are names, not rankings.

The printed ward assignment table contains 108 ward records; the notebook output displays the first ten and notes that 98 rows remain.

### 19.3 Zone summary output

The notebook groups wards by `Zone` and calculates the number of wards, total population, total households, and intended average population/households.

**Recorded output:**

| Zone | Wards | Population total | Households total | `Avg_Population` as printed | `Avg_Households` as printed |
|---|---:|---:|---:|---:|---:|
| Zone 1 | 49 | 1,922,256 | 414,164 | 1,922,256 | 414,164 |
| Zone 2 | 45 | 1,202,798 | 262,991 | 1,202,798 | 262,991 |
| Zone 3 | 14 | 818,269 | 171,896 | 818,269 | 171,896 |

The totals add to 3,943,323 people and 849,051 households across the 108 ward rows in the clustering output.

#### Important code issue in the recorded zone averages

The values shown in the last two columns are **not actual per-ward averages**. In the notebook's `summarise()` call, `Population` and `Households` are overwritten with their sums before `Avg_Population = mean(Population)` and `Avg_Households = mean(Households)` are evaluated. This causes the “average” fields to repeat the totals, as seen in the output.

A corrected summary would use different names for totals so the mean calculations still use the original ward-level vectors:

```r
cluster_summary <- ward_data %>%
  group_by(Zone) %>%
  summarise(
    Wards = n(),
    Total_Population = sum(Population, na.rm = TRUE),
    Total_Households = sum(Households, na.rm = TRUE),
    Avg_Population = mean(Population, na.rm = TRUE),
    Avg_Households = mean(Households, na.rm = TRUE),
    .groups = "drop"
  ) %>%
  arrange(Zone)

print(cluster_summary)
```

Using the totals and ward counts in the recorded result, the corresponding averages are approximately:

| Zone | Mean population per ward | Mean households per ward |
|---|---:|---:|
| Zone 1 | 39,229.7 | 8,452.3 |
| Zone 2 | 26,728.8 | 5,844.2 |
| Zone 3 | 58,447.8 | 12,278.3 |

These are the total values divided by the number of wards for each group. The notebook's original output was not corrected during its recorded run; the corrected code above is a recommended fix.

### 19.4 Collection-zone scatter plot

The first zone chart plots population against households and uses point shape to show cluster membership.

![Hyderabad ward collection zones](E_Waste_Project_Guide_assets/10_collection_zones.png)

**Recorded plot file:** `10_collection_zones.png`.

### 19.5 Total population by zone

![Population by collection zone](E_Waste_Project_Guide_assets/11_zone_population.png)

**Recorded plot file:** `11_zone_population.png`.

### 19.6 Total households by zone

![Households by collection zone](E_Waste_Project_Guide_assets/12_zone_households.png)

**Recorded plot file:** `12_zone_households.png`.

The charts present the zone totals and the demographic clustering result. They can help explain how the wards were grouped, but actual operational planning would also need geographic proximity, expected e-waste quantities, travel times, collection frequency, and vehicle capacity.

## 20. Final results printed by the notebook

The last code cell prints the dataset sizes, model metrics, the selected model, and the cluster summary. The key recorded output is:

```text
E-WASTE COLLECTION DEMAND FORECASTING
AND ZONE OPTIMIZATION USING R

1. DATASET SUMMARY
Historical observations: 5
Ward observations: 108

2. ARIMA PERFORMANCE
MAE  : 192801.8
RMSE : 205749.2
MAPE : 14.87586 %

3. RANDOM FOREST PERFORMANCE
MAE  : 267627.4
RMSE : 277100.4
MAPE : 20.5349 %

4. BEST MODEL
ARIMA performed better based on RMSE.

PROJECT EXECUTION COMPLETED
```

The final execution summary matches the metric table above and reports the clustering totals shown in the zone summary section.

## 21. Files the notebook saves

The notebook saves the following files beneath `/content/EWaste_Case_Study/outputs/` when the relevant cells execute successfully:

### Charts

| File | Contents |
|---|---|
| `01_ewaste_generation_vs_recycling.png` | Generation versus recycling bars |
| `02_ewaste_recycling_percentage.png` | Recycling percentage by financial year |
| `03_annual_ewaste_trend.png` | Annual generation trend |
| `04_ewaste_time_series.png` | Time-series plot |
| `05_arima_forecast.png` | ARIMA forecasts and intervals |
| `06_model_comparison.png` | MAE/RMSE/MAPE comparison |
| `07_actual_vs_predicted.png` | Actual versus test predictions |
| `08_additional_ewaste_trend.png` | Additional data trend, including partial-year entry |
| `09_ward_population.png` | Population across wards |
| `10_collection_zones.png` | Population/household points by cluster |
| `11_zone_population.png` | Total population by zone |
| `12_zone_households.png` | Total households by zone |

### CSV result tables

| File | Contents |
|---|---|
| `arima_forecast_values.csv` | ARIMA point forecasts and 80% interval values for 2025 and 2026 |
| `model_comparison.csv` | ARIMA and Random Forest MAE, RMSE, and MAPE |
| `prediction_comparison.csv` | Actual, ARIMA, and Random Forest values for test years 2023 and 2024 |
| `zone_summary.csv` | Zone ward counts, population totals, household totals, and the average fields as originally computed |

The notebook creates a `data` folder too, but the shown cells save the generated charts and CSV result tables to the `outputs` folder. In a hosted notebook environment, files under `/content` may need to be downloaded or copied before the session is closed.

## 22. Key findings

1. **E-waste generation varies year to year.** The main dataset rises through 2022-23, drops in 2023-24, and rises again in 2024-25.
2. **The supplied generation/recycling ratio rises.** The computed recycling percentage increases from 26.3% to 82.9% over the five reported rows.
3. **ARIMA performed better in this notebook's test.** It produced smaller MAE, RMSE, and MAPE than Random Forest for the two held-out observations.
4. **Forecasts are constant in the fitted ARIMA model.** The automatically selected ARIMA(0,0,0) model predicts about 1.442 million MT for each forecast step.
5. **Random Forest is weakly supported by the available data.** The training set has only three rows and the model reports negative percentage variance explained.
6. **Ward demographics vary materially.** The 108 selected wards have populations from 6,762 to 69,177 and household counts from 627 to 15,740.
7. **K-Means creates three demographic groups.** The recorded output assigns 49, 45, and 14 wards to the three zones. This is a demographic grouping, not a validated geographic route optimization.
8. **The zone summary contains a calculation bug.** The recorded `Avg_Population` and `Avg_Households` columns repeat the totals because of the order of expressions in `summarise()`.

## 23. Limitations and cautions

These limitations are important when presenting the project or interpreting the results:

- **Very small forecasting dataset:** The annual modelling dataset has only five records. Only three records are used to train each model in the test experiment, and the evaluation is based on two test observations. The metrics are highly sensitive to individual years.
- **The additional partial-year data is not modelled:** The row marked `2025-26 (so far)` is plotted separately but does not update the ARIMA or Random Forest models.
- **Financial year handling needs care:** The code extracts the first four characters of financial-year labels and then forecasts numeric years. The reporting-year labels should be explicitly mapped before operational use.
- **Random Forest cannot learn much from three training samples:** The warning about few unique response values and negative variance explained indicate that this model result is not a reliable forecasting solution yet.
- **No rich predictor variables:** Random Forest is supplied only `Year_Num` as its predictor. It does not use electronics sales, population, household income, collection volumes, recycling capacity, policy changes, or other potentially relevant variables.
- **Census recency:** The Excel filename indicates a 2011 census source. Current ward boundaries and current population/household estimates should be checked before planning a present-day service.
- **Proxy-based zone formation:** Population and household counts are used as proxies. The notebook does not have direct ward-wise e-waste volumes, ward coordinates, road-network distances, travel time, collection points, or vehicle-capacity constraints.
- **Cluster labels are arbitrary:** Zone 1, Zone 2, and Zone 3 are labels assigned to K-Means cluster IDs; the numbering does not itself mean low, medium, or high demand.
- **Summary-average bug:** The average columns in the recorded cluster summary are wrong and should be fixed before those columns are used in a report.
- **Data provenance is incomplete in the notebook:** The code lists file names but not the full source URLs, publication dates, or definitions. Those should be documented in the final academic report.

## 24. Recommended improvements

1. **Collect more years of consistent data.** Use a longer series of fully completed financial years, and document how each annual quantity is defined.
2. **Incorporate validated current data.** Investigate whether the partial-year figure can be converted to a comparable period; do not mix partial and full-year totals without an explicit method.
3. **Use time-aware evaluation.** Use rolling-origin or expanding-window validation and compare against a simple baseline such as the latest observed value or historical mean. Report uncertainty rather than relying on one two-year holdout.
4. **Reconsider model complexity.** With only five observations, a complex machine-learning model is not justified. Collect more data before selecting or tuning models such as Random Forest.
5. **Fix and re-run the zone summary.** Use distinct names for total and average fields, as shown in Section 19.3.
6. **Use direct collection-demand data when available.** Ward-level e-waste collection records, drop-off volumes, pickup requests, or collection-centre weights would be stronger targets than population alone.
7. **Improve spatial planning.** Add ward polygons or coordinates, road distances, travel times, collection-center locations, vehicle capacity, and target service intervals. A routing or location-allocation method would then be needed to optimize actual routes.
8. **Refresh demographic data.** Confirm current ward boundaries and current population/household estimates rather than assuming an older census snapshot reflects today's demand.
9. **Document sources and units.** Record source links, extraction dates, missing-value decisions, units, definitions, and any revisions to the data.
10. **Rerun the notebook after corrections.** Save updated CSV tables and plots so the written results match the corrected code and current data.

## 25. How to run the notebook

1. Open the `.ipynb` notebook in an R-capable environment, such as a configured R notebook runtime.
2. Upload the three data files used in the code: `RS_Session_269_AU_525_A.csv`, `RS_Session_270_AS_140_A_i.csv`, and `DDW_PCA2805_2011_MDDS with UI.xlsx`.
3. Confirm the files are available at the paths used in the notebook. The current code expects them under `/content/`.
4. Run the package installation cell and then the library-loading cell.
5. Execute the remaining cells in order because later cells use objects created earlier (for example, `ewaste_clean`, `annual_data`, `model_comparison`, and `ward_data`).
6. Check the console outputs and plots. Inspect the forecast metrics and confirm whether the data and validation approach are appropriate for the conclusions you plan to make.
7. If using the corrected zone summary in Section 19.3, replace the original summary cell and rerun the summary and zone-plot cells after it.
8. Download the generated files from `/content/EWaste_Case_Study/outputs/` before ending the hosted session.

**Execution note:** The paths and outputs described above reflect the uploaded notebook's recorded run. Re-running it with changed input files, a different package version, or a different runtime may produce different results.

## 26. Conclusion

The project combines descriptive analysis, forecasting, and demographic clustering to build a first-stage view of e-waste collection planning. The main dataset shows year-to-year variation in generation and a rise in the recorded recycled-to-generated ratio across the five included financial years. On the small two-observation test set, ARIMA has lower prediction errors than Random Forest, so it is the better of those two models in this particular recorded experiment. The census analysis also groups 108 wards into three clusters using population and household counts, offering a preliminary demographic segmentation for collection planning.

The results should be treated as **exploratory rather than deployment-ready**. A stronger version needs a longer and consistently defined time series, proper time-based validation, current ward demographic data, direct ward-level collection-demand records, geographic information for route planning, and correction of the summary-average bug. With those improvements, the workflow could become more useful for evidence-based e-waste collection capacity planning.

---

*This guide is based on the code and outputs already stored in the uploaded notebook. Interpretations and recommended fixes are explicitly identified as such; the original recorded outputs are preserved in the tables above.*
