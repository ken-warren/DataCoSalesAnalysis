# DataCo Sales Analysis

## Table of Contents

- [Project Overview](#project-overview)
- [Business Questions](#business-questions)
- [Tools](#tools)
- [Data Source & Structure](#data-source--structure)
- [SQL Data Analysis](#sql-data-analysis)
- [ETL Pipeline](#etl-pipeline)
  - [Extraction](#extraction)
  - [Transformation](#transformation) 
  - [Loading](#loading)
- [Database Schema](#database-schema)
- [Power BI Analysis](#power-bi-analysis)
  - [Modeling](#modeling)
  - [Measures (DAX)](#measures-dax)
- [Findings](#findings)
- [Conclusion](#conclusion)
- [Recommendations](#recommendations)
- [Reference](#reference)

## Project Overview
This project presents a comprehensive DataCo Sales Analysis using SQL and Power BI. The goal is to transform raw sales data (sourced from [Kaggle](https://www.kaggle.com/datasets/shashwatwork/dataco-smart-supply-chain-for-big-data-analysis) into a clean, analytical data model through the design of fact and dimension tables, followed by the creation of insightful Power BI dashboards to derive insights.

The main objective of this analysis is to provide a 360° view of DataCo’s sales performance, exploring key drivers of business growth and decline across five strategic dimensions - sales and profitability trends, customer behavior, product performance, operational efficiency, and regional market dynamics.

## Business Questions
### 1. Sales & Profitability Trends
- What are the total sales by region, state, and country?
- What are the sales trends by month, quarter, and year?
- How do different payment types contribute to sales?
- What is the relationship between discounts and sales volume?

### 2. Customer Insights
- Who are our most valuable customers (Customer Lifetime Value)?
- How are customers distributed across cities, states, and countries?
- Which customer segments (Consumer, Corporate, Home Office) generate the most sales and profit?
- Which segments are declining over time (churn rate analysis)?

### 3. Product Performance
- What is the profit per order, product, and category?
- Which products are highly profitable vs. loss-making?
- Do discounted products generate more sales but lower profit margins?
- Which products are top-selling or slow-moving?

### 4. Shipping & Delivery Efficiency
- What is the relationship between delivery speed and sales or profit?
- What is the average scheduled vs. actual shipping time?
- What percentage of orders are delivered late?
- Which regions or shipping modes have the highest late delivery rates?

### 5. Regional Market Dynamics
- What is the market share of each region (APAC, EU, US, etc.)?
- Which regions are growing the fastest in terms of sales?
- How do profits and sales vary geographically across states and cities?

## Tools
1. SQL – Data cleaning, transformation, and relational modeling (fact & dimension tables)
2. Power BI – Data connection, DAX calculations, and dashboard visualization
3. Excel – Initial data validation and exploration

## Data Source & Structure
The dataset used in this project was obtained from [Kaggle’s DataCo Supply Chain dataset](https://www.kaggle.com/datasets/shashwatwork/dataco-smart-supply-chain-for-big-data-analysis), which contains detailed records of customer orders, product categories, shipping details, payment methods, and profitability metrics.

### SQL Data Analysis
A database was created in SQL and raw data loaded in for normalization.

```
CREATE DATABASE sales_db;

USE sales_db;

-- Create a table to store the raw data with their respective data types
CREATE TABLE RawData (
    `Type` VARCHAR(50),
    Days_for_shipping_real INT,
    Days_for_shipment_scheduled INT,
    Benefit_per_order DECIMAL(10,2),
    Sales_per_customer DECIMAL(10,2),
    Delivery_Status VARCHAR(50),
    Late_delivery_risk TINYINT,
    Category_Id INT,
    Category_Name VARCHAR(100),
    Customer_City VARCHAR(100),
    Customer_Country VARCHAR(100),
    Customer_Id INT,
    Customer_Name VARCHAR(150),
    Customer_Segment VARCHAR(100),
    Customer_State VARCHAR(100),
    Department_Id INT,
    Department_Name VARCHAR(100),
    Latitude DECIMAL(15,11),
    Longitude DECIMAL(15,11),
    Market VARCHAR(100),
    Order_City VARCHAR(100),
    Order_Country VARCHAR(100),
    Order_Customer_Id INT,
    Order_Date DATE,
    Order_Id INT,
    Order_Item_Cardprod_Id INT,
    Order_Item_Discount DECIMAL(10,2),
    Order_Item_Discount_Rate DECIMAL(5,2),
    Order_Item_Id INT,
    Order_Item_Product_Price VARCHAR(20),
    Order_Item_Profit_Ratio VARCHAR(20),
    Order_Item_Quantity INT,
    Sales DECIMAL(10,2),
    Order_Item_Total DECIMAL(10,2),
    Order_Profit_Per_Order DECIMAL(10,2),
    Order_Region VARCHAR(100),
    Order_State VARCHAR(100),
    Order_Status VARCHAR(50),
    Product_Card_Id INT,
    Product_Category_Id INT,
    Product_Name VARCHAR(255),
    Product_Price DECIMAL(10,2),
    Product_Status VARCHAR(50),
    Shipping_Date DATE,
    Shipping_Mode VARCHAR(50)
);
TRUNCATE TABLE rawdata; -- Just to make sure you don't load the table twice

-- To load bulky data fast
LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/Data_Co.csv'
INTO TABLE rawdata
FIELDS TERMINATED BY ','
IGNORE 1 LINES;

```
The raw data was broken down into dimension tables before connecting the database with Power BI for further analysis.
```
-- DIMENSION TABLES

-- 1. Customer Dimension
CREATE TABLE dim_customer (
    customer_id INT AUTO_INCREMENT PRIMARY KEY,
    customer_original_id VARCHAR(50),
    customer_name VARCHAR(255) NOT NULL,
    customer_segment VARCHAR(50),
    customer_city VARCHAR(100),
    customer_state VARCHAR(100),
    customer_country VARCHAR(100),
    UNIQUE KEY unique_customer (customer_original_id)
);

-- 2. Product Dimension
CREATE TABLE dim_product (
    product_id INT AUTO_INCREMENT PRIMARY KEY,
    product_card_id VARCHAR(50),
    product_name VARCHAR(255) NOT NULL,
    product_price DECIMAL(10,2),
    product_status VARCHAR(50),
    product_category_id INT,
    category_name VARCHAR(100),
    UNIQUE KEY unique_product (product_card_id)
);

-- 3. Department/Category Dimension
CREATE TABLE dim_department (
    department_id INT AUTO_INCREMENT PRIMARY KEY,
    department_original_id VARCHAR(50),
    department_name VARCHAR(100) NOT NULL,
    category_id VARCHAR(50),
    category_name VARCHAR(100),
    UNIQUE KEY unique_department (department_original_id, category_id)
);

-- 4. Geography Dimension
CREATE TABLE dim_geography (
    geography_id INT AUTO_INCREMENT PRIMARY KEY,
    city VARCHAR(100),
    state VARCHAR(100),
    country VARCHAR(100),
    latitude DECIMAL(15,11),
    longitude DECIMAL(15,11),
    market VARCHAR(50),
    UNIQUE KEY unique_location (city, state, country)
);

-- FACT TABLE
-- Main Fact Table: Order Items (Grain: one row per order item)
CREATE TABLE fact_order_items (
    order_item_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    
    -- Foreign Keys to Dimensions
    customer_key INT NOT NULL,
    product_key INT NOT NULL,
    department_key INT NOT NULL,
    order_geography_key INT NOT NULL,
    shipping_geography_key INT,
    
    -- Date columns (kept in fact table)
    order_date DATE NOT NULL,
    shipping_date DATE,
    
    -- Shipping details directly in fact table
    shipping_mode VARCHAR(50) NOT NULL,
    
    -- Degenerate Dimensions
    order_id VARCHAR(50) NOT NULL,
    order_item_original_id VARCHAR(50),
    order_item_cardprod_id VARCHAR(50),
    type VARCHAR(50),
    
    -- Facts
    days_for_shipping_real INT,
    days_for_shipment_scheduled INT,
    benefit_per_order DECIMAL(10,2),
    sales_per_customer DECIMAL(10,2),
    delivery_status VARCHAR(50),
    late_delivery_risk INT,
    order_item_discount DECIMAL(10,2),
    order_item_discount_rate DECIMAL(5,4),
    order_item_product_price DECIMAL(10,2),
    order_item_profit_ratio DECIMAL(5,4),
    order_item_quantity INT,
    sales DECIMAL(10,2),
    order_item_total DECIMAL(10,2),
    order_profit_per_order DECIMAL(10,2),
    order_region VARCHAR(50),
    order_state VARCHAR(100),
    order_status VARCHAR(50),
    
    -- Indexes for performance
    INDEX idx_customer (customer_key),
    INDEX idx_product (product_key),
    INDEX idx_order_date (order_date),
    INDEX idx_order_id (order_id),
    INDEX idx_shipping_date (shipping_date),
    INDEX idx_shipping_mode (shipping_mode),
    
    -- Foreign Key Constraints (no shipping dimension FK)
    FOREIGN KEY (customer_key) REFERENCES dim_customer(customer_id),
    FOREIGN KEY (product_key) REFERENCES dim_product(product_id),
    FOREIGN KEY (department_key) REFERENCES dim_department(department_id),
    FOREIGN KEY (order_geography_key) REFERENCES dim_geography(geography_id)
);
```
### Data Structure:
The raw dataset was cleaned and transformed into a Star Schema model to optimize reporting and analysis in Power BI. The final structure includes:
- Fact Table:
  - Fact_Sales – Contains transaction-level data such as Order ID, Product ID, Customer ID, Quantity, Sales, Profit, Discount, Shipping Cost, and Order Date.

- Dimension Tables:
  - Dim_Customer – Customer demographics, segment, and market information.
  - Dim_Product – Product name, category, sub-category, and related details.
  - Dim_Geography – City, state, country, and regional hierarchy (APAC, EU, US, etc.).
  - Dim Departments- Business departments
  - Dim_Categories
  - Date table
  
 ### Database Schema
 
 [image of our data model]

 ### Power BI Analysis
 
 #### Modeling
 
 #### Measures (DAX)
 Base Measures
 - Total Sales 
```
Total Sales = SUM(Sales[Order_Item_Total])
```
- Total Profit
```
Total Profit = SUM(Sales[Benefit Per Order])
```
- Total Quantity
```
Total Quantity = SUM(Sales[Order_Item_Quantity])
```
- Total Orders
```
Total Orders = DISTINCTCOUNT(Sales[Order ID])
```
- Total Customers
```
Customers = DISTINCTCOUNT(Sales[Customer ID])
```
Previous Year /Growth Measure
- Sales Previous Year
```
Sales LY = 
VAR Curr_Year = [Total Sales]
VAR PY = CALCULATE([Total Sales],SAMEPERIODLASTYEAR('Date'[Date]))
RETURN PY
```
- Sales Growth
```
YoY Sales % = 
VAR Curr_Yr = [Total Sales]
VAR Prev_Yr = CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date]))
VAR YoY_Change = DIVIDE(Curr_Yr - Prev_Yr, Prev_Yr)

RETURN
SWITCH(
    TRUE(),
    YoY_Change > 0, UNICHAR(9650) & " " & FORMAT(YoY_Change, "0.0%"),   // ▲
    YoY_Change < 0, UNICHAR(9660) & " " & FORMAT(YoY_Change, "0.0%"),   // ▼
    FORMAT(YoY_Change, "0.0%")
)

```
Additional Measures

- Average Discount
```
Avg Discount Rate = AVERAGE(Sales[Order_Item_Discount_Rate])
```
- Profit Per Order
```
Profit Per Order = AVERAGE (Sales[Order_Profit_Per_Order])
```
- Return on Investment (ROI)
```
ROI =  DIVIDE(([Total Profit]-[Total Discount]),[Total Discount])
```
- Churn rate measures
```
Lost Customers = 
VAR CurrCust = 
    CALCULATETABLE(
        VALUES(Sales[Customer ID]),
        DATESINPERIOD('Date'[Date], MAX('Date'[Date]), -1, MONTH)
    )
VAR PrevCust = 
    CALCULATETABLE(
        VALUES(Sales[Customer ID]),
        PREVIOUSMONTH('Date'[Date])
    )
RETURN
COUNTROWS(
    EXCEPT(PrevCust, CurrCust)
)

Previous Period Customers = 
CALCULATE(
    DISTINCTCOUNT(Sales[Customer ID]),
    DATESINPERIOD(
        'Date'[Date],
        MIN('Date'[Date]) - 1,
        -1,
        MONTH
    )
)

Churn Rate = 
DIVIDE(
    [Lost Customers],
    [Previous Period Customers]
)

```

- Customer retention percentage measure
```
Customer Retention % = 
VAR currMonthAfter = SELECTEDVALUE('Months After'[Month])
VAR RetainedCustomers = 
    CALCULATE(
        DISTINCTCOUNT(Sales[Customer ID]),
        FILTER(
            Sales,
            VAR customerFirstOrderMonth = 
                RELATED(Customers[First Order Month End])
            RETURN
            EOMONTH(customerFirstOrderMonth,currMonthAfter) = EOMONTH(Sales[Order Date], 0)
        )
    )
VAR TotalCohortCustomers = 
    CALCULATE(
        DISTINCTCOUNT(Sales[Customer ID]),
        FILTER(
            Sales,
            VAR customerFirstOrderMonth = 
                RELATED(Customers[First Order Month End])
            RETURN
            EOMONTH(customerFirstOrderMonth, 0) = EOMONTH(Sales[Order Date], 0)
        )
    )
RETURN
DIVIDE(RetainedCustomers, TotalCohortCustomers, 0)
```
