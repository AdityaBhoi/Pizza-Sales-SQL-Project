# 🍕 Pizza Sales Analysis — SQL Project

<div align="center">

![MySQL](https://img.shields.io/badge/MySQL-8.0-orange?style=for-the-badge&logo=mysql&logoColor=white)
![SQL](https://img.shields.io/badge/SQL-Advanced-red?style=for-the-badge&logo=databricks&logoColor=white)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge)
![Queries](https://img.shields.io/badge/Queries-13-blue?style=for-the-badge)

**A deep-dive SQL analysis of Dominos Pizza sales data — uncovering revenue trends, bestsellers, peak hours, and business insights using MySQL.**

</div>

---

## 📌 Project Overview

This project analyzes a Dominos Pizza sales dataset using **MySQL Workbench**. Through 13 structured SQL queries — ranging from basic aggregations to advanced window functions — we extract meaningful business insights around orders, revenue, pizza preferences, and time-based trends.

---

## 🗂️ Database Schema

The database `dominos` consists of **4 tables**:

```
dominos
├── orders            (order_id, order_date, order_time)
├── order_details     (order_details_id, order_id, pizza_id, quantity)
├── pizzas            (pizza_id, pizza_type_id, size, price)
└── pizza_types       (pizza_type_id, name, category, ingredients)
```

**Relationships:**
- `orders` → `order_details` via `order_id`
- `order_details` → `pizzas` via `pizza_id`
- `pizzas` → `pizza_types` via `pizza_type_id`

---

## 📊 Key Metrics at a Glance

| Metric | Value |
|---|---|
| 🧾 Total Orders | **21,350** |
| 💰 Total Revenue | **$817,860.05** |
| 🍕 Avg Pizzas/Day | **138** |
| 🧩 Pizza Types | **32** |
| 📦 Categories | **4** (Classic, Supreme, Veggie, Chicken) |
| 📐 Sizes Available | **5** (S, M, L, XL, XXL) |

---

## 🔍 Queries & Analysis

### 🟢 Basic Queries

---

#### Q1 — Total Number of Orders Placed

```sql
SELECT COUNT(order_id) AS total_orders
FROM orders;
```

**Result:** `21,350 orders`

---

#### Q2 — Total Revenue Generated from Pizza Sales

```sql
SELECT
    ROUND(SUM(order_details.quantity * pizzas.price), 2) AS total_sales
FROM order_details
    JOIN pizzas ON pizzas.pizza_id = order_details.pizza_id;
```

**Result:** `$817,860.05`

---

#### Q3 — Highest Priced Pizza

```sql
SELECT pizza_types.name, pizzas.price
FROM pizza_types
    JOIN pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
ORDER BY pizzas.price DESC
LIMIT 1;
```

**Result:** `The Greek Pizza — $35.95`

---

#### Q4 — Most Common Pizza Size Ordered

```sql
SELECT pizzas.size,
    COUNT(order_details.order_details_id) AS order_count
FROM pizzas
    JOIN order_details ON pizzas.pizza_id = order_details.pizza_id
GROUP BY pizzas.size
ORDER BY order_count DESC;
```

| Size | Orders |
|------|--------|
| L    | 18,526 |
| M    | 15,385 |
| S    | 14,137 |
| XL   | 544    |
| XXL  | 28     |

> **Large (L)** is the most preferred size.

---

#### Q5 — Top 5 Most Ordered Pizza Types (by Quantity)

```sql
SELECT pizza_types.name, SUM(order_details.quantity) AS quantity
FROM pizza_types
    JOIN pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.name
ORDER BY quantity DESC
LIMIT 5;
```

| Pizza | Quantity |
|-------|----------|
| The Classic Deluxe Pizza | 2,453 |
| The Barbecue Chicken Pizza | 2,432 |
| The Hawaiian Pizza | 2,422 |
| The Pepperoni Pizza | 2,418 |
| The Thai Chicken Pizza | 2,371 |

---

### 🟡 Intermediate Queries

---

#### Q6 — Total Quantity Ordered by Pizza Category

```sql
SELECT pizza_types.category,
    SUM(order_details.quantity) AS quantity
FROM pizza_types
    JOIN pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.category
ORDER BY quantity DESC;
```

| Category | Quantity |
|----------|----------|
| Classic  | 14,888   |
| Supreme  | 11,987   |
| Veggie   | 11,649   |
| Chicken  | 11,050   |

---

#### Q7 — Distribution of Orders by Hour of the Day

```sql
SELECT HOUR(order_time) AS hour, COUNT(order_id) AS order_count
FROM orders
GROUP BY HOUR(order_time);
```

> **Peak hour: 1 PM (13:00)** with **2,455 orders** — ideal for staffing and prep planning.

---

#### Q8 — Category-wise Distribution of Pizzas

```sql
SELECT category, COUNT(name)
FROM pizza_types
GROUP BY category;
```

| Category | Count |
|----------|-------|
| Chicken  | 6     |
| Classic  | 8     |
| Supreme  | 9     |
| Veggie   | 9     |

---

#### Q9 — Average Number of Pizzas Ordered Per Day

```sql
SELECT ROUND(AVG(quantity), 0) AS avg_pizza_ordered_per_day
FROM (
    SELECT orders.order_date, SUM(order_details.quantity) AS quantity
    FROM orders
    JOIN order_details ON orders.order_id = order_details.order_id
    GROUP BY orders.order_date
) AS order_quantity;
```

**Result:** `138 pizzas/day` on average.

---

#### Q10 — Top 3 Most Ordered Pizza Types Based on Revenue

```sql
SELECT pizza_types.name, SUM(order_details.quantity * pizzas.price) AS revenue
FROM pizza_types
    JOIN pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.name
ORDER BY revenue DESC
LIMIT 3;
```

| Pizza | Revenue |
|-------|---------|
| The Thai Chicken Pizza | $43,434.25 |
| The Barbecue Chicken Pizza | $42,768.00 |
| The California Chicken Pizza | $41,409.50 |

---

### 🔴 Advanced Queries

---

#### Q11 — % Contribution of Each Pizza Category to Total Revenue

```sql
SELECT pizza_types.category,
    ROUND(SUM(order_details.quantity * pizzas.price) /
        (SELECT ROUND(SUM(order_details.quantity * pizzas.price), 2)
         FROM order_details
             JOIN pizzas ON pizzas.pizza_id = order_details.pizza_id)
    * 100, 2) AS revenue
FROM pizza_types
    JOIN pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.category
ORDER BY revenue DESC;
```

| Category | Revenue % |
|----------|-----------|
| Classic  | 26.91%    |
| Supreme  | 25.46%    |
| Chicken  | 23.96%    |
| Veggie   | 23.68%    |

> All categories are remarkably balanced — no single category dominates.

---

#### Q12 — Cumulative Revenue Generated Over Time

```sql
SELECT order_date,
    SUM(revenue) OVER (ORDER BY order_date) AS cum_revenue
FROM (
    SELECT orders.order_date,
        SUM(order_details.quantity * pizzas.price) AS revenue
    FROM order_details
        JOIN pizzas ON order_details.pizza_id = pizzas.pizza_id
        JOIN orders ON orders.order_id = order_details.order_id
    GROUP BY orders.order_date
) AS sales;
```

> Uses **`SUM() OVER`** window function to track running total of revenue across all dates.

---

#### Q13 — Top 3 Most Ordered Pizza Types per Category (by Revenue)

```sql
SELECT name, revenue
FROM (
    SELECT category, name, revenue,
        RANK() OVER(PARTITION BY category ORDER BY revenue DESC) AS rn
    FROM (
        SELECT pizza_types.category, pizza_types.name,
            SUM(order_details.quantity * pizzas.price) AS revenue
        FROM pizza_types
            JOIN pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
            JOIN order_details ON order_details.pizza_id = pizzas.pizza_id
        GROUP BY pizza_types.category, pizza_types.name
    ) AS a
) AS b
WHERE rn <= 3;
```

> Uses **`RANK() OVER (PARTITION BY category)`** to rank pizzas independently within each category.

---

## 💡 Key Insights

| # | Insight |
|---|---------|
| 🕐 | **Peak ordering hour is 1 PM (13:00)** — great for operational planning |
| 🍗 | **Chicken pizzas lead in revenue** despite having only 6 types |
| 📏 | **Large size dominates** with 18,526 orders — customers prefer big value |
| 🏆 | **Classic category tops quantity** with 14,888 pizzas sold |
| 💵 | **The Greek Pizza** is the most expensive at $35.95 |
| 📊 | **Revenue is nearly equal** across all 4 categories (~24–27% each) |
| 📈 | **Cumulative revenue grows steadily** — consistent business performance |

---

## 🛠️ SQL Concepts Used

| Concept | Used In |
|---------|---------|
| `JOIN` (INNER) | Q2, Q4, Q5, Q6, Q8, Q10, Q11, Q12, Q13 |
| `GROUP BY` + `ORDER BY` | Q4, Q5, Q6, Q7, Q8, Q10, Q11 |
| `SUM()`, `COUNT()`, `AVG()`, `ROUND()` | Q1–Q11 |
| Subqueries | Q9, Q11, Q12, Q13 |
| `RANK() OVER (PARTITION BY ...)` | Q13 |
| `SUM() OVER (ORDER BY ...)` | Q12 |
| `LIMIT` | Q3, Q5, Q10 |
| `HOUR()` date function | Q7 |

---

## 🚀 How to Run

1. **Clone or download** this repository
2. Open **MySQL Workbench** and connect to your local instance
3. Create the database:
   ```sql
   CREATE DATABASE dominos;
   USE dominos;
   ```
4. Import the dataset (CSV files or SQL dump) into the 4 tables
5. Run queries from the `.sql` files in order (Q1 through Q13)

---

## 📁 Project Structure

```
pizza-sales-sql/
├── README.md
├── queries/
│   ├── Q1_total_orders.sql
│   ├── Q2_total_revenue.sql
│   ├── Q3_highest_priced_pizza.sql
│   ├── Q4_most_common_size.sql
│   ├── Q5_top5_pizza_types.sql
│   ├── Q6_category_quantity.sql
│   ├── Q7_orders_by_hour.sql
│   ├── Q8_category_distribution.sql
│   ├── Q9_avg_pizzas_per_day.sql
│   ├── Q10_top3_by_revenue.sql
│   ├── Q11_revenue_contribution.sql
│   ├── Q12_cumulative_revenue.sql
│   └── Q13_top3_per_category.sql
├── dataset/
│   ├── orders.csv
│   ├── order_details.csv
│   ├── pizzas.csv
│   └── pizza_types.csv
└── Pizza_Sales_SQL_Project.pdf
```

---

## 🧰 Tools & Technologies

- **Database:** MySQL 8.0
- **IDE:** MySQL Workbench
- **Language:** SQL
- **Dataset:** Dominos Pizza Sales (2015)

---

## 👤 Author

**Aditya Bhoi**
- 📧 2303031240176@paruluniversity.ac.in
- 🔗 [LinkedIn](https://www.linkedin.com/in/adityabhoi/)
- 🐙 [GitHub](https://github.com/AdityaBhoi/)

---

<div align="center">

⭐ **If you found this project useful, give it a star!** ⭐

Made with ❤️ and 🍕

</div>
