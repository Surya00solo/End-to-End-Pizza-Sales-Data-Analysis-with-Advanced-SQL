# 📊 End-to-End Pizza Sales Data Analysis with Advanced SQL

## 🚀 Project Overview

This project is an end-to-end SQL portfolio project designed to showcase comprehensive data analysis skills using a real-world Pizza Sales dataset. The primary goal is to demonstrate proficiency in SQL querying, ranging from basic to advanced techniques, to extract meaningful business insights from raw data. The project leverages **MySQL Workbench** for database management and querying, and includes steps for data ingestion, transformation, analysis, and visualization/presentation. The dataset is in CSV format and comprises four key files/tables: `orders`, `order_details`, `pizzas`, and `pizza_types`.

-----

## 🔑 Key Findings & Business Questions Answered

This analysis provides key insights into pizza sales performance and customer preferences by answering the following business questions:

  * **Total Orders & Revenue:** Calculated the total number of orders placed and the total revenue generated.
  * **Top Sellers:** Identified the pizza with the highest price and the top three most ordered pizza types based on revenue.
  * **Customer Preferences:** Determined the most common pizza size and the top five most ordered pizza types by quantity.
  * **Sales Distribution:** Analyzed the distribution of orders by hour of the day and calculated the average number of pizzas ordered daily.
  * **Category Analysis:** Found the total quantity and revenue contribution of each pizza category.
  * **Advanced Metrics:** Analyzed the cumulative revenue over time and determined the top three most ordered pizza types by revenue for each pizza category.

-----

## 🛠️ Skillsets Utilized

This project demonstrates a robust set of skills essential for a data analyst:

  * **Advanced SQL Querying:**
      * **Data Retrieval & Aggregation:** `SELECT`, `WHERE`, `COUNT`, `SUM`, `AVG`, `GROUP BY`.
      * **Complex Joins:** `INNER JOIN` across multiple tables (`orders`, `pizza_types`, `pizzas`, `order_details`).
      * **Subqueries and Common Table Expressions (CTEs):** Used for multi-step calculations and readability.
      * **Window Functions:** `ROW_NUMBER()` and `SUM() OVER()` for ranking and cumulative analysis.
      * **Date and Time Functions:** `EXTRACT(HOUR)` for temporal analysis.
      * **Formatting:** `ROUND()` for numerical results.
  * **Database Management (MySQL Workbench):**
      * Database and table creation with appropriate data types and constraints.
      * Efficient CSV data import.
  * **Project Presentation & Version Control:**
      * Creating a project presentation (e.g., using Canva).
      * Utilizing **GitHub** for version control, hosting SQL scripts, and the final presentation.

-----

## 💻 SQL Code

Here are the complete SQL scripts used for this project, including table creation and all the queries for the analysis.

### **Part 1: Database and Table Creation**

```sql
-- Create the new database
CREATE DATABASE IF NOT EXISTS pizzahut;
USE pizzahut;

-- Create the pizzas table
CREATE TABLE pizzas (
    pizza_id VARCHAR(50) PRIMARY KEY,
    pizza_type_id VARCHAR(50),
    size VARCHAR(10),
    price DECIMAL(10, 2)
);

-- Create the pizza_types table
CREATE TABLE pizza_types (
    pizza_type_id VARCHAR(50) PRIMARY KEY,
    name VARCHAR(100),
    category VARCHAR(50),
    ingredients TEXT
);

-- Create the orders table
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    date DATE,
    time TIME
);

-- Create the order_details table
CREATE TABLE order_details (
    order_details_id INT PRIMARY KEY,
    order_id INT,
    pizza_id VARCHAR(50),
    quantity INT
);
```

### **Part 2: Data Analysis Queries**

```sql
-- 1. Calculate the total number of orders placed.
SELECT COUNT(DISTINCT order_id) AS total_orders FROM orders;

-- 2. Calculate the total revenue generated from pizza sales.
SELECT SUM(pizzas.price * order_details.quantity) AS total_revenue
FROM order_details
JOIN pizzas ON order_details.pizza_id = pizzas.pizza_id;

-- 3. Identify the pizza with the highest price.
SELECT pizza_types.name, pizzas.price
FROM pizzas
JOIN pizza_types ON pizzas.pizza_type_id = pizza_types.pizza_type_id
ORDER BY pizzas.price DESC
LIMIT 1;

-- 4. Determine the most commonly ordered pizza size.
SELECT pizzas.size, SUM(order_details.quantity) AS total_quantity
FROM order_details
JOIN pizzas ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizzas.size
ORDER BY total_quantity DESC
LIMIT 1;

-- 5. List the top five most ordered pizza types along with their total quantities.
SELECT pizza_types.name, SUM(order_details.quantity) AS total_quantity
FROM order_details
JOIN pizzas ON order_details.pizza_id = pizzas.pizza_id
JOIN pizza_types ON pizzas.pizza_type_id = pizza_types.pizza_type_id
GROUP BY pizza_types.name
ORDER BY total_quantity DESC
LIMIT 5;

-- 6. Find the total quantity of each pizza category ordered.
SELECT pizza_types.category, SUM(order_details.quantity) AS total_quantity
FROM order_details
JOIN pizzas ON order_details.pizza_id = pizzas.pizza_id
JOIN pizza_types ON pizzas.pizza_type_id = pizza_types.pizza_type_id
GROUP BY pizza_types.category
ORDER BY total_quantity DESC;

-- 7. Determine the distribution of orders by hour of the day.
SELECT EXTRACT(HOUR FROM time) AS order_hour, COUNT(DISTINCT order_id) AS total_orders
FROM orders
GROUP BY order_hour
ORDER BY order_hour;

-- 8. Analyze the category-wise distribution of pizzas.
SELECT category, COUNT(pizza_type_id) AS number_of_pizzas
FROM pizza_types
GROUP BY category;

-- 9. Calculate the average number of pizzas ordered per day.
WITH DailyPizzas AS (
    SELECT orders.date, SUM(order_details.quantity) AS total_pizzas
    FROM orders
    JOIN order_details ON orders.order_id = order_details.order_id
    GROUP BY orders.date
)
SELECT AVG(total_pizzas) AS avg_pizzas_per_day
FROM DailyPizzas;

-- 10. Identify the top three most ordered pizza types based on revenue.
SELECT pizza_types.name, SUM(pizzas.price * order_details.quantity) AS total_revenue
FROM order_details
JOIN pizzas ON order_details.pizza_id = pizzas.pizza_id
JOIN pizza_types ON pizzas.pizza_type_id = pizza_types.pizza_type_id
GROUP BY pizza_types.name
ORDER BY total_revenue DESC
LIMIT 3;

-- 11. Calculate the percentage contribution of each pizza category to total revenue.
SELECT pizza_types.category,
    (SUM(pizzas.price * order_details.quantity) / (SELECT SUM(pizzas.price * order_details.quantity) FROM order_details JOIN pizzas ON order_details.pizza_id = pizzas.pizza_id)) * 100 AS percentage_contribution
FROM order_details
JOIN pizzas ON order_details.pizza_id = pizzas.pizza_id
JOIN pizza_types ON pizzas.pizza_type_id = pizza_types.pizza_type_id
GROUP BY pizza_types.category
ORDER BY percentage_contribution DESC;

-- 12. Analyze the cumulative revenue generated over time.
SELECT orders.date, SUM(pizzas.price * order_details.quantity) AS daily_revenue,
    SUM(SUM(pizzas.price * order_details.quantity)) OVER (ORDER BY orders.date) AS cumulative_revenue
FROM orders
JOIN order_details ON orders.order_id = order_details.order_id
JOIN pizzas ON order_details.pizza_id = pizzas.pizza_id
GROUP BY orders.date
ORDER BY orders.date;

-- 13. Determine the top three most ordered pizza types based on revenue for each pizza category.
WITH RankedPizzas AS (
    SELECT pizza_types.category, pizza_types.name,
        SUM(pizzas.price * order_details.quantity) AS total_revenue,
        ROW_NUMBER() OVER (PARTITION BY pizza_types.category ORDER BY SUM(pizzas.price * order_details.quantity) DESC) AS ranking
    FROM order_details
    JOIN pizzas ON order_details.pizza_id = pizzas.pizza_id
    JOIN pizza_types ON pizzas.pizza_type_id = pizza_types.pizza_type_id
    GROUP BY pizza_types.category, pizza_types.name
)
SELECT category, name, total_revenue
FROM RankedPizzas
WHERE ranking <= 3
ORDER BY category, ranking;
```

-----
