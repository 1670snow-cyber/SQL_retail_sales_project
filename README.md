# SQL_retail_sales_project
This project explores the retail sales dataset using techniques typically used by data analysts to explore, clean, and analyze retail sales data. The project involves setting up a retail sales database, performing exploratory data analysis (EDA), and answering specific business questions through SQL queries.

**Objectives**
Set up a retail sales database: Create and populate a retail sales database with the provided sales data.
Data Cleaning: Identify and remove any records with missing or null values.
Exploratory Data Analysis (EDA): Perform basic exploratory data analysis to understand the dataset.
Business Analysis: Use SQL to answer specific business questions and derive insights from the sales data.

**Project Structure**

**1. Database Setup**
Database Creation: The project starts by creating a database named Retail_Sales_DB.
Table Creation: A table named retail_sales is created to store the sales data. The table structure includes columns for transaction ID, sale date, sale time, customer ID, gender, age, product category, quantity sold, price per unit, cost of goods sold (COGS), and total sale amount.

```sql
CREATE DATABASE Retail Sales DB;

--CREATE TABLE--
CREATE TABLE retail_sales (transactions_id	INT PRIMARY KEY,
                          sale_date	DATE,
                          sale_time	TIME,
                          customer_id INT,
                          gender VARCHAR(15),
                          age	INT,
                          category VARCHAR(15),
                          quantiy	INT,
                          price_per_unit FLOAT,
                          cogs	FLOAT,
                          total_sale FLOAT
                          ) ;
```
                          
**2. Data Exploration & Cleaning**
Record Count: Determine the total number of records in the dataset.
Customer Count: Find out how many unique customers are in the dataset.
Category Count: Identify all unique product categories in the dataset.
Null Value Check: Check for any null values in the dataset and delete records with missing data.
```sql -- VIEW COMPLETE DATA --
select * from retail_sales;
-- CONVERT TIME TO hh:mm:ss --

ALTER TABLE retail_sales
ALTER COLUMN sale_time TIME(0);
--VIEW THE DATA --
SELECT TOP 10 * FROM retail_sales;
SELECT count( * ) FROM retail_sales;

--DATA CLEANING--
--VIEW NULL VALUES--
select * from retail_sales
WHERE transactions_id IS NULL
    OR
    sale_date IS NULL
    OR 
    sale_time IS NULL
    OR
    gender IS NULL
    OR
    category IS NULL
    OR
    quantiy IS NULL
    OR
    cogs IS NULL
    OR
    total_sale IS NULL;
    

-- DELETE NULL VALUES --
 DELETE FROM retail_sales
 WHERE transactions_id IS NULL
    OR
    sale_date IS NULL
    OR 
    sale_time IS NULL
    OR
    gender IS NULL
    OR
    category IS NULL
    OR
    quantiy IS NULL
    OR
    cogs IS NULL
    OR
    total_sale IS NULL;

--CHECK THE NUMBER OF ROWS LEFT --

select count(*) from retail_sales;
```
**3. Data Analysis & Findings**
The following SQL queries were developed to answer specific business questions:
```sql
--HOW MUCH SALES DO WE HAVE?
SELECT COUNT(*) as Total_Sales FROM retail_sales;

--HOW MANY CUSTOMERS WE HAVE?
SELECT 
COUNT(DISTINCT customer_id) AS Total_Customers
FROM retail_sales;

--HOW MANY CATEGORIES WE HAVE?
SELECT 
COUNT(DISTINCT category) as Categories
from retail_sales;

--DATA ANALYSIS (BUSINESS KEY PROBLEMS AND ANSWERS)

--Q1.sql query to retrieve information of the sales made on 2022-11-05
SELECT * FROM retail_sales
WHERE sale_date='2022-11-05';

--Q2. retreive the transactions mase in the mont of Novemeber 2022 where the category is clothing and the quantity sold is more than or equal to 4
SELECT
    category ,
    SUM(quantiy) 
FROM retail_sales
GROUP BY category;


SELECT * 
FROM retail_sales 
WHERE category='Clothing'
  AND FORMAT(sale_date,'yyyy-MM')='2022-11'
  AND quantiy >= 4;

-- Q.3 Write a SQL query to calculate the total sales (total_sale) for each category.
SELECT
    category,
	sum(total_sale) as total_sales,
	count(*) as total_orders
FROM retail_sales
GROUP BY category;

-- Q.4 Write a SQL query to find the average age of customers who purchased items from the 'Beauty' category.
SELECT 
	ROUND(AVG(age *1.0),2) as average_age
FROM retail_sales
WHERE category='Beauty';


-- Q.5 Write a SQL query to find all transactions where the total_sale is greater than 1000.
SELECT * 
FROM retail_sales 
WHERE total_sale > 1000;

-- Q.6 Write a SQL query to find the total number of transactions (transaction_id) made by each gender in each category.
SELECT 
    category,
    gender,
    count(transactions_id) AS transactions
FROM retail_sales
GROUP BY gender,category
ORDER BY 1;

-- Q.7 Write a SQL query to calculate the average sale for each month. Find out best selling month in each year
SELECT * FROM
(
SELECT 
    DATEPART(year,sale_date) AS yr,
    DATENAME(month, sale_date) AS mnth,
	ROUND(AVG(total_sale),2) AS avg_sales,
	RANK() OVER (PARTITION BY DATEPART(year,sale_date) ORDER BY AVG(total_sale) DESC) AS rank
FROM retail_sales
GROUP BY DATENAME(month, sale_date),DATEPART(year,sale_date)
) AS t1
WHERE rank=1;

-- Q.8 Write a SQL query to find the top 5 customers based on the highest total sales 
SELECT
    TOP 5 customer_id ,
	sum(total_sale) AS total_sale
FROM retail_sales
GROUP BY customer_id
ORDER BY 2 DESC;

-- Q.9 Write a SQL query to find the number of unique customers who purchased items from each category.
SELECT
      category AS categories,
      COUNT(DISTINCT customer_id) AS cnt_unique_customers	
FROM retail_sales
GROUP BY category;
     
-- Q.10 Write a SQL query to create each shift and number of orders (Example Morning <12, Afternoon Between 12 & 17, Evening >17)
WITH Shift_sales
AS
(
SELECT *,
    CASE
	   WHEN DATEPART(HOUR,sale_time) < 12 THEN 'Morning'
	   WHEN DATEPART(HOUR,sale_time) BETWEEN 12 AND 17 THEN 'Afternoon'
	   ELSE 'Evening'
	END AS shift
FROM retail_sales
)
SELECT
    shift,
	COUNT(transactions_id) AS no_of_orders
FROM Shift_sales
GROUP BY shift;

--Q11. the age group where maximum orders were placed 
WITH age_sales
AS
(
SELECT *,
    CASE
	    WHEN age <20 THEN 'teenager'
		WHEN age between 20 AND 40 THEN 'middle-age'
		ELSE 'elderly'
	END AS age_group
FROM retail_sales
)
SELECT 
    age_group,
	COUNT(*) as Customers
FROM age_sales
GROUP BY age_group
ORDER BY Customers DESC;

--Q12. Which category of goods has the highest COGS and the corresponding sales 
SELECT 
    category,
	ROUND(SUM(cogs),2) AS Cost_of_goods_sold,
	SUM(total_sale) AS sales
FROM retail_sales
GROUP BY category
ORDER BY 2 DESC;

--Q13.Is there a strong correlation between quantity bought and profit per transaction?
SELECT
    quantiy,
	ROUND(AVG(total_sale-cogs),2) AS Profit
FROM retail_sales
GROUP BY quantiy
ORDER BY quantiy;

--Q14.Which category has the highest profit margin?
SELECT 
    category,
    ROUND(AVG(total_sale - cogs), 2) AS avg_profit,
    ROUND(AVG((total_sale - cogs) * 1.0 / NULLIF(cogs, 0)), 2) AS avg_margin_ratio
FROM retail_sales
GROUP BY category
ORDER BY avg_margin_ratio DESC;

--Q15. Which customers are repeat buyers and how much do they spend on average?
SELECT
    customer_id,
	COUNT(*) AS visits,
	ROUND(AVG(total_sale),2) AS avg_amount_spend
FROM retail_sales
GROUP BY customer_id
HAVING 2>1
ORDER BY 2 DESC;
```
**Findings**
Customer Demographics: The dataset includes customers from various age groups, with sales distributed across different categories such as Clothing and Beauty.
High-Value Transactions: Several transactions had a total sale amount greater than 1000, indicating premium purchases.
Sales Trends: Monthly analysis shows variations in sales, helping identify peak seasons.
Customer Insights: The analysis identifies the top-spending customers and the most popular product categories.

**Reports**
Sales Summary: A detailed report summarizing total sales, customer demographics, and category performance.
Trend Analysis: Insights into sales trends across different months and shifts.
Customer Insights: Reports on top customers and unique customer counts per category.

**Conclusion**
This project serves as a comprehensive introduction to SQL for data analysts, covering database setup, data cleaning, exploratory data analysis, and business-driven SQL queries. The findings from this project can help drive business decisions by understanding sales patterns, customer behavior, and product performance.
