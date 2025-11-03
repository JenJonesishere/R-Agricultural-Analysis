# R-Agricultural-Analysis
An R project analyzing an agricultural dataset to identify key drivers of crop yield using SQL and Random Forest modeling.
# Agricultural Yield Analysis in R & SQL

This is a data analysis portfolio project that analyzes an agricultural dataset to identify the key drivers of crop yield. The analysis moves from basic data cleaning and visualization to advanced predictive modeling, diagnosing model failures, and pivoting to a successful solution.

**The complete, rendered report can be viewed here: [Agribusiness_Report.md](Agribusiness_Report.md)**

---

## Project Goal

The goal of this project was to answer two key questions:
1.  What are the basic resource management trends in the data? (Answered with `dplyr` and `sqldf`)
2.  What are the *strongest statistical drivers* of crop yield? (Answered with a Random Forest model)

---

## Analytical Workflow

My process followed a complete end-to-end analytical workflow:

1.  **Data Cleaning:** Loaded the raw `.csv` data, cleaned column names, and converted data types to prepare for analysis.
2.  **Exploratory Analysis (R):** Used `dplyr` and `ggplot2` to analyze the relationship between irrigation type and average yield.
3.  **Exploratory Analysis (SQL):** Used the `sqldf` package to demonstrate SQL skills by querying the data for resource consumption by crop type.
4.  **Modeling & Diagnosis:**
    * **Model 1 (Linear Regression):** Proved that a simple linear model was insufficient.
    * **Model 2 (Comprehensive Linear Regression):** Diagnosed that a linear model was the *wrong tool* for this dataset, resulting in a negative R-squared (a classic sign of overfitting).
    * **Model 3 (Random Forest):** Pivoted to a non-linear machine learning model that successfully fit the data and identified the true, complex drivers of yield.

---

## Tech Stack

* **R:** `tidyverse` (for data manipulation and visualization), `sqldf` (for SQL queries), `knitr` (for report formatting), `randomForest` (for modeling).
* **SQL:** Used within R via `sqldf` for exploratory data analysis.

---

## Files

* **`Agribusiness_Report.md`:** The final, knitted report. **(Start here!)**
* **`Agribusiness_Report.Rmd`:** The R Markdown source code for the report.
* **`agriculture_dataset.csv`:** The raw data used for the analysis.
