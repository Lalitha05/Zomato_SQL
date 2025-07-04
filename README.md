# 🍽️ Zomato SQL Analytics Project

## 📊 Project Overview

This is a self-initiated SQL analytics case study using a simulated Zomato-like food delivery dataset. The objective is to demonstrate end-to-end SQL capabilities: from relational schema design to advanced querying for generating business insights.

### Key Objectives

* Analyze customer ordering behavior
* Evaluate restaurant performance and revenue
* Track rider efficiency and delivery KPIs
* Uncover insights on order patterns and trends

---

## 📂 Dataset and Schema

The database, `zomato_db`, consists of five relational tables:

### 📌 Table Structures

* **Customers**

  * `customer_id` (Primary Key)
  * `customer_name`
  * `reg_date`

* **Restaurants**

  * `restaurant_id` (Primary Key)
  * `restaurant_name`
  * `city`
  * `opening_hours`

* **Orders**

  * `order_id` (Primary Key)
  * `customer_id` (Foreign Key → Customers)
  * `restaurant_id` (Foreign Key → Restaurants)
  * `order_item`
  * `order_date`
  * `order_time`
  * `order_status`
  * `order_amount`

* **Riders**

  * `rider_id` (Primary Key)
  * `rider_name`
  * `sign_up`

* **Deliveries**

  * `delivery_id` (Primary Key)
  * `order_id` (Foreign Key → Orders)
  * `delivery_status`
  * `delivery_time`
  * `rider_id` (Foreign Key → Riders)

---

## 🛠️ Database Setup

```sql
CREATE DATABASE zomato_db;
USE zomato_db;

CREATE TABLE customers (
  customer_id INT PRIMARY KEY,
  customer_name VARCHAR(30),
  reg_date DATE
);

CREATE TABLE restaurants (
  restaurant_id INT PRIMARY KEY,
  restaurant_name VARCHAR(55),
  city VARCHAR(25),
  opening_hours VARCHAR(55)
);

CREATE TABLE orders (
  order_id INT PRIMARY KEY,
  customer_id INT,
  restaurant_id INT,
  order_item VARCHAR(55),
  order_date DATE,
  order_time TIME,
  order_status VARCHAR(30),
  order_amount FLOAT,
  FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
  FOREIGN KEY (restaurant_id) REFERENCES restaurants(restaurant_id)
);

CREATE TABLE riders (
  rider_id INT PRIMARY KEY,
  rider_name VARCHAR(55),
  sign_up DATE
);

CREATE TABLE deliveries (
  delivery_id INT PRIMARY KEY,
  order_id INT,
  delivery_status VARCHAR(55),
  delivery_time TIME,
  rider_id INT,
  FOREIGN KEY (order_id) REFERENCES orders(order_id),
  FOREIGN KEY (rider_id) REFERENCES riders(rider_id)
);
```

---

## 📈 SQL Analytical Queries

Below are analytical SQL queries that help derive meaningful insights from the Zomato-like dataset:

### 1. Top 5 Most Frequent Dishes by Customer (Last 1 Year)

```sql
SELECT * FROM (
  SELECT
    c.customer_id,
    c.customer_name,
    o.order_item AS dishes,
    COUNT(*) AS total_orders,
    DENSE_RANK() OVER(ORDER BY COUNT(*) DESC) AS rnk
  FROM orders o
  LEFT JOIN customers c ON o.customer_id = c.customer_id
  WHERE o.order_date >= CURRENT_DATE() - INTERVAL 1 YEAR
    AND c.customer_name = 'Yogesh Kulkarni'
  GROUP BY c.customer_id, c.customer_name, o.order_item
) AS sub
WHERE rnk <= 5;
```

### 2. Popular Order Time Slots (2-Hour Intervals)

```sql
SELECT
  CASE
    WHEN HOUR(order_time) BETWEEN 0 AND 1 THEN '00:00 - 02:00'
    WHEN HOUR(order_time) BETWEEN 2 AND 3 THEN '02:00 - 04:00'
    WHEN HOUR(order_time) BETWEEN 4 AND 5 THEN '04:00 - 06:00'
    WHEN HOUR(order_time) BETWEEN 6 AND 7 THEN '06:00 - 08:00'
    WHEN HOUR(order_time) BETWEEN 8 AND 9 THEN '08:00 - 10:00'
    WHEN HOUR(order_time) BETWEEN 10 AND 11 THEN '10:00 - 12:00'
    WHEN HOUR(order_time) BETWEEN 12 AND 13 THEN '12:00 - 14:00'
    WHEN HOUR(order_time) BETWEEN 14 AND 15 THEN '14:00 - 16:00'
    WHEN HOUR(order_time) BETWEEN 16 AND 17 THEN '16:00 - 18:00'
    WHEN HOUR(order_time) BETWEEN 18 AND 19 THEN '18:00 - 20:00'
    WHEN HOUR(order_time) BETWEEN 20 AND 21 THEN '20:00 - 22:00'
    WHEN HOUR(order_time) BETWEEN 22 AND 23 THEN '22:00 - 00:00'
  END AS time_slot,
  COUNT(order_id) AS no_of_orders
FROM orders
GROUP BY time_slot
ORDER BY no_of_orders DESC;
```

### 3. Average Order Value for High-Frequency Customers (>130 Orders)

```sql
SELECT
  c.customer_id,
  c.customer_name,
  AVG(o.order_amount) AS aov
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.customer_id
GROUP BY c.customer_id, c.customer_name
HAVING COUNT(*) > 130;
```

### 4. High-Value Customers (Spend > ₹3500)

```sql
SELECT
  c.customer_id,
  c.customer_name,
  SUM(order_amount) AS total_spent
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.customer_id
GROUP BY c.customer_id, c.customer_name
HAVING SUM(order_amount) >= 3500;
```

### 5. Orders Not Delivered

```sql
SELECT
  r.restaurant_name,  
  r.city,
  COUNT(o.order_id) AS count_not_delivered
FROM orders o
LEFT JOIN restaurants r ON o.restaurant_id = r.restaurant_id
LEFT JOIN deliveries d ON o.order_id = d.order_id
WHERE d.delivery_id IS NULL
GROUP BY r.restaurant_id
ORDER BY count_not_delivered DESC;
```

### 6. Restaurant Revenue Ranking by City

```sql
SELECT
  r.restaurant_name,
  r.city,
  SUM(o.order_amount) AS total_rev,
  RANK() OVER (PARTITION BY r.city ORDER BY SUM(o.order_amount) DESC) AS rnk
FROM orders o
LEFT JOIN restaurants r ON o.restaurant_id = r.restaurant_id
WHERE o.order_date >= CURRENT_DATE() - INTERVAL 1 YEAR
GROUP BY r.restaurant_name, r.city
ORDER BY r.city, total_rev DESC;
```

### 7. Most Popular Dish (by City)

```sql
SELECT * FROM (
  SELECT 
    r.city, 
    o.order_item, 
    COUNT(o.order_item) AS total_order,
    RANK() OVER (PARTITION BY r.city ORDER BY COUNT(o.order_item) DESC) AS popular
  FROM orders o 
  LEFT JOIN restaurants r ON o.restaurant_id = r.restaurant_id
  GROUP BY r.city, o.order_item
) AS tb
WHERE popular = 1;

```
 
### 8. Customer Churn (2023 → 2024)

```sql

SELECT DISTINCT customer_id 
FROM orders
WHERE EXTRACT(YEAR FROM order_date) = 2023
  AND customer_id NOT IN (
    SELECT DISTINCT customer_id 
    FROM orders 
    WHERE EXTRACT(YEAR FROM order_date) = 2024
);
 ```

### 9. Cancellation Rate Comparison (2024 vs 2025)

```sql

WITH cancel_rate_24 AS (
  SELECT 
    o.restaurant_id, 
    COUNT(o.order_id) AS total_orders,
    COUNT(CASE WHEN d.delivery_id IS NULL THEN 1 END) AS not_delivered
  FROM orders o 
  LEFT JOIN deliveries d ON o.order_id = d.order_id
  WHERE EXTRACT(YEAR FROM o.order_date) = 2024
  GROUP BY o.restaurant_id
),
cancel_rate_25 AS (
  SELECT 
    o.restaurant_id, 
    COUNT(o.order_id) AS total_orders,
    COUNT(CASE WHEN d.delivery_id IS NULL THEN 1 END) AS not_delivered
  FROM orders o 
  LEFT JOIN deliveries d ON o.order_id = d.order_id
  WHERE EXTRACT(YEAR FROM o.order_date) = 2025
  GROUP BY o.restaurant_id
)
SELECT 
  cr24.restaurant_id, 
  (cr24.not_delivered / cr24.total_orders) * 100 AS cancel_ratio_24,
  (cr25.not_delivered / cr25.total_orders) * 100 AS cancel_ratio_25
FROM cancel_rate_24 cr24
LEFT JOIN cancel_rate_25 cr25 ON cr24.restaurant_id = cr25.restaurant_id;

```

### 10. Rider Average Delivery Time


```sql

SELECT 
  d.rider_id,
  SEC_TO_TIME(AVG(TIMESTAMPDIFF(SECOND, o.order_time, d.delivery_time))) AS ride_time
FROM deliveries d 
LEFT JOIN orders o ON d.order_id = o.order_id
WHERE delivery_status IN ('Delivered', 'Delayed')
GROUP BY d.rider_id;

```

### 11. Customer Segmentation (Gold vs Silver)


```sql

SELECT 
  customer_category,
  SUM(total_orders) AS no_of_orders,
  SUM(total_spent) AS total_revenue
FROM (
  SELECT 
    customer_id,
    COUNT(order_id) AS total_orders,
    SUM(order_amount) AS total_spent,
    CASE 
      WHEN SUM(order_amount) > (SELECT AVG(order_amount) FROM orders) THEN 'Gold'
      ELSE 'Silver'
    END AS customer_category
  FROM orders
  GROUP BY customer_id
) AS tb
GROUP BY customer_category;

```

### 12. Rider Monthly Earnings (8% Commission)


```sql

SELECT 
  d.rider_id,
  DATE_FORMAT(o.order_date, '%m-%y') AS month,
  SUM(order_amount) * 0.08 AS riders_earning
FROM orders o 
LEFT JOIN deliveries d ON o.order_id = d.order_id
WHERE delivery_status IN ('Delivered', 'Delayed')
GROUP BY d.rider_id, month
ORDER BY d.rider_id, month;

```

13. Rider Ratings Based on Delivery Time

```sql

SELECT 
  rider_id, 
  rider_time,
  CASE 
    WHEN rider_time < '00:15:00' THEN '5 STAR'
    WHEN rider_time BETWEEN '00:15:00' AND '00:20:00' THEN '4 STAR'
    ELSE '3 STAR'
  END AS rider_ratings
FROM (
  SELECT 
    d.rider_id,
    SEC_TO_TIME(TIMESTAMPDIFF(SECOND, o.order_time, d.delivery_time)) AS rider_time
  FROM orders o 
  LEFT JOIN deliveries d ON o.order_id = d.order_id
  WHERE delivery_status = 'Delivered'
) AS tb;
```

### 14. Order Frequency by Day (Peak Day per Restaurant)

```sql

SELECT 
  r.restaurant_name, 
  DAYNAME(o.order_date) AS day_of_week,
  COUNT(o.order_id) AS total_orders,
  RANK() OVER (PARTITION BY r.restaurant_name ORDER BY COUNT(o.order_id) DESC) AS ranking
FROM orders o
LEFT JOIN restaurants r ON o.restaurant_id = r.restaurant_id
GROUP BY r.restaurant_name, day_of_week
ORDER BY r.restaurant_name, ranking;

```

### 15. Customer Lifetime Value (CLV)

```sql

SELECT 
  customer_id,
  SUM(order_amount) AS CLV
FROM orders 
GROUP BY customer_id
ORDER BY CLV DESC;
```

### 16. Order Item Popularity (Seasonal Demand)

```sql

SELECT 
  order_item, 
  season, 
  COUNT(order_id) AS total_orders
FROM (
  SELECT 
    order_item, 
    order_id, 
    order_date,
    CASE 
      WHEN MONTH(order_date) BETWEEN 4 AND 6 THEN 'Spring'
      WHEN MONTH(order_date) BETWEEN 7 AND 8 THEN 'Summer'
      ELSE 'Winter'
    END AS season
  FROM orders
) AS t1
GROUP BY order_item, season
ORDER BY order_item, season DESC;

```
### 17. City-Wise Revenue Ranking (2024)

```sql

SELECT 
  city, 
  total_revenue,
  RANK() OVER (ORDER BY total_revenue DESC) AS city_rank
FROM (
  SELECT 
    r.city, 
    SUM(order_amount) AS total_revenue
  FROM orders o 
  LEFT JOIN restaurants r ON o.restaurant_id = r.restaurant_id
  WHERE YEAR(o.order_date) = 2024
  GROUP BY r.city
) AS ranked_data;

```

END
