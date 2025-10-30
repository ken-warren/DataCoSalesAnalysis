# DataCo Sales Analysis

## Table of Contents

- [Project Overview](#project-overview)
- [Business Questions](#business-questions)
- [Tools](#tools)
- [Data Source & Structure](#data-source--structure)
- [SQL Data Analysis](#sql-data-analysis)
- [Database Schema](#database-schema)
- [Power BI Analysis](#power-bi-analysis)
  - [Modeling](#modeling)
  - [Measures (DAX)](#measures-dax)
- [Findings](#findings)
  - [Sales](#sales)
  - [Customers](#customers)
  - [Products](#products)
  - [Shipping](#shipping)
  - [Regions](#regions)
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
  - Sales – Contains transaction-level data such as Order ID, Product ID, Customer ID, Quantity, Sales, Profit, Discount, Shipping Cost, and Order Date.

- Dimension Tables:
  - Customer – Customer demographics, segment, and market information.
  - Product – Product name, category, sub-category, and related details.
  - Order/Store Location – City, state, country, and regional hierarchy (APAC, EU, US, etc.).
  - Departments - Business departments
  - Categories - Product categories
  - Date table
  
 ### Database Schema
 
   <img width="1047" height="802" alt="Data Model" src="https://github.com/user-attachments/assets/04e59623-6804-4180-8bbf-edbb6dfd6abc" />
 
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
Calculated columns and measures for cohort analysis.
- First Order Date(End of Month)
```
First Order Month End = 
VAR _FirstOrderDate =
    CALCULATE(
        MIN(Sales[Order Date]),
        FILTER(Sales, Sales[Customer ID] = VALUE(EARLIER(Customers[Customer ID])))
    )
RETURN
    FORMAT(EOMONTH(_FirstOrderDate, 0),"MMM YYYY")

```
- Last Order Date(End of Month)
```
Last Order Month End = 
VAR _FirstOrderDate =
    CALCULATE(
        MAX(Sales[Order Date]),
        FILTER(Sales, Sales[Customer ID] = VALUE(EARLIER(Customers[Customer ID])))
    )
RETURN
    FORMAT(EOMONTH(_FirstOrderDate, 0),"MMM YYYY")

```
- Retention Period
```
Retention Period = 
ABS(
  DATEDIFF(Customers[First Order Month End],Customers[Last Order Month End],DAY)
  )
```
- Days Since Last Purchase
```
Days Since Last Purchase = 
DATEDIFF(
    [Last Order Month End],
   EOMONTH( MAX('Date'[Date]),1),
    DAY
)

```
- Churn Classification
```
Churned? = 
VAR _churnThreshold = 180
RETURN
SWITCH(
    TRUE(),
    [Days Since Last Purchase] <= _churnThreshold, "Active",
    "Churned"
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

## Findings

#### 1. Sales

  <img width="1496" height="842" alt="Sales" src="https://github.com/user-attachments/assets/6377650d-61e2-410d-9359-730651920e1a" />

- Our total sales are $33.05M post-discount with the European Market contributing to 30% of the overall sales from 2015 - 2018. The chart below shows sales by market:

  <img width="683" height="550" alt="Screenshot 2025-10-30 142912" src="https://github.com/user-attachments/assets/53c0501d-ece3-4991-a5ac-4697124b6d42" />

- Generally, sales are seasonal with peaks in January and October, and the lowest sales are recorded in February and November. An overall consistent trend observed through the rest of the months. 
  This trend is,however, not consistent with different peaks and troughs observed every year.

  <img width="650" height="268" alt="Screenshot 2025-10-30 143900" src="https://github.com/user-attachments/assets/27c17315-2514-4584-a706-3a053fd6edaf" />

- Debit payment mode is the most popular and accounts for 38% of sales ~ $12M
  Cash is the least popular payment mode at 11% - $3.62M.

- The average discount rate across all categories is ~10% with a slight increase in sales volume over the years corresponding to the slight increase in discount. The increasing sales volumes are, however, not directly linked to increasing discounts.

#### 2. Customers

  <img width="1492" height="839" alt="Customers" src="https://github.com/user-attachments/assets/59222acc-a8b5-4c77-8d7f-fc46a588384b" />

- Our top 5 customers revenue-wise are Smith Mary, Smith Robert, Smith James, Smith David and Smith John. Only a small percentage of customers contribute to the overall sales.

   <img width="446" height="297" alt="Screenshot 2025-10-30 144848" src="https://github.com/user-attachments/assets/310ef27b-a8a0-4f3c-8366-183de0d07521" />

-  The USA, France and Mexico account for 69% of DataCo’s customers(~14k)and about 28% of sales from 2015 to 2018.

    <img width="686" height="400" alt="Screenshot 2025-10-30 150528" src="https://github.com/user-attachments/assets/81436bc9-7598-4b50-9148-8d1915e2b5f5" />

- The consumer segment contributes $17M in sales(51% of sales ),Corporate $10M and Home office $6M.

- Over 70% of new customers don't return following their initial order. Acquisition is followed by a high churn rate.
  The cohort analysis table below shows a glimpse of customer behaviour over 12 months following the first order:
  
  <img width="637" height="453" alt="Screenshot 2025-10-24 115205" src="https://github.com/user-attachments/assets/6b12e317-f520-477e-9f5f-b860211acb60" />

#### 3. Products

  <img width="1493" height="837" alt="Products" src="https://github.com/user-attachments/assets/a710b6b1-a79b-45b9-9c73-e8ba0c92e754" />

- Profit varies for different product categories with the highest profit recorded on fishing products($756,220) after a discount of 10.17%.
  Strength training products record the lowest profit at $332 after a discount of 10.5%.

- Fishing — $756K profit, Cleats — $495K profit and Camping & Hiking — $427K profit.
  These categories drive both high revenue and strong profit, with Fishing leading overall.
  
- Music ($33.26 profit/order), Sporting Goods ($35.07) and Children’s Clothing ($41.68) categories yield low profit per order, suggesting slower movement or lower volume.

#### 4. Shipping

  <img width="1493" height="837" alt="Shipping" src="https://github.com/user-attachments/assets/60c56063-fbf6-4d4f-a261-6d76103fe2c8" />

- Shipping delay is experienced across all shipping modes and markets and no direct effect on sales/profit is observed.
  It could however be attributed to the loss of customers over time.

- 55% of orders are delivered late across all markets
  The average scheduled delivery time is 70hrs, while the average actual delivery time is 84hrs, presenting an average delay of 19%.

- LATAM and Europe have the highest delay risk at 28% but they also have more orders compared to regions like Africa which has a 6% risk and the lowest number of orders.
  Standard class has the timely delivery while first class and second class have a 24-48 hour delay respectively.

#### 5. Regions

  <img width="1492" height="837" alt="Regions" src="https://github.com/user-attachments/assets/0f1de658-3752-4ad6-963d-8f8062d2f1a6" />

- DataCo’s sales are at $33.05M (2015-2018)
  Europe accounts for 30%, LATAM 28%, Pacific Asia 22%, USCA 12% and Africa 6%.

- Pacific Asia shows the highest YoY Sales Growth of 4.2% which can be attributed to the customer growth of 19.1%.

- Profit vs. sales: Similar margins are observed across regions, but absolute profits follow total sales - led by Europe and LATAM, with Africa trailing.

## Conclusion

From 2015 to 2018, DataCo recorded **$33.05M** in total sales, with Europe (**30%**) and LATAM (**28%**) leading contributions, and Pacific Asia showing the highest growth (**4.2%**) due to customer expansion. Despite an average **10%** discount rate, higher discounts did not significantly boost sales, suggesting growth was driven by customer reach rather than pricing. Sales peaked in **January** and **October**, reflecting seasonal demand patterns, and debit payments dominated (**38%**), indicating strong digital payment adoption.

The consumer segment accounted for **51%** of total sales, but over **70%** of new customers failed to return, signaling high churn. Profits were highest in **Fishing**, **Cleats**, and **Camping & Hiking** categories, while categories like **Music** and **Strength Training** underperformed. Shipping delays affected over half of all orders, particularly in Europe and LATAM, averaging a **19%** delay. Overall, DataCo shows solid regional diversification and high-growth potential, but challenges in **customer retention**, **logistics reliability**, and **product profitability** remain key areas for improvement.

## Recommendations  

### 1. Customer Retention & Churn Reduction  
- Implement retention campaigns: loyalty programs, personalized emails, and post-purchase engagement.  
- Analyze customer cohorts to identify high-potential repeat buyers. 
- Develop a **churn prediction model** using variables such as order delay, product category, and discount rate to determine have a higher risk of churning.
- Reallocate part of the acquisition budget toward retention - retaining customers is 5-25 times cheaper than acquiring new ones.


### 2. Logistics & Delivery Optimization  
- Investigate root causes of delays by carrier, route, and region.  
- Introduce **real-time tracking** and proactive communication to mitigate negative perceptions.  
- Consider **regional fulfillment hubs**, e.g, strategically located warehouses, in areas affected with high delivery risks like Europe and LATAM to shorten delivery time.

### 3. Product Portfolio Focus  
- Invest marketing and stock in high-margin categories (Fishing, Cleats, Camping & Hiking).  
- Reassess under-performing categories (Music, Strength Training, Children’s Clothing) for redesign, repositioning, or discontinuation.  
- Replace price-off discounts with **targeted or conditional discounts** (e.g., bundling, loyalty-based).


### 4. Market Strategy  
- Double down on **Pacific Asia** for expansion given its strong customer growth.  
- Maintain focus on **Europe and LATAM**, but improve delivery reliability.  
- For **Africa**, pilot local partnerships and logistics improvements before scaling.  
- Leverage **seasonal peaks** (January, October) for promotions and manage inventory to mitigate low seasons.


### 5. Payment Optimization  
- Continue promoting debit and card payments; explore **digital wallets** and **Buy-Now-Pay-Later** solutions.  
- Maintain minimal dependency on cash payments except where regionally necessary.  


### 6. Customer Lifetime Value (CLV) Tracking  
- Implement CLV models to quantify long-term customer profitability.  
- Develop retention dashboards in Power BI to track repeat purchase behavior and customer profitability by region and category.  
- Prioritize customers with the highest potential LTV for personalized offers and loyalty incentives.

## Reference

