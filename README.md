Agricultural Yield Analysis in R & SQL

➡️ View the Full Interactive Report (Recommended) ⬅️

The link above is the primary, interactive HTML version of this report, hosted on GitHub Pages.

For browsing directly on GitHub, you can also see the simple Markdown version.

Project Goal 

The goal of this project was to answer two key questions:

What are the basic resource management trends in the data? (Answered with dplyr and sqldf)

What are the strongest statistical drivers of crop yield? (Answered with a Random Forest model)

Analytical Workflow

My process followed a complete end-to-end analytical workflow:

Data Cleaning: Loaded the raw .csv data, cleaned column names, and converted data types to prepare for analysis.

Exploratory Analysis (R): Used dplyr and ggplot2 to analyze the relationship between irrigation type and average yield.

Exploratory Analysis (SQL): Used the sqldf package to demonstrate SQL skills by querying the data for resource consumption by crop type.

Modeling & Diagnosis:

Model 1 (Linear Regression): Proved that a simple linear model was insufficient.

Model 2 (Comprehensive Linear Regression): Diagnosed that a linear model was the wrong tool for this dataset, resulting in a negative R-squared (a classic sign of overfitting).

Model 3 (Random Forest): Pivoted to a non-linear machine learning model that successfully fit the data and identified the true, complex drivers of yield.

Tech Stack

R: tidyverse (for data manipulation and visualization), sqldf (for SQL queries), knitr (for report formatting), randomForest (for modeling).

SQL: Used within R via sqldf for exploratory data analysis.

Files

Agribusiness_Report_HTML.html: The formatted, interactive report (linked above).

Agribusiness_Report_MD.md: The basic GitHub-flavored markdown report.

Agribusiness_Report.Rmd: The R Markdown source code for the main report.

agriculture_dataset.csv: The raw data used for the analysis.
