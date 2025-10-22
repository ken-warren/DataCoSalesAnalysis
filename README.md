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

### Data Structure:
The raw dataset was cleaned and transformed into a Star Schema model to optimize reporting and analysis in Power BI. The structure includes:
- Fact Table:
  - Fact_Sales – Contains transaction-level data such as Order ID, Product ID, Customer ID, Quantity, Sales, Profit, Discount, Shipping Cost, and Order Date.

- Dimension Tables:
  - Dim_Customer – Customer demographics, segment, and market information.
  - Dim_Product – Product name, category, sub-category, and related details.
  - Dim_Shipping – Shipping mode, shipping cost, delivery status, and shipping duration.
  - Dim_Geography – City, state, country, and regional hierarchy (APAC, EU, US, etc.).

