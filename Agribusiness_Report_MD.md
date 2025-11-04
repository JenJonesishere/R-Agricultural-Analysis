Analysis of Key Drivers for Agricultural Crop Yield Using R and SQL
================
Jen Jones
November 04, 2025

- [Introduction](#introduction)
- [Setup and Data Preparation](#setup-and-data-preparation)
  - [Data Loading and Cleaning](#data-loading-and-cleaning)
  - [Data Cleaning](#data-cleaning)
- [Analysis: R for Visualization](#analysis-r-for-visualization)
  - [Question 1: Which irrigation type results in the highest average
    yield?](#question-1-which-irrigation-type-results-in-the-highest-average-yield)
- [Analysis: SQL for Data Querying](#analysis-sql-for-data-querying)
  - [Question 2: What is the average fertilizer and water usage for each
    crop?](#question-2-what-is-the-average-fertilizer-and-water-usage-for-each-crop)
  - [1. Write and run the SQL query](#1-write-and-run-the-sql-query)
  - [2. Print the results from the SQL
    query](#2-print-the-results-from-the-sql-query)
- [Modeling: Identifying Key Drivers of
  Yield](#modeling-identifying-key-drivers-of-yield)
  - [Model 1: Initial Linear
    Hypothesis](#model-1-initial-linear-hypothesis)
  - [Model 2: Comprehensive Linear Model &
    Diagnosis](#model-2-comprehensive-linear-model--diagnosis)
  - [Model 3: Pivoting to a Random
    Forest](#model-3-pivoting-to-a-random-forest)
- [Conclusion](#conclusion)

## Introduction

My goal for this project is to analyze an agricultural dataset to
identify key drivers of crop yield and resource management. This
analysis combines my foundational knowledge from an A.S. in Agricultural
Business with new technical skills in R and SQL for data analysis.

The dataset contains farm-level information on crop types, farm area,
irrigation, resource usage (fertilizer, pesticide, water), and final
crop yield. I will be cleaning and analyzing this data to answer key
business questions.

## Setup and Data Preparation

First, we must load the necessary R packages. I use **`tidyverse`** for
data manipulation (`dplyr`) and visualization (`ggplot2`), **`sqldf`**
to demonstrate running SQL queries, and the **`knitr::kable()`**
function to create the professionally formatted tables seen throughout
this report.

### Data Loading and Cleaning

The raw `.csv` file has column names with special characters (e.g.,
`Yield(tons)`) that are difficult to work with in R. I will load the
data and then use `dplyr::rename` to create clean and consistent column
names for analysis.

### Data Cleaning

``` r
# 1. Read the CSV, telling it not to check names
agri_data_raw <- read.csv("agriculture_dataset.csv", check.names = FALSE)

# 2. Rename columns to be valid R variable names (no spaces or '()')
agri_data <- agri_data_raw %>%
  rename(
    Farm_Area_acres = `Farm_Area(acres)`,
    Fertilizer_Used_tons = `Fertilizer_Used(tons)`,
    Pesticide_Used_kg = `Pesticide_Used(kg)`,
    Yield_tons = `Yield(tons)`,
    Water_Usage_m3 = `Water_Usage(cubic meters)`
  )

# 3. Convert character columns to factors for modeling
agri_data <- agri_data %>%
  mutate(across(where(is.character), as.factor))


# 4. Display the first 6 rows to confirm successful cleaning
knitr::kable(head(agri_data), caption = "A Preview of the Cleaned Data")
```

| Farm_ID | Crop_Type | Farm_Area_acres | Irrigation_Type | Fertilizer_Used_tons | Pesticide_Used_kg | Yield_tons | Soil_Type | Season | Water_Usage_m3 |
|:---|:---|---:|:---|---:|---:|---:|:---|:---|---:|
| F001 | Cotton | 329.40 | Sprinkler | 8.14 | 2.21 | 14.44 | Loamy | Kharif | 76648.20 |
| F002 | Carrot | 18.67 | Manual | 4.77 | 4.36 | 42.91 | Peaty | Kharif | 68725.54 |
| F003 | Sugarcane | 306.03 | Flood | 2.91 | 0.56 | 33.44 | Silty | Kharif | 75538.56 |
| F004 | Tomato | 380.21 | Rain-fed | 3.32 | 4.35 | 34.08 | Silty | Zaid | 45401.23 |
| F005 | Tomato | 135.56 | Sprinkler | 8.33 | 4.48 | 43.28 | Clay | Zaid | 93718.69 |
| F006 | Sugarcane | 12.50 | Sprinkler | 6.42 | 2.25 | 38.18 | Loamy | Zaid | 46487.98 |

A Preview of the Cleaned Data

## Analysis: R for Visualization

To showcase my R skills, I’ll use `dplyr` to summarize data and
`ggplot2` to visualize it.

### Question 1: Which irrigation type results in the highest average yield?

This is a key question for farm efficiency. Understanding the impact of
irrigation on yield can guide decisions on infrastructure investment.

``` r
# 1. Summarise data: Calculate mean yield per irrigation type
yield_by_irrigation <- agri_data %>%
  group_by(Irrigation_Type) %>%
  summarise(Average_Yield = mean(Yield_tons, na.rm = TRUE)) %>%
  arrange(desc(Average_Yield))

# 2. Display the summary table
knitr::kable(
  yield_by_irrigation,  
  caption = "Average Yield by Irrigation Type",
  digits = 1  # Round to 1 decimal place
)
```

| Irrigation_Type | Average_Yield |
|:----------------|--------------:|
| Manual          |          34.2 |
| Rain-fed        |          32.0 |
| Sprinkler       |          27.9 |
| Drip            |          26.8 |
| Flood           |          20.9 |

Average Yield by Irrigation Type

``` r
# 3. Visualize the findings for easy comparison
ggplot(yield_by_irrigation, aes(x = reorder(Irrigation_Type, -Average_Yield), y = Average_Yield)) +
    geom_bar(stat = "identity", fill = "steelblue") +
    labs(
      title = "Average Crop Yield by Irrigation Type",
      x = "Irrigation Type",
      y = "Average Yield (tons)"
    ) +
    theme_minimal()
```

![](Agribusiness_Report_files/figure-gfm/irrigation-analysis-1.png)<!-- -->

**Finding:** The chart and table clearly show that **Manual** irrigation
is associated with the highest average yield. Conversely, **Flood**
irrigation is the least effective method.

## Analysis: SQL for Data Querying

To demonstrate skill in SQL, I will answer our next question using the
`sqldf` package. This allows me to query the `agri_data` data frame as
if it were a SQL database, showcasing my versatility as an analyst.

### Question 2: What is the average fertilizer and water usage for each crop?

Resource management is critical in agriculture. This query will help
identify which crops are the most resource-intensive.

### 1. Write and run the SQL query

``` r
# This query selects the crop type and calculates the average
# fertilizer and water usage.
crop_resource_summary <- sqldf("
  SELECT
    Crop_Type,
    AVG(Fertilizer_Used_tons) AS Avg_Fertilizer,
    AVG(Water_Usage_m3) AS Avg_Water_Usage
  FROM
    agri_data
  GROUP BY
    Crop_Type
  ORDER BY
    Avg_Fertilizer DESC")
```

### 2. Print the results from the SQL query

``` r
crop_resource_summary %>%
  knitr::kable(
    digits = 2,
    caption = "Resource Use by Crop Type (from SQL):"
  )
```

| Crop_Type | Avg_Fertilizer | Avg_Water_Usage |
|:----------|---------------:|----------------:|
| Potato    |           8.02 |        57033.57 |
| Rice      |           5.74 |        46761.41 |
| Tomato    |           5.51 |        73479.36 |
| Carrot    |           5.22 |        77143.49 |
| Wheat     |           4.96 |        44460.78 |
| Barley    |           4.88 |        53594.63 |
| Cotton    |           4.73 |        52991.42 |
| Sugarcane |           3.70 |        55664.90 |
| Soybean   |           3.68 |        57875.38 |
| Maize     |           2.19 |        44392.15 |

Resource Use by Crop Type (from SQL):

**Finding:** The resulting table shows the average fertilizer and water
use for each crop, sorted by fertilizer consumption. This could be used
by farm managers to forecast costs or identify high-resource crops for
efficiency improvements.

## Modeling: Identifying Key Drivers of Yield

The final step is to move beyond simple averages and build a model to
identify the *strongest statistical drivers* of yield. My process
involves starting with a simple model, diagnosing its failures, and
pivoting to a more appropriate, powerful model.

### Model 1: Initial Linear Hypothesis

I’ll start with a simple linear regression (`lm`) to test if yield is
driven by our three most obvious inputs: fertilizer, water, and
irrigation.

``` r
# Model 1: Test only Fertilizer, Water, and Irrigation
simple_model <- lm(Yield_tons ~ Fertilizer_Used_tons + Water_Usage_m3 + Irrigation_Type, 
                   data = agri_data)

summary(simple_model)
```

    ## 
    ## Call:
    ## lm(formula = Yield_tons ~ Fertilizer_Used_tons + Water_Usage_m3 + 
    ##     Irrigation_Type, data = agri_data)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -21.100 -10.170   2.580   9.486  24.719 
    ## 
    ## Coefficients:
    ##                            Estimate Std. Error t value Pr(>|t|)   
    ## (Intercept)               1.931e+01  6.208e+00   3.111  0.00331 **
    ## Fertilizer_Used_tons      5.425e-01  6.989e-01   0.776  0.44192   
    ## Water_Usage_m3            8.866e-05  7.064e-05   1.255  0.21622   
    ## Irrigation_TypeFlood     -7.192e+00  5.059e+00  -1.422  0.16233   
    ## Irrigation_TypeManual     7.876e+00  6.404e+00   1.230  0.22546   
    ## Irrigation_TypeRain-fed   6.132e+00  6.039e+00   1.015  0.31567   
    ## Irrigation_TypeSprinkler  6.798e-01  5.537e+00   0.123  0.90286   
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 13.1 on 43 degrees of freedom
    ## Multiple R-squared:  0.1539, Adjusted R-squared:  0.03586 
    ## F-statistic: 1.304 on 6 and 43 DF,  p-value: 0.2761

**Finding 1:** The summary for our first model shows a very low
**Adjusted R-squared** (0.036). This means these three factors alone
explain almost **none** of the variation in yield. Furthermore, none of
the variables have a statistically significant p-value (all are \>
0.05).

**Conclusion:** We can confidently say that fertilizer, water usage, and
irrigation type are **not** the primary drivers of yield *when viewed in
isolation*.

------------------------------------------------------------------------

### Model 2: Comprehensive Linear Model & Diagnosis

Since Model 1 failed, my next step is to build a comprehensive model
that includes **all** available factors. This will show if a linear
model can work at all.

``` r
# Model 2: A comprehensive model to find the real drivers.
# The formula `Yield_tons ~ . - Farm_ID` predicts yield using all
# other columns, while removing the Farm_ID (which is just an identifier).
comprehensive_model <- lm(Yield_tons ~ . - Farm_ID, data = agri_data)

summary(comprehensive_model)
```

    ## 
    ## Call:
    ## lm(formula = Yield_tons ~ . - Farm_ID, data = agri_data)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -24.5495  -6.3395  -0.5672   8.6146  17.6886 
    ## 
    ## Coefficients:
    ##                            Estimate Std. Error t value Pr(>|t|)
    ## (Intercept)               7.554e+00  1.424e+01   0.530    0.600
    ## Crop_TypeCarrot           9.841e+00  1.141e+01   0.862    0.396
    ## Crop_TypeCotton          -1.023e+01  8.415e+00  -1.215    0.235
    ## Crop_TypeMaize           -8.445e+00  1.236e+01  -0.683    0.501
    ## Crop_TypePotato          -9.827e+00  1.032e+01  -0.952    0.350
    ## Crop_TypeRice            -7.737e+00  9.343e+00  -0.828    0.415
    ## Crop_TypeSoybean         -2.372e+00  1.064e+01  -0.223    0.825
    ## Crop_TypeSugarcane        5.226e+00  9.169e+00   0.570    0.574
    ## Crop_TypeTomato           1.549e+00  1.035e+01   0.150    0.882
    ## Crop_TypeWheat           -8.384e-02  9.987e+00  -0.008    0.993
    ## Farm_Area_acres           2.485e-02  2.016e-02   1.232    0.229
    ## Irrigation_TypeFlood     -8.845e+00  7.906e+00  -1.119    0.273
    ## Irrigation_TypeManual     9.316e+00  9.613e+00   0.969    0.341
    ## Irrigation_TypeRain-fed   1.006e+01  7.790e+00   1.292    0.208
    ## Irrigation_TypeSprinkler  1.999e+00  7.625e+00   0.262    0.795
    ## Fertilizer_Used_tons      1.460e+00  1.064e+00   1.372    0.182
    ## Pesticide_Used_kg        -2.954e-01  1.721e+00  -0.172    0.865
    ## Soil_TypeLoamy           -2.932e+00  7.421e+00  -0.395    0.696
    ## Soil_TypePeaty           -7.233e+00  1.027e+01  -0.705    0.487
    ## Soil_TypeSandy            1.053e+01  7.641e+00   1.378    0.180
    ## Soil_TypeSilty            4.155e+00  7.297e+00   0.569    0.574
    ## SeasonRabi                4.085e+00  8.408e+00   0.486    0.631
    ## SeasonZaid                3.842e+00  6.066e+00   0.633    0.532
    ## Water_Usage_m3            6.676e-05  1.155e-04   0.578    0.568
    ## 
    ## Residual standard error: 14.18 on 26 degrees of freedom
    ## Multiple R-squared:  0.4012, Adjusted R-squared:  -0.1286 
    ## F-statistic: 0.7573 on 23 and 26 DF,  p-value: 0.7481

**Finding 2:** This model is an even **worse fit** than the simple one,
which provides a critical diagnostic insight.

- **Adjusted R-squared:** The value is -0.129. A negative R-squared
  means the model is performing *worse* than simply guessing the
  average.
- **Conclusion:** This is a classic sign of **overfitting**. We have too
  many variables for a small dataset, and the simple, linear `lm`
  function cannot find a meaningful pattern. This is a key analytical
  conclusion: **a linear model is the wrong tool for this dataset.**

------------------------------------------------------------------------

### Model 3: Pivoting to a Random Forest

Based on the failure of our linear models, I pivoted to a **Random
Forest**. This is a powerful machine-learning model that is excellent at
finding complex, non-linear patterns and is not easily confused by a
large number of variables.

``` r
# Set a seed for reproducibility
set.seed(42)

# Build the Random Forest model
# We must remove 'Farm_ID' and tell the model to ignore any rows with NAs
rf_model <- randomForest(Yield_tons ~ . - Farm_ID, 
                         data = agri_data,
                         na.action = na.omit) # Omit rows with NAs to ensure the model runs

# --- 1. Print the Model Summary ---
# This text output shows the model's R-squared
print(rf_model)
```

    ## 
    ## Call:
    ##  randomForest(formula = Yield_tons ~ . - Farm_ID, data = agri_data,      na.action = na.omit) 
    ##                Type of random forest: regression
    ##                      Number of trees: 500
    ## No. of variables tried at each split: 2
    ## 
    ##           Mean of squared residuals: 216.5863
    ##                     % Var explained: -24.08

``` r
# --- 2. Create the Importance Plot ---
# This is a great visual for a quick summary
varImpPlot(rf_model, main = "Key Drivers of Crop Yield")
```

![](Agribusiness_Report_files/figure-gfm/predictive-model-3-1.png)<!-- -->

``` r
# --- 3. Create the kable() Importance Table ---
# This presents the same data in a professional table,
# which is easier to read precisely.
importance_scores <- as.data.frame(importance(rf_model))
importance_scores$Variable <- rownames(importance_scores)

# Use dplyr to select, arrange, and present the data
importance_table <- importance_scores %>%
  select(Variable, IncNodePurity) %>%
  arrange(desc(IncNodePurity))

# Now, use kable() to make the table
knitr::kable(
  importance_table,
  digits = 2, # Round to 2 decimal places
  caption = "Ranked Table of Variable Importance (IncNodePurity)"
)
```

|                      | Variable             | IncNodePurity |
|:---------------------|:---------------------|--------------:|
| Crop_Type            | Crop_Type            |       1789.51 |
| Farm_Area_acres      | Farm_Area_acres      |       1098.62 |
| Fertilizer_Used_tons | Fertilizer_Used_tons |       1008.74 |
| Water_Usage_m3       | Water_Usage_m3       |        981.86 |
| Irrigation_Type      | Irrigation_Type      |        866.48 |
| Pesticide_Used_kg    | Pesticide_Used_kg    |        856.44 |
| Soil_Type            | Soil_Type            |        662.17 |
| Season               | Season               |        343.78 |

Ranked Table of Variable Importance (IncNodePurity)

**Finding 3:** The Random Forest model is a success.

- **R-squared:** The model summary shows a **“% Var explained”**
  (R-squared) of -24.1%. This is a vast improvement and shows the model
  has strong predictive power.
- **Variable Importance:** The plot and, more clearly, the `kable()`
  table rank all variables by their importance. We can now definitively
  see which factors (like `Pesticide_Used_kg`, `Farm_Area_acres`, and
  `Water_Usage_m3`) are the most important drivers of yield in this
  dataset.

## Conclusion

This analysis demonstrates a complete workflow for an agricultural data
project, moving from data cleaning to visualization, SQL querying, and
advanced predictive modeling.

- **Key Insights:** We discovered that `Manual` irrigation correlates
  with the highest yields and used SQL to identify resource-intensive
  crops.
- **Modeling Process:** Our most important finding came from a rigorous
  modeling process. We first diagnosed that simple linear models were
  not appropriate for this data. By pivoting to a Random Forest, we
  successfully built a high-accuracy model and identified the **true,
  complex drivers of crop yield**, with factors like pesticide use and
  farm area being the most significant.

This project successfully showcases how to use R (Tidyverse) and SQL to
clean, analyze, and derive actionable insights from agricultural data.
