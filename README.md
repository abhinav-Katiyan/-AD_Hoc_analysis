

<h1>AD_Hoc_Analysis </h1>
<h3>An Insight-Providing Financial and Sales Analysis Challenge for Management in the Consumer Goods Sector

Subject: Consumer Products | Role: Executive Management


Problem description

Imaginary firm Atliq Hardware is one of India's top manufacturers of computer hardware, with a strong international presence. The management has observed, nevertheless, that they do not receive enough information to enable them to act quickly and wisely based on facts. In order to better comprehend the data, the organisation decided to run a SQL challenge. Task: Review ad-hoc-requests.pdf; the business requires insights for ten ad hoc requests. To respond to these requests, a SQL query must be executed. Top-level management is the dashboard's intended audience, hence a presentation showcasing the insights is desired.
</h3>
<h3>
  MySQL concepts used :
  Joins, CTEs, Subqueries, Window functions, and various types of functions
</h3>
<h3>1. Markets of Operation - APAC Region</h3>
<p>This query retrieves a distinct list of all the markets where "Atliq Exclusive" operates its business within the APAC region. It helps in identifying the scope of the business geographically.</p>
<pre>
<code>
SELECT distinct(market)
FROM dim_customer
WHERE customer="Atliq Exclusive" AND region ='APAC';
</code>
</pre>

<h3>2. Percentage Increase in Unique Products (2021 vs. 2020)</h3>
<p>This query calculates the percentage change in unique products sold in 2021 compared to 2020. The output provides the number of unique products for both years and the calculated percentage change.</p>
<pre>
<code>
WITH 2020_ AS (
    SELECT COUNT(distinct(product_CODE)) AS UNIQUE_PRODUCT_2020
    FROM fact_sales_monthly
    WHERE fiscal_year = 2020),
2021_ AS (
    SELECT COUNT(distinct(product_CODE)) AS UNIQUE_PRODUCT_2021
    FROM fact_sales_monthly
    WHERE fiscal_year = 2021)

SELECT UNIQUE_PRODUCT_2020, UNIQUE_PRODUCT_2021,
    round(((UNIQUE_PRODUCT_2021 - UNIQUE_PRODUCT_2020) * 100 / UNIQUE_PRODUCT_2020), 1) AS percentage_Change
FROM 2020_, 2021_;
</code>
</pre>

<h3>3. Unique Product Count by Segment</h3>
<p>This query provides the count of unique products for each segment and orders the segments in descending order based on product counts. It helps in understanding which segments have the highest product variety.</p>
<pre>
<code>
SELECT segment, COUNT(distinct(product_code)) AS product_count
FROM dim_product
GROUP BY segment
ORDER BY product_count DESC;
</code>
</pre>

<h3>4. Segment with the Most Increase in Unique Products (2021 vs. 2020)</h3>
<p>This query compares the unique product counts per segment for 2020 and 2021 and calculates the difference, helping identify which segment experienced the highest growth in product variety.</p>
<pre>
<code>
WITH fy20 AS (
    SELECT segment, COUNT(distinct(p.product_code)) AS product_count_2020
    FROM dim_product p
    JOIN fact_sales_monthly f ON p.product_code = f.product_code
    WHERE fiscal_year = 2020
    GROUP BY p.segment),
fy21 AS (
    SELECT segment, COUNT(distinct(p.product_code)) AS product_count_2021
    FROM dim_product p
    JOIN fact_sales_monthly f ON p.product_code = f.product_code
    WHERE fiscal_year = 2021
    GROUP BY p.segment)

SELECT F.segment, F.product_count_2020, U.product_count_2021,
    U.product_count_2021 - F.product_count_2020 AS difference
FROM fy20 F
JOIN fy21 U ON F.segment = U.segment
ORDER BY difference DESC;
</code>
</pre>

<h3>5. Products with Highest and Lowest Manufacturing Costs</h3>
<p>This query identifies the products with the highest and lowest manufacturing costs. It provides details on product codes, product names, and the manufacturing cost per unit.</p>
<pre>
<code>
SELECT m.product_code, product, CONCAT(manufacturing_cost, '/unit') AS manufacturing_cost 
FROM fact_manufacturing_cost AS m
JOIN dim_product AS p ON m.product_code = p.product_code
WHERE m.manufacturing_cost = (
    SELECT MAX(manufacturing_cost) FROM fact_manufacturing_cost
) OR m.manufacturing_cost = (
    SELECT MIN(manufacturing_cost) FROM fact_manufacturing_cost
)
ORDER BY manufacturing_cost DESC;
</code>
</pre>

<h3>6. Top 5 Customers with Highest Average Pre-Invoice Discounts (India, 2021)</h3>
<p>This query generates a list of the top 5 customers who received the highest average pre-invoice discount percentages in the Indian market for the fiscal year 2021.</p>
<pre>
<code>
SELECT c.customer_code, c.customer, f.pre_invoice_discount_pct * 100 AS average_discount_percentage
FROM dim_customer c
JOIN fact_pre_invoice_deductions f ON c.customer_code = f.customer_code
WHERE fiscal_year = 2021 AND c.market = 'India'
ORDER BY pre_invoice_discount_pct DESC
LIMIT 5;
</code>
</pre>

<h3>7. Gross Sales Amount for "Atliq Exclusive" by Month</h3>
<p>This query provides a complete report of the gross sales amount for "Atliq Exclusive" for each month. It is used to analyze the high and low-performing months to support strategic decision-making.</p>
<pre>
<code>
WITH sales AS (
    SELECT date, fm.customer_code, fp.fiscal_year, gross_price * sold_quantity AS gross_sales
    FROM fact_gross_price fp
    JOIN fact_sales_monthly fm ON fm.product_code = fp.product_code AND fm.fiscal_year = fp.fiscal_year),
customer AS (
    SELECT date, c.customer_code, gross_sales 
    FROM sales s
    JOIN dim_customer c ON s.customer_code = c.customer_code
    WHERE customer = 'Atliq Exclusive')

SELECT MONTH(date) AS Month, YEAR(date) AS Year, ROUND(SUM(gross_sales) / 1000000, 2) AS Gross_sales_Amount_mln 
FROM customer
GROUP BY Month, Year
ORDER BY Year, Month;
</code>
</pre>

<h3>8. Quarter with Maximum Total Sold Quantity (2020)</h3>
<p>This query identifies the quarter in 2020 with the highest total sold quantity, grouped and sorted by total sales.</p>
<pre>
<code>
WITH quarter AS (
    SELECT sold_quantity,
        CASE
            WHEN MONTH(date) BETWEEN 9 AND 11 THEN 'Q1'
            WHEN MONTH(date) IN (12, 1, 2) THEN 'Q2'
            WHEN MONTH(date) BETWEEN 3 AND 5 THEN 'Q3'
            WHEN MONTH(date) BETWEEN 6 AND 8 THEN 'Q4'
        END AS Quarter
    FROM fact_sales_monthly
    WHERE fiscal_year = 2020)

SELECT Quarter, SUM(sold_quantity) AS total_sold_quantity 
FROM quarter
GROUP BY Quarter
ORDER BY total_sold_quantity DESC;
</code>
</pre>
