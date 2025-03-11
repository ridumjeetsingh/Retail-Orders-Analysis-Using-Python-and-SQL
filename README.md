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

## Code Explanation
### Step 1: Import Libraries and Download Dataset
- Python Code
```
import kaggle
!kaggle datasets download ankitbansal06/retail-orders -f orders.csv
```

- Imports the Kaggle library to download data.
- Downloads the dataset orders.csv from the specified dataset.

### Step 2: Extract Zip File
```
import zipfile
zip_ref = zipfile.ZipFile('orders.csv.zip') 
zip_ref.extractall()  
zip_ref.close()  
```
- Opens the zip file named 'orders.csv.zip'.
- Extracts all its contents to the current directory.
- Closes the zip file to free resources.
### Step 3: Load Data and Handle Null Values
```
import pandas as pd
df = pd.read_csv('orders.csv', na_values=['Not Available', 'unknown'])
df['Ship Mode'].unique()
```
- Imports Pandas for data manipulation.
- Reads the CSV file and treats 'Not Available' and 'unknown' as NaN (null values).
- Displays the unique values in the 'Ship Mode' column.

### Step 4: Clean Column Names
```
# df.rename(columns={'Order Id':'order_id', 'City':'city'})
# df.columns = df.columns.str.lower()  
# df.columns = df.columns.str.replace(' ', '_')  
df.head(5)
```
- Initially commented out. These lines would:
- Standardize column names by converting them to lowercase and replacing spaces with underscores.
- The df.head(5) command displays the first five rows of the data.
### Step 5: Derive New Columns
```
df['profit'] = df['sale_price'] - df['cost_price']
df
Calculates the 'profit' column by subtracting 'cost_price' from 'sale_price'.
```
- Displays the updated dataframe.
### Step 6: Convert Date Column
```
df['order_date'] = pd.to_datetime(df['order_date'], format="%Y-%m-%d")
```
- Converts the 'order_date' column from string format to datetime format for easier analysis.
### Step 7: Drop Unnecessary Columns
```
df.drop(columns=['list_price', 'cost_price', 'discount_percent'], inplace=True)
```
- Drops the columns 'list_price', 'cost_price', and 'discount_percent' to remove redundant data.
### Step 8: Load Data into SQL Server
```
import sqlalchemy as sal
engine = sal.create_engine('mssql://ANKIT\SQLEXPRESS/master?driver=ODBC+DRIVER+17+FOR+SQL+SERVER')
conn = engine.connect()
```
- Imports SQLAlchemy to connect to an SQL Server database.
- Establishes a connection with the specified SQL Server instance.
### Step 9: Insert Data into SQL Table
```
df.to_sql('df_orders', con=conn, index=False, if_exists='append')
Inserts the cleaned dataframe (df) into the SQL table 'df_orders'.
```
- The if_exists='append' argument ensures new data is added to the existing table.
Commented Code with Improvements

## - After That Load this data to SQL Server Management Studio to find out the final Insghits.

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

## Top 10 Highest Revenue Generating Products

1. Top 5 Best-Selling Products in Each Region

2. Month-over-Month Growth for 2022 vs 2023

3.  Best Performing Sales Month for Each Category

4. Sub-Category with Highest Profit Growth in 2023

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

