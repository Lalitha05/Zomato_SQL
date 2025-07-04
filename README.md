# 🍽️ Zomato SQL Analytics Project

## 📊 Project Overview

This is a self-initiated SQL analytics case study based on a simulated Zomato-like food delivery dataset. The goal is to demonstrate end-to-end SQL capabilities including schema design, data exploration, and advanced querying for deriving business insights.

Key focus areas include:
- Customer behavior analysis
- Restaurant performance
- Rider efficiency
- Revenue and order patterns

---

## 🗂️ Dataset and Schema

The project uses a relational database named `zomato_db`, which consists of five interconnected tables:

### 📌 Tables and Columns

- **Customers**
  - `customer_id` (Primary Key)
  - `customer_name`
  - `reg_date`

- **Restaurants**
  - `restaurant_id` (Primary Key)
  - `restaurant_name`
  - `city`
  - `opening_hours`

- **Orders**
  - `order_id` (Primary Key)
  - `customer_id` (Foreign Key → Customers)
  - `restaurant_id` (Foreign Key → Restaurants)
  - `order_item`
  - `order_date`
  - `order_time`
  - `order_status`
  - `order_amount`

- **Riders**
  - `rider_id` (Primary Key)
  - `rider_name`
  - `sign_up` (date)

- **Deliveries**
  - `delivery_id` (Primary Key)
  - `order_id` (Foreign Key → Orders)
  - `delivery_status`
  - `delivery_time`
  - `rider_id` (Foreign Key → Riders)

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


## 📊 Project Overview
