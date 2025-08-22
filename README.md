# apple_data_analysis

# ![Apple Logo](https://github.com/YeshaswiniGowda16/apple_data_analysis/blob/main/store.jpg) Apple Retail Sales SQL Project - Analyzing Millions of Sales Rows

## Project Overview

This project is designed to showcase advanced SQL querying techniques through the analysis of over 1 million rows of Apple retail sales data. The dataset includes information about products, stores, sales transactions, and warranty claims across various Apple retail locations globally. By tackling a variety of questions, from basic to complex, demonstrating ability to write sophisticated SQL queries that extract valuable insights from large datasets.

## Entity Relationship Diagram (ERD)

![ERD](https://github.com/najirh/Apple-Retail-Sales-SQL-Project---Analyzing-Millions-of-Sales-Rows/blob/main/erd.png)

## Database Schema

The project uses five main tables:

1. **stores**: Contains information about Apple retail stores.
   - `store_id`: Unique identifier for each store.
   - `store_name`: Name of the store.
   - `city`: City where the store is located.
   - `country`: Country of the store.

2. **category**: Holds product category information.
   - `category_id`: Unique identifier for each product category.
   - `category_name`: Name of the category.

3. **products**: Details about Apple products.
   - `product_id`: Unique identifier for each product.
   - `product_name`: Name of the product.
   - `category_id`: References the category table.
   - `launch_date`: Date when the product was launched.
   - `price`: Price of the product.

4. **sales**: Stores sales transactions.
   - `sale_id`: Unique identifier for each sale.
   - `sale_date`: Date of the sale.
   - `store_id`: References the store table.
   - `product_id`: References the product table.
   - `quantity`: Number of units sold.

5. **warranty**: Contains information about warranty claims.
   - `claim_id`: Unique identifier for each warranty claim.
   - `claim_date`: Date the claim was made.
   - `sale_id`: References the sales table.
   - `repair_status`: Status of the warranty claim (e.g., Paid Repaired, Warranty Void).

## Objectives

The project is split into three tiers of questions to test SQL skills of increasing complexity:

### Easy to Medium 

1. Find the number of stores in each country.
```sql
SELECT 
	country,
	COUNT(store_id) as count_of_stores
FROM stores
GROUP BY country
ORDER BY 2 desc
```

2. Calculate the total number of units sold by each store.
```sql
SELECT 
	s.store_id,
	st.store_name,
	SUM(quantity) as total_units_sold
FROM sales as s
JOIN 
stores as st
ON st.store_id = s.store_id
GROUP BY 1, 2
ORDER BY 3 DESC;
```

3. Identify how many sales occurred in December 2023.
```sql
SELECT 
	COUNT(sale_id) as total_sales
FROM sales
WHERE TO_CHAR(sale_date, 'MM-YYYY') = '12-2023'
```
 
4. Determine how many stores have never had a warranty claim filed.
```sql
SELECT count(*) from stores
WHERE store_id NOT IN (
			SELECT 
			DISTINCT store_id
			FROM sales as s
			RIGHT JOIN warranty as w
			on s.sale_id = w.sale_id);
```

5. Calculate the percentage of warranty claims marked as "Warranty Void".
```sql
SELECT 
	ROUND
	(COUNT(claim_id)/ 
	(SELECT COUNT(*) FROM warranty)::numeric * 100, 2)
	as warranty_void_percent
FROM warranty
WHERE repair_status = 'Warranty Void';
```

6. Identify which store had the highest total units sold in the last year.
```sql
SELECT 
	store_id,
	SUM(quantity)
FROM sales
WHERE sale_date >= (CURRENT_DATE - INTERVAL '1 year')
GROUP BY 1
ORDER BY 2 DESC
LIMIT 1;
```

7. Count the number of unique products sold in the last year.
```sql
SELECT 
	COUNT(DISTINCT product_id)
FROM sales
WHERE sale_date >= CURRENT_DATE - INTERVAL '1 year';
```

8. Find the average price of products in each category.
```sql
SELECT 
	p.category_id,
	c.category_name,
	AVG(p.price) as average_price
FROM products as p
JOIN 
category as c
on p.category_id = c.category_id
GROUP BY 1,2
ORDER BY 3 DESC;
```

9. How many warranty claims were filed in 2020?
```sql
SELECT *,
	 EXTRACT(YEAR FROM claim_date) = 2020
FROM warranty
WHERE EXTRACT(YEAR FROM claim_date) = 2020;
```

10. For each store, identify the best-selling day based on highest quantity sold.
```sql
SELECT * 
FROM
(
	SELECT 
		store_id,
		TO_CHAR(sale_date, 'DAY') as day_name,
		SUM(quantity) as total_unit_sold,
		RANK () OVER(PARTITION BY store_id ORDER BY SUM(quantity) DESC) as rank
	FROM sales
	GROUP By 1, 2
) as t1
WHERE rank = 1;
```

### Medium to Hard 

11. Identify the least selling product in each country for each year based on total units sold.
```sql
SELECT *
FROM
(SELECT 
	st.country,
	p.product_name,
	SUM(s.quantity) as total_qty_sold,
	RANK() OVER (PARTITION BY st.country ORDER BY SUM(s.quantity)) as rank
FROM sales as s
JOIN
stores as st
ON s.store_id = st.store_id
JOIN
products as p
ON s.product_id = p.product_id
GROUP BY 1,2
) as t1
WHERE rank = 1;

--USING CTE:

WITH 
product_rank --CTE name
AS
(
	SELECT 
		st.country,
		p.product_name,
		SUM(s.quantity) as total_qty_sold,
		RANK() OVER (PARTITION BY st.country ORDER BY SUM(s.quantity)) as rank
	FROM sales as s
	JOIN
	stores as st
	ON s.store_id = st.store_id
	JOIN
	products as p
	ON s.product_id = p.product_id
	GROUP BY 1,2
) 
SELECT * FROM product_rank
WHERE rank = 1;
```

12. Calculate how many warranty claims were filed within 180 days of a product sale.
```sql
SELECT w.*,
	   s.sale_date
FROM warranty AS w
LEFT JOIN 
sales AS s
ON s.sale_id = w.sale_id
WHERE 
	w.claim_date - sale_date <= 180
```

13. Determine how many warranty claims were filed for products launched in the last two years.
```sql
SELECT 
	p.product_name,
	COUNT(w.claim_id) as no_claim,
	COUNT(s.sale_id)
FROM warranty AS w
RIGHT JOIN 
sales AS s
ON s.sale_id = w.sale_id
JOIN products as p
ON p.product_id = s.product_id
WHERE p.launch_date >= CURRENT_DATE - INTERVAL '2 years' 
GROUP BY 1
HAVING COUNT(s.sale_id) > 0
```

14. List the months in the last three years where sales exceeded 5,000 units in the USA.
```sql
SELECT 
	TO_CHAR(sale_date, 'MM_YYYY') as month,
	SUM(s.quantity) as total_unit_sold
FROM sales AS s
JOIN
stores AS st
ON s.store_id = st.store_id
WHERE 
	st.country = 'USA'
	AND 
	s.sale_date >= CURRENT_DATE - INTERVAL '3 year'
GROUP BY 1
HAVING SUM(s.quantity) > 5000
```

15. Identify the product category with the most warranty claims filed in the last two years.
```sql
SELECT 
	c.category_name,
	COUNT(w.claim_id) as total_claims
FROM warranty as w
LEFT JOIN
sales as s
ON w.sale_id = s.sale_id
JOIN products as p
ON p.product_id = s.product_id
JOIN
category as c
ON c.category_id = p.category_id
WHERE w.claim_date >= CURRENT_DATE - INTERVAL '2 year'
GROUP BY 1
ORDER BY 2 DESC;
```

### Complex 

16. Determine the percentage chance of receiving warranty claims after each purchase for each country.
```sql
SELECT 
	country,
	total_unit_sold,
	total_claims,
	ROUND(COALESCE(total_claims::numeric / total_unit_sold::numeric * 100, 0),2)
	as risk
FROM
(SELECT 
	st.country,
	SUM(s.quantity) as total_unit_sold,
	COUNT(w.claim_id) as total_claims
FROM sales AS s
JOIN stores AS st
ON s.store_id = st.store_id 
LEFT JOIN warranty as w
ON w.sale_id = s.sale_id
GROUP BY 1) t1
ORDER BY 4 DESC
```

17. Analyze the year-by-year growth ratio for each store.
```sql
WITH yearly_sales 
AS
(SELECT 
	s.store_id,
	st.store_name,
	EXTRACT(YEAR FROM sale_date) as year,
	SUM(s.quantity * p.price) as total_sale
FROM sales AS s
JOIN
products AS p
ON s.product_id = p.product_id
JOIN stores as st
ON st.store_id = s.store_id
GROUP BY 1, 2, 3
ORDER BY 2, 3
),

growth_ratio AS
(SELECT 
	store_name,
	year,
	LAG(total_sale,1) OVER(PARTITION BY store_name ORDER BY year) AS last_year_sales,
    total_sale as current_year_sale
FROM yearly_sales
)

SELECT 
	store_name,
	year,
	last_year_sales,
	current_year_sale,
	ROUND(
		(current_year_sale - last_year_sales) :: numeric / last_year_sales :: numeric *100
		,3) as growth_ratio
FROM growth_ratio
WHERE 
	last_year_sales IS NOT NULL
	AND 
	YEAR <> EXTRACT(YEAR FROM CURRENT_DATE )
```

18. Calculate the correlation between product price and warranty claims for products sold in the last five years, segmented by price range.
```sql
SELECT 
	CASE
		WHEN p.price < 500 THEN 'Less Expensive Product'
		WHEN p.price BETWEEN 500 AND 1000 THEN 'Mid Range Product'
		ELSE 'Expensive Product'
	END AS price_segment,
	COUNT(w.claim_id) as total_claim
FROM warranty as w
LEFT JOIN
sales as s
ON w.sale_id = s.sale_id
JOIN
products as p
ON p.product_id = s.product_id
WHERE claim_date >= CURRENT_DATE - INTERVAL '5 year'
GROUP BY 1
```

19. Identify the store with the highest percentage of "Paid Repaired" claims relative to total claims filed.
```sql
WITH paid_repair
AS
(SELECT
	s.store_id,
	COUNT(w.claim_id) AS paid_repair
FROM sales as s
JOIN warranty as w
ON w.sale_id = s.sale_id
WHERE w.repair_status = 'Paid Repaired'
GROUP BY 1),

total_repair
AS
(SELECT
	s.store_id,
	COUNT(w.claim_id) AS total_repair
FROM sales as s
JOIN warranty as w
ON w.sale_id = s.sale_id
GROUP BY 1)

SELECT 
	tr.store_id,
	st.store_name,
	pr.paid_repair,
	tr.total_repair,
	ROUND(pr.paid_repair::numeric / tr.total_repair::numeric * 100 ,2) as percentage_paid_repaired
FROM paid_repair as pr
JOIN
total_repair as tr
ON pr.store_id = tr.store_id
JOIN stores as st
ON tr.store_id = st.store_id
```

20. Write a query to calculate the monthly running total of sales for each store over the past four years and compare trends during this period.
```sql
WITH monthly_sales
AS
(SELECT
	store_id,
	EXTRACT(YEAR FROM sale_date) as year,
	EXTRACT(MONTH FROM sale_date) as month,
	SUM(p.price * s.quantity) as total_revenue
FROM sales as s
JOIN 
products as p
ON s.product_id = p.product_id
GROUP BY 1, 2, 3
ORDER BY 1, 2, 3
)
SELECT
	store_id,
	month,
	year,
	total_revenue,
	SUM(total_revenue) OVER (PARTITION BY store_id ORDER BY year, month) as running_total
FROM monthly_sales
```

21. Analyze product sales trends over time, segmented into key periods: from launch to 6 months, 6-12 months, 12-18 months, and beyond 18 months.
```sql
SELECT 
	p.product_name,
	CASE
		WHEN s.sale_date BETWEEN p.launch_date AND p.launch_date + INTERVAL '6 month' THEN '0-6 months'
		WHEN s.sale_date BETWEEN p.launch_date + INTERVAL '6 month' AND p.launch_date + INTERVAL '12 month' THEN '6-12 months'
		WHEN s.sale_date BETWEEN p.launch_date + INTERVAL '12 month' AND p.launch_date + INTERVAL '18 month' THEN '12-18 months'
		ELSE '18'
	END as product_lifecycle,
	SUM(s.quantity) as total_qty_sale
FROM sales as s
JOIN 
products as p
ON s.product_id = p.product_id
GROUP BY 1, 2
ORDER BY 1, 3 DESC
```

## Project Focus

This project primarily focuses on developing and showcasing the following SQL skills:

- **Complex Joins and Aggregations**: Demonstrating the ability to perform complex SQL joins and aggregate data meaningfully.
- **Window Functions**: Using advanced window functions for running totals, growth analysis, and time-based queries.
- **Data Segmentation**: Analyzing data across different time frames to gain insights into product performance.
- **Correlation Analysis**: Applying SQL functions to determine relationships between variables, such as product price and warranty claims.
- **Real-World Problem Solving**: Answering business-related questions that reflect real-world scenarios faced by data analysts.


## Dataset

- **Size**: 1 million+ rows of sales data.
- **Period Covered**: The data spans multiple years, allowing for long-term trend analysis.
- **Geographical Coverage**: Sales data from Apple stores across various countries.

## Conclusion

By completing this project, I have developed advanced SQL querying skills, improved my ability to handle large datasets, and gain practical experience in solving complex data analysis problems that are crucial for business decision-making. 

---
