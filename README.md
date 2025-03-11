# Retail Orders Analysis Using Python and SQL

## Overview

This project involves data cleaning, transformation, and insightful analysis using SQL queries. The dataset is sourced from Kaggle and demonstrates skills in data preparation, analysis, and database integration.

### Key Features

1. Data sourced directly from Kaggle

2. Data cleaning steps include handling null values, standardizing column names, and deriving new columns

3. SQL queries provide actionable insights such as top-performing products, regional sales leaders, and growth analysis

4. Data is loaded into SQL Server for scalable data management

## Technologies Used

1. Python (Pandas, SQLAlchemy, Zipfile)

2. Kaggle API

3. SQL Server

4. Jupyter Notebook

5. SQL Analysis Highlights

## Top 10 Highest Revenue Generating Products

1. Top 5 Best-Selling Products in Each Region

2. Month-over-Month Growth for 2022 vs 2023

3.  Best Performing Sales Month for Each Category

4. Sub-Category with Highest Profit Growth in 2023

## Setup Instructions

1. Install the required libraries:
``` 
pip install pandas sqlalchemy kaggle
```
1. Download the dataset using the Kaggle API.

2. Extract the contents of the zip file.

3. Clean the dataset following the steps in the code.

4. Establish a connection to your SQL Server.

5. Run the SQL queries to analyze the data.

```sql

-- Find top 10 highest revenue generating products
SELECT TOP 10 product_id, SUM(sale_price) AS sales
FROM df_orders
GROUP BY product_id
ORDER BY sales DESC;

-- Find top 5 highest selling products in each region
WITH cte AS (
    SELECT region, product_id, SUM(sale_price) AS sales
    FROM df_orders
    GROUP BY region, product_id)
SELECT * FROM (
    SELECT *, ROW_NUMBER() OVER(PARTITION BY region ORDER BY sales DESC) AS rn
    FROM cte) A
WHERE rn <= 5;

-- Find month-over-month growth comparison for 2022 and 2023 sales
WITH cte AS (
    SELECT YEAR(order_date) AS order_year, MONTH(order_date) AS order_month,
    SUM(sale_price) AS sales
    FROM df_orders
    GROUP BY YEAR(order_date), MONTH(order_date)
)
SELECT order_month,
    SUM(CASE WHEN order_year=2022 THEN sales ELSE 0 END) AS sales_2022,
    SUM(CASE WHEN order_year=2023 THEN sales ELSE 0 END) AS sales_2023
FROM cte
GROUP BY order_month
ORDER BY order_month;

-- Identify the month with highest sales for each category
WITH cte AS (
    SELECT category, FORMAT(order_date, 'yyyyMM') AS order_year_month,
    SUM(sale_price) AS sales
    FROM df_orders
    GROUP BY category, FORMAT(order_date, 'yyyyMM')
)
SELECT * FROM (
    SELECT *, ROW_NUMBER() OVER(PARTITION BY category ORDER BY sales DESC) AS rn
    FROM cte) a
WHERE rn = 1;

-- Identify which sub-category had the highest profit growth in 2023
WITH cte AS (
    SELECT sub_category, YEAR(order_date) AS order_year,
    SUM(sale_price) AS sales
    FROM df_orders
    GROUP BY sub_category, YEAR(order_date)
)
, cte2 AS (
    SELECT sub_category,
    SUM(CASE WHEN order_year=2022 THEN sales ELSE 0 END) AS sales_2022,
    SUM(CASE WHEN order_year=2023 THEN sales ELSE 0 END) AS sales_2023
    FROM cte
    GROUP BY sub_category
)
SELECT TOP 1 *
, (sales_2023 - sales_2022) AS growth
FROM cte2
ORDER BY growth DESC;
```

