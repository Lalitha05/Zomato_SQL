
# 🍽️ Zomato SQL Analytics Project

## 📊 Project Overview

This project is a self-driven analytics case study designed to explore and analyze restaurant, orders, riders data using SQL. Using a simulated Zomato-like dataset, I created an end-to-end analytics pipeline — from designing a relational database schema to writing analytical queries that provide deep insights into customer behavior, restaurant performance, rider efficiency, and more.

## 📁 Dataset and Schema

The database used is called `zomato_db`, and it consists of five interrelated tables:

### 🧱 Entity Relationship Diagram (ERD)

```text
+-------------+        +-----------+       +-----------+
|  customers  |        |  orders   |       | restaurants|
|-------------|        |-----------|       |------------|
| customer_id |◄───────┤ customer_id       | restaurant_id|
| customer_name        | order_id ├───────►| restaurant_id
| reg_date    |        | restaurant_id     | city
+-------------+        | order_item        | opening_hours
                       | order_date        +------------+
                       | order_time
                       | order_status
                       | order_amount
                       +-----------+
                            ▲
                            │
                            │
                     +-------------+
                     |  deliveries |
                     |-------------|
                     | delivery_id |
                     | order_id    |
                     | delivery_status
                     | delivery_time
                     | rider_id    |
                     +-------------+
                            ▲
                            │
                     +-------------+
                     |   riders     |
                     |-------------|
                     | rider_id    |
                     | rider_name  |
                     | sign_up     |
                     +-------------+
