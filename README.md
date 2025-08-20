# SQL project: Data Analysis for Zomato - A food Delivery company in india

## Overview

This project demonstrates my sql problem solving skills through the analysis of data for zomato, a popular food delivery company in india. The project involves setting up the database, importing database, erase null values, and solving variety of business problems using complex SQL quries.

## Project Structure

- Database setup: creating of the 'Zomato_db' database and the required tables.
- Data import: Inserting sample data into the tables.
- Data cleaning: Handling null values and ensure data integrity.
- Business problems: Solving 20 specific business problems using SQL quries.

## Database Setup
```sql
CREATE DATABASE zomato_db;
```
### 1. Dropping Existing Tables
```sql
Drop Table if exists customer;
Drop Table if exists restaurants;
Drop Table if exists orders;
Drop Table if exists riders;
Drop Table if exists deliveries;
```
### 2. Creating Tables
```sql
create table customer
(
	customer_id int primary key,
	customer_name varchar(25),
	reg_date date
	);

create table restaurants
(
	restaurant_id int primary key,
	restaurant_name varchar(55),
	city varchar(15),
	opening_hours varchar(55)
	);

create table orders
(
	order_id int primary key,
	customer_id int, --this col coming from cust table
	restaurant_id int, -- this col coming from rest table
	order_item varchar(55),
	order_date date,
	order_time time,
	order_status varchar(55),
	total_amount float
	);

--Adding fk constraint
Alter table orders
Add constraint fk_customers
Foreign Key (customer_id)
references customer(customer_id);

--Adding fk constraint
Alter table orders
Add constraint fk_restaurant_id
Foreign Key (restaurant_id)
references restaurants(restaurant_id);

create table riders
(
	rider_id int primary key,
	rider_name varchar(55),
	sign_up date
	);

Drop Table if exists deliveries;
create table deliveries
(
	delivery_id int primary key,
	order_id int, --this coming from orders table
	delivery_status varchar(35),
	delivery_time time,
	rider_id int, --this is coming from riders table
	constraint fk_orders foreign key (order_id) references orders(order_id),
	constraint fk_riders foreign key (rider_id) references riders(rider_id)
);
```
## Data Import
### Data cleaning and Handling Null Values
- Before performing analysis, i ensured that the data was clean and free from null values where necessary.
```sql
select count(*) from customer
where
	customer_name is null
	or
	reg_date is null

select count(*) from restaurants
where
	restaurant_name is null
	or
	city is null
	or
	opening_hours is null

select * from orders
where
	order_item is null
	or
	order_date is null
	or
	order_time is null
	or
	order_status is null
	or
	total_amount is null
```

## Problems Solved

### 1- Write a query to find the top 5 most frequently ordered dishes by customer called "Arjun Mehta" in the last one year.
```sql
SELECT
	CUSTOMER_NAME,
	DISHES,
	TOTAL_ORDERS
FROM ---- table name
	(
		SELECT
			C.CUSTOMER_ID,
			C.CUSTOMER_NAME,
			O.ORDER_ITEM AS DISHES,
			COUNT(*) AS TOTAL_ORDERS,
			DENSE_RANK() OVER (ORDER BY COUNT(*) DESC) AS RANK
		FROM
			ORDERS AS O
			JOIN CUSTOMER AS C ON C.CUSTOMER_ID = O.CUSTOMER_ID
		WHERE
			O.ORDER_DATE >= CURRENT_DATE - INTERVAL '1 Year'
			AND C.CUSTOMER_NAME = 'Arjun Mehta'
		GROUP BY 1,2,3
		ORDER BY 1,4 DESC
	) AS T1
WHERE RANK <= 5
```

### 2- Popular Time Slot
#### identify the time slots during which the most orders are placed. based on 2-hours intervals.
```sql
SELECT
	CASE
		WHEN EXTRACT(HOUR FROM ORDER_TIME) BETWEEN 0 AND 1  THEN '00:00 - 02:00'
		WHEN EXTRACT(HOUR FROM ORDER_TIME) BETWEEN 2 AND 3  THEN '02:00 - 04:00'
		WHEN EXTRACT(HOUR FROM ORDER_TIME) BETWEEN 4 AND 5  THEN '04:00 - 06:00'
		WHEN EXTRACT(HOUR FROM ORDER_TIME) BETWEEN 6 AND 7  THEN '06:00 - 08:00'
		WHEN EXTRACT(HOUR FROM ORDER_TIME) BETWEEN 8 AND 9  THEN '08:00 - 10:00'
		WHEN EXTRACT(HOUR FROM ORDER_TIME) BETWEEN 10 AND 11  THEN '10:00 - 12:00'
		WHEN EXTRACT(HOUR FROM ORDER_TIME) BETWEEN 12 AND 13  THEN '12:00 - 14:00'
		WHEN EXTRACT(HOUR FROM ORDER_TIME) BETWEEN 14 AND 15  THEN '14:00 - 16:00'
		WHEN EXTRACT(HOUR FROM ORDER_TIME) BETWEEN 16 AND 17  THEN '16:00 - 18:00'
		WHEN EXTRACT(HOUR FROM ORDER_TIME) BETWEEN 18 AND 19  THEN '18:00 - 20:00'
		WHEN EXTRACT(HOUR FROM ORDER_TIME) BETWEEN 20 AND 21  THEN '20:00 - 22:00'
		WHEN EXTRACT(HOUR FROM ORDER_TIME) BETWEEN 22 AND 23  THEN '22:00 - 00:00'
	END AS TIME_SLOT,
	COUNT(ORDER_ID) AS ORDER_COUNT
FROM 
	ORDERS
GROUP BY
	TIME_SLOT
ORDER BY
	ORDER_COUNT DESC;
```

#### Another way to slove this Q.
```sql
SELECT
	FLOOR(EXTRACT(HOUR FROM ORDER_TIME) / 2) * 2 AS START_TIME,
	FLOOR(EXTRACT(HOUR FROM ORDER_TIME) / 2) * 2 + 2 AS END_TIME,
FROM ORDERS
GROUP BY 1,2
ORDER BY 3 DESC
```

### 3. Order value Analysis
#### Q. Find the average order value per customer who has pqlaced more than 750 orders. Return customer_name, and AOV (average order value)
```sql
SELECT
	C.CUSTOMER_NAME,
	AVG(O.TOTAL_AMOUNT) AS AOV
FROM
	ORDERS AS O
	JOIN CUSTOMER AS C 
	ON C.CUSTOMER_ID = O.CUSTOMER_ID
GROUP BY 1
HAVING
	COUNT(ORDER_ID) > 750
```

### 4. High value customers
#### Q. List the customers who has spent more than 100k in total on food orders. return customer_name and Customer_id
```sql
SELECT
	C.CUSTOMER_NAME,
	O.CUSTOMER_ID,
	SUM(O.TOTAL_AMOUNT) AS TOTAL_SPENT
FROM
	ORDERS AS O
	JOIN CUSTOMER AS C 
	ON C.CUSTOMER_ID = O.CUSTOMER_ID
GROUP BY 1
HAVING
	SUM(O.TOTAL_AMOUNT) > 100000
```

### 5. Orders without delivery
#### Q. Write a query to find orders that were placed but not delivered. return each restaurant name, city and number of not delivered orders.
```sql 
SELECT
	R.RESTAURANT_NAME,
	R.CITY,
	COUNT(O.ORDER_ID) AS CNT_NOT_DELIVERY_ORDERS
FROM
	ORDERS AS O
	LEFT JOIN RESTAURANT AS R 
	ON R.RESTAURANT_ID = O.RESTAURANT_ID
	LEFT JOIN DELIVERY AS D 
	ON D.ORDER_ID = O.ORDER_ID
WHERE
	D.DELIVERY_ID IS NULL
GROUP BY 1
ORDER BY 2 DESC
```

### 6. Restaurant revenue ranking:
#### Q. Rank restaurant by their total revenue from the last year, including their name,total revenue, and rank within their city.
```sql
WITH
	RANKING_TABLE AS (
		SELECT
			R.CITY,
			R.RESTAURANT_NAME,
			SUM(TOTAL_AMOUNT) AS REVENUE,
			RANK() OVER (PARTITION BY R.CITY ORDER BY SUM(TOTAL_AMOUNT) DESC) AS RANK
		FROM
			ORDERS AS O
			JOIN RESTAURANT AS R 
			ON O.RESTAURANT_ID = R.RESTAURANT_ID
			WHERE O.ORDER_DATE >= CURRENT_DATE - INTERVAL '1 YEAR'
		GROUP BY 1,2
	)
SELECT
	*
FROM RANKING_TABLE
WHERE RANK = 1
```

### 7. Most popular dish by city:
#### Q. Identify the most popular dish in each city based on the number of orders.

```sql
SELECT * FROM
	(SELECT
			R.CITY,
			O.ORDER_ITEM AS DISH,
			COUNT(ORDER_ID) AS TOTAL_ORDERS,
			RANK() OVER (PARTITION BY R.CITY ORDER BY COUNT(ORDER_ID) DESC) AS RANK
		FROM
			ORDERS AS O
			JOIN RESTAURANT AS R 
			ON O.RESTAURANT_ID = R.RESTAURANT_ID
		GROUP BY
			1,2) AS T1
WHERE RANK = 1
```

### 8. Customer Churn
#### Q. Find the customer who have not placed an order in 2024 but did in 2023.

```sql
SELECT
  DISTINCT CUSTOMER_ID
FROM ORDERS
WHERE 
	(EXTRACT YEAR FROM ORDERS) = 2023
	AND 
	CUSTOMER_ID NOT IN (
		               SELECT DISTINCT CUSTOMER_ID FROM ORDERS
		               WHERE EXTRACT(YEAR FROM ORDERS) = 2024)
```

### 9. Cancellation rate comparision
#### Q. calculate and compare the order cancellation rate for each restaurant between the current year and previous year
```sql
WITH CANCEL_RATIO_23 AS (
	SELECT
		O.RESTAURANT_ID,
		COUNT(ORDER_ID) AS TOTAL_ORDERS,
		COUNT(CASE WHEN D.DELIVERY_ID IS NULL THEN 1 END) NOT_DELIVERED
	FROM ORDERS AS O
	LEFT JOIN DELIVERIES AS D 
	ON O.ORDER_ID = D.ORDER_ID
 	WHERE EXTRACT(YEAR FROM O.ORDER_DATE) = 2023
	GROUP BY O.RESTAURANT_ID
),
CANCEL_RATIO_24 AS (
	SELECT
		O.RESTAURANT_ID,
		COUNT(ORDER_ID) AS TOTAL_ORDERS,
		COUNT(CASE WHEN D.DELIVERY_ID IS NULL THEN 1 END) NOT_DELIVERED
	FROM ORDERS AS O
	LEFT JOIN DELIVERIES AS D 
	ON O.ORDER_ID = D.ORDER_ID
	WHERE EXTRACT(YEAR FROM O.ORDER_DATE) = 2024
	GROUP BY O.RESTAURANT_ID
),
LAST_YEAR_DATA AS (
	SELECT
		RESTAURANT_ID,
		TOTAL_ORDERS,
		NOT_DELIVERED,
		ROUND(NOT_DELIVERED::NUMERIC / TOTAL_ORDERS::NUMERIC * 100,2) AS CANCEL_RATIO
	FROM
		CANCEL_RATIO_23
),
CURRENT_YEAR_DATA AS (
	SELECT
		RESTAURANT_ID,
		TOTAL_ORDERS,
		NOT_DELIVERED,
		ROUND(NOT_DELIVERED::NUMERIC / TOTAL_ORDERS::NUMERIC * 100,2) AS CANCEL_RATIO
	FROM 
		CANCEL_RATIO_24
)
SELECT
	C.CURRENT_YEAR_DATA.RESTAURANT_ID AS REST_ID,
	C.CURRENT_YEAR_DATA.CANCEL_RATIO AS CRNT_CNL_RATIO,
	L.LAST_YEAR_DATA.CANCEL_RATIO AS LST_YR_RATIO
FROM CURRENT_YEAR_DATA AS C
JOIN 
LAST_YEAR_DATA AS L
ON C.CURRENT_YEAR_DATA.RESTAURANT_ID = L.LAST_YEAR_DATA.RESTAURANT_ID;
```

### 10. Rider Average delivery time
#### Q. Determine each rider's average delivery time.
```sql
SELECT
	O.ORDER_ID,
	O.ORDER_TIME,
	D.DELIVERY_TIME,
	D.RIDER_ID,
	O.ORDER_TIME - D.DELIVERY_TIME AS TIME_DIFFERENCE 
	EXTRACT(EPOCH FROM(O.ORDER_TIME - D.DELIVERY_TIME + 
	CASE WHEN D.DELIVERY_TIME < O.ORDER_TIME THEN INTERVAL '1 day'
	ELSE INTERVAL '0 day' END)) / 60 AS TIME_DIFFERENCE_INSEC
FROM ORDERS AS O
JOIN DELIVERIES AS D 
ON O.ORDER_ID = D.ORDER_ID
WHERE	D.DELIVERY_STATUS = 'Delivered';
```

### 11. Monthly Restaurant Growth Ratio:
#### Q. calculate each restaurant's growth ratio based on the total number of delivered orders since its joining
```sql
WITH GROWTH_RATIO
AS
(
SELECT
	O.RESTAURANT_ID,
	TO_CHAR(O.ORDER_DATE, 'mm-yy') AS MONTH,
	COUNT(O.ORDER_ID) AS crnt_month_ORDERS,
	LAG(COUNT(O.ORDER_ID),1) OVER(PARTITION BY O.RESTAURANT_ID ORDER BY TO_CHAR(O.ORDER_DATE, 'mm-yy')) AS PREV_MONTH_ORDERS
FROM ORDERS AS O
JOIN DELIVERIES AS D 
ON O.ORDER_ID = D.ORDER_ID
WHERE D.DELIVERY_STATUS = 'Delivered'
GROUP BY 1,2 
ORDER BY 1,2
)
SELECT 
	RESTAURANT_ID,
	MONTH,
	PREV_MONTH_ORDERS,
	crnt_month_ORDERS,
	(crnt_month_ORDERS::NUMERIC-PREV_MONTH_ORDERS::NUMERIC)/PREV_MONTH_ORDERS::NUMERIC * 100
	AS MONTLY_GROWTH_RATIO
FROM GROWTH_RATIO
```

### 12. Customer Segmentation:
#### Q. Customer Segmentation: Segment customers into 'Gold' or 'Silver' groups based on their total spending
#### - Compared to the average order value (AOV). If a customer's total spending exceeds the (AOV).
#### - Label them as 'Gold': otherwise label them as 'silver'. Write an SQL query to determine each segment's
#### - total number of orders and total revenue
```sql
select
	cust_category,
	sum(total_orders) as total_orders,
	sum(total_spent) as total_revenue
from
	(select
		customer_id,
		sum(total_amount) as total_spent,
		count(order_id) as total_orders,
		case
			when sum(total_amount) > (select avg(total_amount) from orders) then 'Gold'
			else 'Silver'
		end as cust_category
	from orders
	group by 1
	) as t1
group by 1
```

### 13. Rider Monthly Earning
#### Q. Calculate each riders total monthly earning, assuming they earning 8% of the order amount
```sql
SELECT
	D.RIDER_ID,
	TO_CHAR(O.ORDER_DATE, 'mm-yy') AS MONTH,
	SUM(TOTAL_AMOUNT) AS REVENUE,
	SUM(TOTAL_AMOUNT) * 0.08 AS RIDERS_EARNING
FROM ORDERS AS O
JOIN DELIVERIES AS D 
ON O.ORDER_ID = D.ORDER_ID
GROUP BY 1,2 
ORDER BY 1,2
```

### 14. Riders Rating analysis:
#### Q. Find the number of 5 stars, 4 stars and 3 stars ratings each rider has.
#### - riders receive this rating based on delivery time.
#### - if orders are delivered less than 15 minuts of order received time the rider get 5 star rating.
#### - if they deliver 15-20 minuts they get 4 star rating
#### - if they deliver after 20 minuts they get 3 star rating.
```sql
select 
	rider_id,
	stars,
	count(*) as total_stars
from
	(select 
		rider_id,
		delivery_took_time,
		case
			when delivery_took_time < 15 then '5 stars'
			when delivery_took_time between 15 and 20 then '4 stars'
			else '3 stars'
		end as stars
	from
	(
		select
			o.order_id,
			o.order_time,
			d.delivery_time,
			extract(epoch from (d.delivery_time - o.order_time +
			case when d.delivery_time < o.order_time then interval '1day'
			else interval '0 day' end
			))/60 as delivery_took_time,
			d.rider_id
		from orders as o
		join deliveries as d
		on o.order_id = d.order_id
		where delivery_status = 'Delivered'
	) as t1
) as t2
group by 1,2
order by 1,3 desc
```
 
### 15.Order frequency by day
#### Q. Analyze order frequency per day of the week and identify the peak day for each restaurant.
```sql
select * from
(
	select 
		r.restaurant_name,
		--o.order_date,
		to_char(o.order_date, 'day') as day,
		count(o.order_id) as total_orders,
		rank() over(partition by r.restaurant_name order by count(o.order_id) desc) as rank
	from orders as o
	join restaurants as r
	on o.restaurant_id = r.restaurant_id
	group by 1,2
	order by 1,3 desc
	) as t1
where rank = 1
```

### 16. Customer lifetime value (CLV)
#### Q. Calculate the total revenue generated by each customer over all their orders.
```sql
select
	o.customer_id,
	c.customer_name,
	sum(o.total_amount) as CLV
from orders as o
join customer as c
on o.customer_id = c.customer_id
group by 1,2
```

### 17. Monthly Sales Trends
#### Q. Identify sales trends by comparing each month's total sales to the previous month.
```sql
select
	extract(year from order_date) as year,
	extract(month from order_date) as month,
	sum(total_amount) as total_sales,
	lag(sum(total_amount),1) over (order by extract(year from order_date), extract(month from order_date)) as prv_month_sales
from orders
group by 1,2
```

### 18. Rider Efficiency:
#### Q. Evaluate rider efficiency by determining average delivery times and identifying those with the lowest and highest avg.
```sql
with New_table
as
(
	select 
		*,
		d.rider_id as riders_id,
		EXTRACT(EPOCH FROM(D.DELIVERY_TIME - O.ORDER_TIME + 
		CASE WHEN D.DELIVERY_TIME < O.ORDER_TIME THEN INTERVAL '1 day'
		ELSE INTERVAL '0 day' END)) / 60 AS TIME_deliver
	from orders as o
	join deliveries as d
	on o.order_id = d.order_id
	where d.delivery_status = 'Delivered'
),
riders_time
as
(
	select
		riders_id,
		avg(TIME_deliver) as avg_time
	from New_table
	group by 1
)
select
	min(avg_time),
	max(avg_time)
from riders_time
```

### 19. Order item populrity:
#### Q. Track the popularity of specific order items over time and identify seasonal demand spikes.
```sql
select
	order_item,
	seasons,
	count(order_id) as total_orders
from
(
	select 
		*,
		extract(month from order_date) as month,
		case
			when extract(month from order_date) between 4 and 6  then 'spring'
			when extract(month from order_date) > 6 and 
			extract(month from order_date) < 9 then 'summer'
			else 'winter'
		end as seasons
	from orders
) as t1

group by 1,2
order by 1,3
```

### 20. Rank each city based on the total revenue for last year 2023
```sql
select 
	r.city,
	sum(total_amount) as total_revenue,
	rank() over(order by sum(total_amount)desc) as city_rank
from orders as o
join restaurants as r
on o.restaurant_id = r.restaurant_id
group by 1
```

## Stay Updated and Join the Community
LinkedIn: www.linkedin.com/in/tusarkant-jena-6762b8208

Email-id: jtusarkant@gmail.com

**Thank you for your support, and I look forward to connecting with you!**
