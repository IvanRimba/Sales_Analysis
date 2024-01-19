# Sales analysis - Q1 2023
### Project Overview

This data analysis project delves into the sales perormance of a business in the first quarter of 2023. By examining the various aspects, the analysis aims to pinpoint both the triumphs and setbacks of the business. Through this exploration, the objective is to gain a comprehensive understanding of the business's performance, creating an avenue for strategic recommendations. The ultimate goal is to propose measures that will not only rectify shortcomings but also enhance the overall performance of the business.

### Data Sources

The data set used for this analysis was obtained from kaggle superSales. 

### Tools
- SQL Server - Data cleaning and Data Analysis
  
### Data cleaning/preparation
1. Data loading
   Build a databas, create a table and insert the data.
2. Data inspection
   Make sure NULL values and MISSING values are taken care of.
   Select columns with null Values in them. There are no null values in our database. When creating the table, we set NOT NULL for each field, hence null values were filtered out.

### Variable Engineering
   1. Add a new column named 'time_of_the_day' to show sales in the morning, afternoon and evening. This will help us to know the busiest hours.
   2. Add another column named 'day_of_the_week' to show the day on which a transcation was made. This will help us to know the bussy days.
   3. Add another column named 'month_name' to show the month in which a transaction happened. This will help us to show which month of the quarter the business recorded the most gross 
      income.
   4. Add another column named 'gross_margin' to show the gross income as a percentage of the total price.

### Exploratory Data Analysis
   1. Product Line
      - What is the distribution of revenue across the product lines?
      - What product line has the largest VAT?
      - What is the average unit price of each product line?
      - What is the average quantity for each product line?
      - Use average ratings to rate the product lines as Excellent, Good, Fair or Bad.
      - How does the revenue of the product lines vary by city?


  2. Sales
     - What is the overall revenue, costs and gross income of the bussiness?
     - Which branch has the most revenue?
     - In which city does the business incur the highest costs?
     - Determine the average quantity sold per order.
     - What is the bussiest time of the day?
     - Which branch sells the most?
     - Which month had the highest gross income?
    
  
  ### Code

  ```sql
-- --CREATE YOUR DATABASE-- 

CREATE TABLE sales (
    invoice_id VARCHAR(30) NOT NULL PRIMARY KEY,
	branch VARCHAR(30) NOT NULL,
    city VARCHAR(30) NOT NULL,
    product_line VARCHAR(100) NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,
    quantity INT NOT NULL,
    tax_5 FLOAT(6, 4) NOT NULL,
    total_price  DECIMAL (12, 4) NOT NULL,
    order_date DATETIME NOT NULL,
    order_time TIME NOT NULL,
    payment VARCHAR(15) NOT NULL,
    costs DECIMAL (10,2) NOT NULL,
    gross_income DECIMAL (10, 2), 
    rating FLOAT (2, 1) 
);


-- ------------------------------VARIABLE ENGINEERING--
-- ------------------------------ time of the day ----
SELECT Order_time,
(CASE WHEN (`Order_time`) BETWEEN "00:00:00" AND "12:00:00" THEN 'Morning'
      WHEN (`Order_time`) BETWEEN "12:01:00" AND "16:00:00" THEN "Afternoon"
      ELSE "Evening"
END) AS 'Time_of_the_Day'  
from sales;

ALTER TABLE SALES ADD COLUMN time_of_the_day VARCHAR(20);

UPDATE SALES
SET time_of_the_day = CASE WHEN `order_time` BETWEEN "00:00:00" AND "12:00:00" THEN "Morning"
                           WHEN `order_time` BETWEEN "12:01:00" AND "16:00:00" THEN "Afternoon"
                           ELSE "Evening"
END;

SELECT order_time, time_of_the_day FROM SALES;

-- --day of the week--

ALTER TABLE SALES ADD COLUMN day_of_the_week VARCHAR(10);

UPDATE SALES
SET day_of_the_week =  DAYNAME(order_date);

SELECT order_date, day_of_the_week FROM SALES;

-- --month name-----
ALTER TABLE SALES ADD COLUMN month_name VARCHAR(10);

UPDATE SALES
SET month_name = MONTHNAME(order_date); 

-- --gross margin--
ALTER TABLE SALES ADD COLUMN gross_margin DECIMAL (10, 2);

UPDATE SALES
SET gross_margin = FORMAT((gross_income)/(total_price) * 100, 2);

-- --------PRODUCT LINE ANALYSIS--
-- --What is the distribution of revenue across the product lines?

SELECT PRODUCT_LINE, SUM(TOTAL_PRICE) TOTAL_REVENUE FROM SALES
GROUP BY PRODUCT_LINE
ORDER BY SUM(TOTAL_PRICE) DESC;

-- --What product line has the largest VAT?--
SELECT product_line, AVG(tax_5) avg_tax FROM sales
GROUP BY product_line
ORDER BY AVG(tax_5) DESC; 

-- ---What is the average unit price of each product line?--

SELECT PRODUCT_LINE, AVG(UNIT_PRICE) AVG_UNIT_PRICE FROM SALES
GROUP BY PRODUCT_LINE
ORDER BY AVG(UNIT_PRICE) DESC; 

-- --What is the average quantity for each product line?--
SELECT PRODUCT_LINE, AVG(QUANTITY) AVG_QUANTITY FROM SALES
GROUP BY PRODUCT_LINE 
ORDER BY AVG(QUANTITY) DESC;

-- --Use average ratings to rate the product lines as excellent, good, fair or bad--

SELECT PRODUCT_LINE, AVG(RATING),
CASE WHEN AVG(RATING) >0 AND AVG(RATING) <5  THEN 'BAD'
     WHEN AVG(RATING) >=5 AND AVG(RATING) <7 THEN 'FAIR'
     WHEN AVG(RATING) >=7 AND AVG(RATING) <9 THEN 'GOOD'
     ELSE 'EXCELLENT'
 END AS SCALE_RATING
 FROM SALES
 GROUP BY PRODUCT_LINE
 ORDER BY AVG(RATING) DESC;
 
 -- ---How does the revenue of the product lines vary by city?--

 SELECT
 `Product_Line`, `City`, SUM(`Total_price`) AS TotalSales FROM `sales`
WHERE `City` IN ('plymouth', 'glasgow', 'bristol')
GROUP BY `Product_Line`, `City`
ORDER BY `Product_Line`, `City`;


-- -----SALES ANALYSIS--
-- -----What is the overall revenue, costs and gross income of the bussiness?--
SELECT SUM(total_price)Total_Revenue, SUM(costs) Total_Costs, SUM(gross_income)Total_gross_income FROM SALES; 

-- --Which branch has the most revenue?--
SELECT branch, SUM(total_price) total_revenue FROM SALES
GROUP BY branch
ORDER BY SUM(total_price) DESC;

-- --Average quantity sold per order--
SELECT SUM(quantity) / COUNT(order_time) AVG_Quantity_Per_Order from sales; 

-- --what is the bussiest time of the day?--
SELECT time_of_the_day, COUNT(time_of_the_day), day_of_the_week, COUNT(day_of_the_week) FROM SALES
GROUP BY time_of_the_day AND day_of_the_week
ORDER BY COUNT(time_of_the_day) DESC;

-- --which branch sells the most?--
SELECT branch, SUM(quantity) FROM SALES
GROUP BY branch
ORDER BY SUM(quantity) DESC;

-- --In wich city does the business incur the highest costs?
SELECT city, SUM(costs) FROM sales
GROUP BY city 
ORDER BY SUM(costs) DESC;

-- --which month had the highest gross income?
SELECT month_name, SUM(gross_income) FROM SALES
GROUP BY month_name
ORDER BY SUM(gross_income) DESC;

 -- --PAYMENT METHOD ANALYSIS---
-- --Which payment method is mostly used?--
SELECT DISTINCT(payment), COUNT(payment) COUNT FROM SALES
GROUP BY payment
ORDER BY COUNT(payment) DESC; 
```

### Findings
1. Food and Beverages was the best performing product line while Electronic Accessories was the worst in terms of revenue.
2. Home and Lifestyle product line generates the highest VAT.
3. Food and Beverages product line was generally rated as good while the rest were rated as fair.
4. All product lines performed best in Bristol, followed by Glasgow and lastly Plymouth city.
5. Branch C was the top revenue generator.
6. Costs were found to be highest in Glasgow City.
7. Evening hours are the peak hours. The business experiences many customers.
8. January contributed the most to the gross income of Q1.
9. Cash is the most preferred payment method.
    
