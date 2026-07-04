# Household Electric Power Consumption Analysis in R

This project explores the **Household Electric Power Consumption** dataset and shows how to prepare it for **regression** and **clustering** in R.

The dataset contains minute-level household electricity measurements collected over almost four years, from December 2006 to November 2010, with over 2 million rows and some missing values in the measurements. [web:1][web:3]

## Dataset description

This dataset is a multivariate time series with the following variables:

- `date`
- `time`
- `global_active_power`
- `global_reactive_power`
- `voltage`
- `global_intensity`
- `sub_metering_1`
- `sub_metering_2`
- `sub_metering_3`

The target in many analyses can be derived from the consumption variables, and the archive includes missing values in about 1.25% of rows, where missing measurements are represented by blank fields between semicolons. [web:1][web:3]

## Project goals

The main goals of this project are:

- Clean and format the raw text file.
- Combine date and time into a proper datetime column.
- Handle missing values.
- Perform exploratory analysis.
- Build a regression model to predict electricity demand or related variables.
- Apply clustering to identify similar consumption patterns.

## Suggested workflow in R

### 1. Load the data

```r
library(data.table)
library(dplyr)
library(ggplot2)
library(lubridate)
library(cluster)
library(factoextra)

df <- fread("household_power_consumption.txt", sep = ";", na.strings = "?", header = TRUE)
```

### 2. Clean and transform

```r
df <- df %>%
  mutate(
    datetime = dmy_hms(paste(date, time)),
    across(
      c(global_active_power, global_reactive_power, voltage, global_intensity,
        sub_metering_1, sub_metering_2, sub_metering_3),
      as.numeric
    )
  ) %>%
  select(datetime, everything())
```

### 3. Handle missing values

```r
df_complete <- df %>% tidyr::drop_na()
```

You can also impute missing values instead of removing them, depending on your modeling goal.

### 4. Create features

Useful engineered features may include:

- Hour of day.
- Day of week.
- Month.
- Lagged power values.
- Rolling averages.
- Total sub-metered consumption.

```r
df_features <- df_complete %>%
  mutate(
    hour = hour(datetime),
    wday = wday(datetime, label = TRUE),
    month = month(datetime),
    total_sub_metering = sub_metering_1 + sub_metering_2 + sub_metering_3
  )
```

## Regression approach

A regression model can be used to predict `global_active_power` or another numeric outcome.

Example linear regression:

```r
model <- lm(
  global_active_power ~ voltage + global_intensity + sub_metering_1 +
    sub_metering_2 + sub_metering_3 + hour + month,
  data = df_features
)

summary(model)
```

If you want stronger predictive performance, consider:

- Random forest.
- Gradient boosting.
- Time-aware train/test splits.
- Lag-based features.

## Clustering approach

Clustering can help identify typical household consumption patterns. For time series, standard clustering often works better after aggregation or feature extraction, because the raw minute-level series is very large and highly dimensional. Time series representation methods and k-medoids are commonly used for this kind of electricity profile analysis. [web:7][web:9]

Example: cluster daily load profiles

```r
df_daily <- df_features %>%
  mutate(day = as.Date(datetime)) %>%
  group_by(day) %>%
  summarise(
    avg_power = mean(global_active_power, na.rm = TRUE),
    avg_intensity = mean(global_intensity, na.rm = TRUE),
    avg_voltage = mean(voltage, na.rm = TRUE),
    total_sub_metering = sum(total_sub_metering, na.rm = TRUE)
  )

scaled_daily <- scale(df_daily %>% select(-day))

set.seed(123)
km <- kmeans(scaled_daily, centers = 3, nstart = 25)

df_daily$cluster <- factor(km$cluster)
```

### Visualize clusters

```r
ggplot(df_daily, aes(x = avg_voltage, y = avg_power, color = cluster)) +
  geom_point(alpha = 0.7) +
  theme_minimal()
```

## Example outputs

This project can produce:

- Summary tables.
- Missing value analysis.
- Regression diagnostics.
- Cluster profiles.
- Time series plots of power consumption.

## Notes

- The raw dataset is stored as a text file with semicolon-separated fields.
- Some timestamps contain missing measurement values, so cleaning is an important first step. [web:1][web:3]
- Because the dataset is time-series data, random train/test splits should be avoided when possible.
- Feature engineering often matters more than the choice of model for this dataset.

## References

- Kaggle dataset: Household Electric Power Consumption. [web:1]
- Dataset description and time-series use case. [web:3][web:7]
- Time series clustering ideas in R. [web:7][web:9]
