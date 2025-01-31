# Walmart Sales Analysis using SQL

## Objective
The objective of this project is to delve into the Walmart Sales data, analyzing top-performing branches and products, examining the sales trends of various products, and understanding customer behavior. The goal is to explore opportunities for enhancing and optimizing sales strategies. The dataset utilized in this analysis was sourced from the Kaggle Walmart Sales Forecasting Competition.

## Key Findings & Takeaways for Top Management for the below mentioned analysis:
### 1. Top-Performing Product Lines & Revenue Contribution**
- **Top-Performing Product Lines with the largest revenue?** Helps top management focus on what’s driving the most revenue. This insight informs decisions around inventory, marketing, and promotions.

  **Fashion Accessories** is the top performing product line followed by **Food & Beverages** and **Electronic Accessories**, where as **Food & Beverages** drives the **largest revenue**.
-  **What product line had the largest VAT?** Highlights the product lines with the highest tax burden, which could inform pricing strategies or cost optimization efforts.

   **Home & lifestyle** has the largest VAT average of **16.03%**
### 2. Revenue Insights by Time Period
- **What is the total revenue & largest COGS by month?**

  **January** month had the largest revenue of **1,16,291** and largest COGS of **1,10,754**. Management can plan budgeting, resource allocation, and promotional planning for the month of
     january for increase in sales.
### 3. Customer Segmentation Insights

- **Which customer type brings the most revenue?**
**Female** is the most valuable customer type and the management must tailor marketing efforts and loyalty programs for the most valuable customers.
### 4. Branch/Location Performance

- **What is the city with the largest revenue?**
  City **Naypyitaw** brings in the largest revenue with **1,10,490** followed by **Yangon** with **1,05,861** and **Mandalay** with **1,04,534**. Management must allow better resource allocation, targeted promotions, and stocking strategies for these locations with the highest potential.
### 5. Financial Forecasting and Tax Strategy

- **What is the most common payment method?**
  **Cash payment** is the most common payment method. Understanding the most commonly used payment method can help optimize payment processing and reduce associated transaction costs.

## Takeaways:
- **Revenue Drivers:** Understanding the most profitable product lines, customer types, and regions.
- **Operational Efficiency:** Insights on COGS, VAT, and sales trends that highlight areas to optimize costs.
- **Customer Satisfaction:** Focus on ratings and feedback to improve products and services, with a focus on customer segments and peak periods.
- **Strategic Resource Allocation:** Data on time-of-day, city, and branch performance guides better allocation of resources and marketing efforts.

## Project Structure
- **Database Creation:** Created a database named WalmartSalesData.
- **Table Creation:** Created tables for Sales.
```sql
CREATE DATABASE IF NOT EXISTS WalmartSalesData;

CREATE TABLE IF NOT EXISTS sales (
	invoice_id VARCHAR (30) NOT NULL PRIMARY KEY,
	branch VARCHAR(5) NOT NULL,
    city VARCHAR(30) NOT NULL,
    customer_type VARCHAR(30) NOT NULL,
    gender VARCHAR(10) NOT NULL,
    product_line VARCHAR(100) NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,
    quantity INT NOT NULL,
    vat FLOAT(6,4) NOT NULL,
    total DECIMAL(12,4) NOT NULL,
    date DATETIME NOT NULL,
    time TIME NOT NULL,
    payment_method VARCHAR(15) NOT NULL,
    cogs DECIMAL(10,2) NOT NULL,
    gross_margin_percentage FLOAT(11,9),
    gross_income DECIMAL(12,4),
    rating FLOAT (2,1)
);
```
## Feature Engineering
**Time of date**
```sql
SELECT
	time,
    (CASE
		WHEN time BETWEEN "00:00:00" AND "12:00:00" THEN "Morning"
        WHEN time BETWEEN "12:01:00" AND "16:00:00" THEN "Afternoon"
        ELSE "Evening"
	END
    ) AS time_of_date
FROM sales;

ALTER TABLE sales ADD COLUMN time_of_date VARCHAR(20);

UPDATE sales
SET time_of_date = (
	CASE
		WHEN time BETWEEN "00:00:00" AND "12:00:00" THEN "Morning"
        WHEN time BETWEEN "12:01:00" AND "16:00:00" THEN "Afternoon"
        ELSE "Evening"
	END
);
```

**Day Name**
```sql
SELECT
	date,
    DAYNAME(date) AS day_name
FROM sales;

ALTER TABLE sales ADD COLUMN day_name VARCHAR(10);

UPDATE sales
SET day_name = DAYNAME(date);
```
**Month Name**
```sql
SELECT MONTHNAME(date) AS month_name
FROM sales;

ALTER TABLE sales ADD COLUMN month_name VARCHAR(10);

UPDATE sales
SET month_name = MONTHNAME(date);
```
## Generic
**How many unique cities does the data have?**
```sql
SELECT 
	DISTINCT  city 
FROM sales;

------ In which city is each branch?
SELECT 
	DISTINCT  branch 
FROM sales;

SELECT 
	DISTINCT city,
    branch
FROM sales;
```
## Product
**How many unique product lines does the data have?**
```sql
SELECT 
	COUNT(DISTINCT product_line)
FROM sales;

-----What is the most common payment method?
SELECT
	payment_method,
	COUNT(payment_method) AS cnt
FROM sales
GROUP BY payment_method 
ORDER BY CNT DESC;
```
**What is the most selling product line?**
```sql
 SELECT 
	product_line,
    COUNT(product_line) AS cnt_pl
 FROM sales
 GROUP BY product_line
 ORDER BY cnt_pl DESC;
 ```
**What is the total revenue by month?**
```sql
SELECT 
	month_name AS month,
    SUM(total) as total_revenue
FROM sales
GROUP BY month_name
ORDER BY total_revenue Desc;
```
**What month had the largest COGS?**
```sql
SELECT 
	month_name,
    SUM(cogs) as monthly_cogs
FROM sales
GROUP BY month_name
ORDER BY monthly_cogs DESC;
```
**What product line had the largest revenue?**
```sql
SELECT 
	product_line,
    SUM(total) as total_revenue
FROM sales
GROUP BY product_line
ORDER BY total_revenue DESC;
```
**What is the city with the largest revenue?**
```sql
SELECT 
	branch,
	city,
    SUM(total) as total_revenue
FROM sales
GROUP BY city, branch
ORDER BY total_revenue DESC;
```
**What product line had the largest VAT?**
```sql
SELECT 
	product_line,
    AVG(vat) as avg_tax
FROM sales
GROUP BY product_line
ORDER BY avg_tax DESC;
```
**Fetch each product line and add a column to those product line showing "Good", "Bad". Good if its greater than average sales**
```sql
SELECT
	product_line,
    total,
    CASE
		WHEN total > (SELECT AVG(total) FROM sales) then 'Good'
        ELSE 'Bad' 
	END AS line_type
FROM sales;
```
**Which branch sold more products than average product sold?**
```sql
SELECT
	branch,
    SUM(quantity) as sum_qty
FROM sales
GROUP BY branch
HAVING SUM(quantity) > (SELECT AVG(quantity) FROM sales);
```
**What is the most common product line by gender?**
```sql
SELECT 
	gender,
    product_line,
    COUNT(product_line) AS total_cnt
 FROM sales
 GROUP BY gender, product_line
 ORDER BY total_cnt DESC;
```
 **What is the average rating of each product line?**
 ```sql
 SELECT 
 product_line,
 ROUND(AVG(rating), 2) as avg_rtg
 FROM sales
 GROUP BY product_line
 ORDER BY avg_rtg DESC;
```
## Sales
**Number of sales made in each time of the day per weekday**
```sql
SELECT 
	time_of_date,
    COUNT(*) AS total_sales
FROM sales
WHERE day_name = "Monday"
GROUP BY time_of_date;
```
**Which of the customer types brings the most revenue?**
```sql
SELECT 
	customer_type,
    SUM(total) AS total_revenue
FROM sales
Group by customer_type
ORDER BY total_revenue DESC;
```
**Which city has the largest tax percent/ VAT (Value Added Tax)?**
```sql
SELECT 
	city,
    AVG(vat) AS VAT
FROM sales
GROUP BY city
ORDER BY VAT DESC;
```
**Which customer type pays the most in VAT?**
```sql
SELECT 
	customer_type,
    avg(vat) AS vat
FROM sales
Group by customer_type
ORDER BY vat DESC;
```
## Customer
**How many unique customer types does the data have?**
```sql
SELECT
	DISTINCT customer_type
FROM sales;
```
**How many unique payment methods does the data have?**
```sql
SELECT
	DISTINCT payment_method
FROM sales;
```
**What is the most common customer type?**
```sql
SELECT
	customer_type,
    COUNT(customer_type) AS CT_type
FROM sales
GROUP BY customer_type
ORDER BY CT_type;
```
**Which customer type buys the most?**
```sql
SELECT
	customer_type,
    COUNT(*)
FROM sales
GROUP BY customer_type;
```
**What is the gender of most of the customers?**
```sql
SELECT 
	gender,
    COUNT(gender) AS cnt
FROM sales
GROUP BY gender
ORDER BY cnt;
```
**What is the gender distribution per branch?**
```sql
SELECT 
	gender,
    COUNT(gender) AS cnt
FROM sales
WHERE branch = "A"
GROUP BY gender
ORDER BY cnt;
```
**Which time of the day do customers give most ratings?**
```sql
SELECT 
	time_of_date,
    AVG(rating) AS total_rtg
FROM sales
GROUP BY time_of_date
ORDER BY total_rtg DESC;
```
**Which time of the day do customers give most ratings per branch?**
```sql
SELECT 
	time_of_date,
    AVG(rating) AS total_rtg
FROM sales
WHERE branch = "B"
GROUP BY time_of_date
ORDER BY total_rtg DESC;
```
**Which day fo the week has the best avg ratings?**
```sql
SELECT
	day_name,
	AVG(rating) AS avg_rtg
FROM sales
GROUP BY day_name
ORDER BY avg_rtg DESC;
```
**Which day of the week has the best average ratings per branch?**
```sql
SELECT
	day_name,
	AVG(rating) AS avg_rtg
FROM sales
WHERE branch = "A"
GROUP BY day_name
ORDER BY avg_rtg DESC; 
```
  
