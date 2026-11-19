---
title: SQL Exercises - 60 Problems with Solutions
tags: [sql, practice, exercises, interview]
---

# 🏋️ SQL Exercises — 60 Problems with Solutions

All problems use the [[SQL Practice Database]]. Every solution was tested on DuckDB and is written in standard SQL that also runs on PostgreSQL.

**Method:** read the question **aloud in English**, write your query, run it, and only then open the solution. Mark each one: ✅ solved alone · 🟡 needed a hint · ❌ looked at the solution (repeat it in 3 days).

Revenue rule used everywhere: `quantity * unit_price * (1 - discount)`, counting only orders with status `paid` or `shipped` unless stated otherwise.

---

## 🟢 Beginner

### 1. List all products cheaper than $50, ordered by price (cheapest first).

<details><summary>Solution</summary>

```sql
SELECT name, price FROM products WHERE price < 50 ORDER BY price;
```
*Rows returned on DataShop: 6*

</details>

### 2. Customers from Mexico or Colombia who signed up in 2025.

<details><summary>Solution</summary>

```sql
SELECT first_name, last_name, country, signup_date FROM customers
WHERE country IN ('MX','CO') AND signup_date >= '2025-01-01' AND signup_date < '2026-01-01';
```
*Rows returned on DataShop: 4*

</details>

### 3. Customers without an email address.

<details><summary>Solution</summary>

```sql
SELECT first_name, last_name FROM customers WHERE email IS NULL;
```
*Rows returned on DataShop: 1*

</details>

### 4. The 3 most expensive products.

<details><summary>Solution</summary>

```sql
SELECT name, price FROM products ORDER BY price DESC LIMIT 3;
```
*Rows returned on DataShop: 3*

</details>

### 5. Products whose name contains 'monitor' (any case).

<details><summary>Solution</summary>

```sql
SELECT name FROM products WHERE LOWER(name) LIKE '%monitor%';
```
*Rows returned on DataShop: 1*

</details>

### 6. Every distinct order status, alphabetically.

<details><summary>Solution</summary>

```sql
SELECT DISTINCT status FROM orders ORDER BY status;
```
*Rows returned on DataShop: 5*

</details>

### 7. Products that are out of stock.

<details><summary>Solution</summary>

```sql
SELECT name FROM products WHERE stock = 0;
```
*Rows returned on DataShop: 1*

</details>

### 8. Orders placed in the first quarter of 2025.

<details><summary>Solution</summary>

```sql
SELECT id, order_date FROM orders WHERE order_date >= '2025-01-01' AND order_date < '2025-04-01';
```
*Rows returned on DataShop: 4*

</details>

### 9. Show each product's price with 16% tax, rounded to 2 decimals.

<details><summary>Solution</summary>

```sql
SELECT name, price, ROUND(price * 1.16, 2) AS price_with_tax FROM products;
```
*Rows returned on DataShop: 15*

</details>

### 10. How many customers do we have per country? Most first.

<details><summary>Solution</summary>

```sql
SELECT country, COUNT(*) AS customers FROM customers GROUP BY country ORDER BY customers DESC, country;
```
*Rows returned on DataShop: 6*

</details>

### 11. Number of orders per status.

<details><summary>Solution</summary>

```sql
SELECT status, COUNT(*) AS orders FROM orders GROUP BY status ORDER BY orders DESC;
```
*Rows returned on DataShop: 5*

</details>

### 12. Number of products and average price per category id.

<details><summary>Solution</summary>

```sql
SELECT category_id, COUNT(*) AS products, ROUND(AVG(price), 2) AS avg_price
FROM products GROUP BY category_id ORDER BY category_id;
```
*Rows returned on DataShop: 5*

</details>

### 13. Customers (id) with more than 2 orders.

<details><summary>Solution</summary>

```sql
SELECT customer_id, COUNT(*) AS orders FROM orders GROUP BY customer_id HAVING COUNT(*) > 2;
```
*Rows returned on DataShop: 2*

</details>

### 14. Total units sold per product id, only products with at least 3 units.

<details><summary>Solution</summary>

```sql
SELECT product_id, SUM(quantity) AS units FROM order_items GROUP BY product_id HAVING SUM(quantity) >= 3 ORDER BY units DESC;
```
*Rows returned on DataShop: 11*

</details>

### 15. Total value of inventory (price × stock) per category id.

<details><summary>Solution</summary>

```sql
SELECT category_id, SUM(price * stock) AS inventory_value FROM products GROUP BY category_id ORDER BY inventory_value DESC;
```
*Rows returned on DataShop: 5*

</details>

### 16. Orders with the customer's full name.

<details><summary>Solution</summary>

```sql
SELECT o.id, o.order_date, c.first_name || ' ' || c.last_name AS customer
FROM orders o JOIN customers c ON c.id = o.customer_id ORDER BY o.id;
```
*Rows returned on DataShop: 20*

</details>

### 17. Products with their category name.

<details><summary>Solution</summary>

```sql
SELECT p.name, c.name AS category FROM products p JOIN categories c ON c.id = p.category_id ORDER BY c.name, p.name;
```
*Rows returned on DataShop: 15*

</details>

### 18. Orders with the employee who handled them — include orders without employee.

<details><summary>Solution</summary>

```sql
SELECT o.id, e.name AS employee FROM orders o LEFT JOIN employees e ON e.id = o.employee_id ORDER BY o.id;
```
*Rows returned on DataShop: 20*

</details>

### 19. Products that have never been sold.

<details><summary>Solution</summary>

```sql
SELECT p.name FROM products p LEFT JOIN order_items oi ON oi.product_id = p.id WHERE oi.product_id IS NULL;
```
*Rows returned on DataShop: 1*

</details>

### 20. Customers who have never placed an order.

<details><summary>Solution</summary>

```sql
SELECT c.first_name, c.last_name FROM customers c LEFT JOIN orders o ON o.customer_id = c.id WHERE o.id IS NULL;
```
*Rows returned on DataShop: 1*

</details>

---

## 🟡 Intermediate

### 21. Each employee and their manager's name.

<details><summary>Solution</summary>

```sql
SELECT e.name AS employee, m.name AS manager FROM employees e LEFT JOIN employees m ON m.id = e.manager_id ORDER BY e.id;
```
*Rows returned on DataShop: 6*

</details>

### 22. Number of orders handled by each employee, including employees with 0.

<details><summary>Solution</summary>

```sql
SELECT e.name, COUNT(o.id) AS orders FROM employees e LEFT JOIN orders o ON o.employee_id = e.id
GROUP BY e.name ORDER BY orders DESC, e.name;
```
*Rows returned on DataShop: 6*

</details>

### 23. Revenue per category (paid + shipped orders only).

<details><summary>Solution</summary>

```sql
SELECT cat.name AS category,
       ROUND(SUM(oi.quantity * oi.unit_price * (1 - oi.discount)), 2) AS revenue
FROM order_items oi
JOIN orders o       ON o.id = oi.order_id
JOIN products p     ON p.id = oi.product_id
JOIN categories cat ON cat.id = p.category_id
WHERE o.status IN ('paid','shipped')
GROUP BY cat.name ORDER BY revenue DESC;
```
*Rows returned on DataShop: 5*

</details>

### 24. Revenue per customer country (paid + shipped).

<details><summary>Solution</summary>

```sql
SELECT c.country, ROUND(SUM(oi.quantity * oi.unit_price * (1 - oi.discount)), 2) AS revenue
FROM orders o
JOIN customers c    ON c.id = o.customer_id
JOIN order_items oi ON oi.order_id = o.id
WHERE o.status IN ('paid','shipped')
GROUP BY c.country ORDER BY revenue DESC;
```
*Rows returned on DataShop: 6*

</details>

### 25. Total of each order (all statuses), biggest first.

<details><summary>Solution</summary>

```sql
SELECT order_id, ROUND(SUM(quantity * unit_price * (1 - discount)), 2) AS order_total
FROM order_items GROUP BY order_id ORDER BY order_total DESC;
```
*Rows returned on DataShop: 20*

</details>

### 26. Average order value (ticket) of paid + shipped orders.

<details><summary>Solution</summary>

```sql
WITH t AS (
    SELECT oi.order_id, SUM(oi.quantity * oi.unit_price * (1 - oi.discount)) AS total
    FROM order_items oi JOIN orders o ON o.id = oi.order_id
    WHERE o.status IN ('paid','shipped')
    GROUP BY oi.order_id
)
SELECT ROUND(AVG(total), 2) AS avg_ticket FROM t;
```
*Rows returned on DataShop: 1*

</details>

### 27. Products more expensive than the average product.

<details><summary>Solution</summary>

```sql
SELECT name, price FROM products WHERE price > (SELECT AVG(price) FROM products) ORDER BY price DESC;
```
*Rows returned on DataShop: 4*

</details>

### 28. Customers who bought at least one laptop (category 1). Use EXISTS.

<details><summary>Solution</summary>

```sql
SELECT c.first_name, c.last_name FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o JOIN order_items oi ON oi.order_id = o.id
    JOIN products p ON p.id = oi.product_id
    WHERE o.customer_id = c.id AND p.category_id = 1
);
```
*Rows returned on DataShop: 6*

</details>

### 29. The most expensive product of each category.

<details><summary>Solution</summary>

```sql
SELECT p.name, p.category_id, p.price FROM products p
WHERE p.price = (SELECT MAX(p2.price) FROM products p2 WHERE p2.category_id = p.category_id)
ORDER BY p.category_id;
```
*Rows returned on DataShop: 5*

</details>

### 30. Classify products into price tiers: premium (≥1000), mid (≥100), budget.

<details><summary>Solution</summary>

```sql
SELECT name, price,
       CASE WHEN price >= 1000 THEN 'premium' WHEN price >= 100 THEN 'mid' ELSE 'budget' END AS tier
FROM products ORDER BY price DESC;
```
*Rows returned on DataShop: 15*

</details>

### 31. Orders per year with columns for shipped, paid and cancelled (conditional aggregation).

<details><summary>Solution</summary>

```sql
SELECT EXTRACT(YEAR FROM order_date) AS year,
       SUM(CASE WHEN status = 'shipped' THEN 1 ELSE 0 END) AS shipped,
       SUM(CASE WHEN status = 'paid' THEN 1 ELSE 0 END) AS paid,
       SUM(CASE WHEN status = 'cancelled' THEN 1 ELSE 0 END) AS cancelled
FROM orders GROUP BY 1 ORDER BY 1;
```
*Rows returned on DataShop: 2*

</details>

### 32. Email domain of each customer and number of customers per domain (NULL emails as 'unknown').

<details><summary>Solution</summary>

```sql
SELECT COALESCE(SPLIT_PART(email, '@', 2), 'unknown') AS domain, COUNT(*) AS customers
FROM customers GROUP BY 1 ORDER BY customers DESC;
```
*Rows returned on DataShop: 2*

</details>

### 33. Number of orders per month (use DATE_TRUNC).

<details><summary>Solution</summary>

```sql
SELECT DATE_TRUNC('month', order_date) AS month, COUNT(*) AS orders FROM orders GROUP BY 1 ORDER BY 1;
```
*Rows returned on DataShop: 18*

</details>

### 34. Days between each customer's signup and their first order.

<details><summary>Solution</summary>

```sql
SELECT c.first_name, c.signup_date, MIN(o.order_date) AS first_order,
       MIN(o.order_date) - c.signup_date AS days_to_first_order
FROM customers c JOIN orders o ON o.customer_id = c.id
GROUP BY c.id, c.first_name, c.signup_date ORDER BY days_to_first_order;
```
*Rows returned on DataShop: 11*

</details>

### 35. Customers who ordered in 2025 AND in 2026 (INTERSECT).

<details><summary>Solution</summary>

```sql
SELECT customer_id FROM orders WHERE order_date < '2026-01-01'
INTERSECT
SELECT customer_id FROM orders WHERE order_date >= '2026-01-01'
ORDER BY customer_id;
```
*Rows returned on DataShop: 4*

</details>

### 36. Customers who ordered in 2025 but NOT in 2026 (EXCEPT) — at-risk customers.

<details><summary>Solution</summary>

```sql
SELECT customer_id FROM orders WHERE order_date < '2026-01-01'
EXCEPT
SELECT customer_id FROM orders WHERE order_date >= '2026-01-01'
ORDER BY customer_id;
```
*Rows returned on DataShop: 5*

</details>

### 37. Lifetime value per customer using CTEs (paid + shipped), top 5.

<details><summary>Solution</summary>

```sql
WITH order_totals AS (
    SELECT order_id, SUM(quantity * unit_price * (1 - discount)) AS total
    FROM order_items GROUP BY order_id
)
SELECT c.first_name, c.last_name, COUNT(*) AS orders, ROUND(SUM(ot.total), 2) AS ltv
FROM orders o
JOIN order_totals ot ON ot.order_id = o.id
JOIN customers c ON c.id = o.customer_id
WHERE o.status IN ('paid','shipped')
GROUP BY c.id, c.first_name, c.last_name
ORDER BY ltv DESC LIMIT 5;
```
*Rows returned on DataShop: 5*

</details>

### 38. Rank products by price inside their category (ROW_NUMBER, RANK, DENSE_RANK).

<details><summary>Solution</summary>

```sql
SELECT name, category_id, price,
       ROW_NUMBER() OVER (PARTITION BY category_id ORDER BY price DESC) AS rn,
       RANK()       OVER (PARTITION BY category_id ORDER BY price DESC) AS rnk,
       DENSE_RANK() OVER (PARTITION BY category_id ORDER BY price DESC) AS drnk
FROM products ORDER BY category_id, rn;
```
*Rows returned on DataShop: 15*

</details>

### 39. Top 2 most expensive products per category.

<details><summary>Solution</summary>

```sql
WITH r AS (
    SELECT name, category_id, price,
           ROW_NUMBER() OVER (PARTITION BY category_id ORDER BY price DESC) AS rn
    FROM products
)
SELECT name, category_id, price FROM r WHERE rn <= 2 ORDER BY category_id, price DESC;
```
*Rows returned on DataShop: 10*

</details>

### 40. Each customer's most recent order.

<details><summary>Solution</summary>

```sql
WITH r AS (
    SELECT id, customer_id, order_date,
           ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date DESC) AS rn
    FROM orders
)
SELECT customer_id, id AS last_order_id, order_date FROM r WHERE rn = 1 ORDER BY customer_id;
```
*Rows returned on DataShop: 11*

</details>

### 41. For each order: the customer's previous order date and days since it.

<details><summary>Solution</summary>

```sql
SELECT customer_id, id, order_date,
       LAG(order_date) OVER (PARTITION BY customer_id ORDER BY order_date) AS prev_date,
       order_date - LAG(order_date) OVER (PARTITION BY customer_id ORDER BY order_date) AS days_since_prev
FROM orders ORDER BY customer_id, order_date;
```
*Rows returned on DataShop: 20*

</details>

### 42. Each product's price as a percentage of the most expensive product in its category.

<details><summary>Solution</summary>

```sql
SELECT name, category_id, price,
       ROUND(100 * price / MAX(price) OVER (PARTITION BY category_id), 1) AS pct_of_max
FROM products ORDER BY category_id, pct_of_max DESC;
```
*Rows returned on DataShop: 15*

</details>

---

## 🔴 Advanced

### 43. Running total of orders by month.

<details><summary>Solution</summary>

```sql
WITH m AS (SELECT DATE_TRUNC('month', order_date) AS month, COUNT(*) AS orders FROM orders GROUP BY 1)
SELECT month, orders,
       SUM(orders) OVER (ORDER BY month ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total
FROM m ORDER BY month;
```
*Rows returned on DataShop: 18*

</details>

### 44. Monthly revenue and month-over-month growth % (paid + shipped).

<details><summary>Solution</summary>

```sql
WITH m AS (
    SELECT DATE_TRUNC('month', o.order_date) AS month,
           SUM(oi.quantity * oi.unit_price * (1 - oi.discount)) AS revenue
    FROM orders o JOIN order_items oi ON oi.order_id = o.id
    WHERE o.status IN ('paid','shipped') GROUP BY 1
)
SELECT month, ROUND(revenue, 2) AS revenue,
       ROUND(100.0 * (revenue - LAG(revenue) OVER (ORDER BY month))
             / NULLIF(LAG(revenue) OVER (ORDER BY month), 0), 1) AS mom_pct
FROM m ORDER BY month;
```
*Rows returned on DataShop: 16*

</details>

### 45. 3-month moving average of order counts.

<details><summary>Solution</summary>

```sql
WITH m AS (SELECT DATE_TRUNC('month', order_date) AS month, COUNT(*) AS orders FROM orders GROUP BY 1)
SELECT month, orders,
       ROUND(AVG(orders) OVER (ORDER BY month ROWS BETWEEN 2 PRECEDING AND CURRENT ROW), 2) AS ma_3
FROM m ORDER BY month;
```
*Rows returned on DataShop: 18*

</details>

### 46. Customers whose lifetime value is above the average customer's lifetime value.

<details><summary>Solution</summary>

```sql
WITH ltv AS (
    SELECT o.customer_id, SUM(oi.quantity * oi.unit_price * (1 - oi.discount)) AS ltv
    FROM orders o JOIN order_items oi ON oi.order_id = o.id
    WHERE o.status IN ('paid','shipped') GROUP BY o.customer_id
)
SELECT customer_id, ROUND(ltv, 2) AS ltv FROM ltv
WHERE ltv > (SELECT AVG(ltv) FROM ltv) ORDER BY ltv DESC;
```
*Rows returned on DataShop: 5*

</details>

### 47. Orders whose total is above that customer's own average order total.

<details><summary>Solution</summary>

```sql
WITH t AS (
    SELECT o.id, o.customer_id, SUM(oi.quantity * oi.unit_price * (1 - oi.discount)) AS total
    FROM orders o JOIN order_items oi ON oi.order_id = o.id GROUP BY o.id, o.customer_id
)
SELECT id, customer_id, ROUND(total, 2) AS total FROM (
    SELECT t.*, AVG(total) OVER (PARTITION BY customer_id) AS cust_avg FROM t
) x
WHERE total > cust_avg ORDER BY customer_id, id;
```
*Rows returned on DataShop: 6*

</details>

### 48. Employee hierarchy with level and path (recursive CTE).

<details><summary>Solution</summary>

```sql
WITH RECURSIVE org AS (
    SELECT id, name, manager_id, 1 AS level, CAST(name AS VARCHAR) AS path
    FROM employees WHERE manager_id IS NULL
    UNION ALL
    SELECT e.id, e.name, e.manager_id, org.level + 1, org.path || ' > ' || e.name
    FROM employees e JOIN org ON e.manager_id = org.id
)
SELECT level, name, path FROM org ORDER BY path;
```
*Rows returned on DataShop: 6*

</details>

### 49. Daily order counts for February 2026 including days with zero orders (date spine).

<details><summary>Solution</summary>

```sql
WITH days AS (
    SELECT CAST(d AS DATE) AS day
    FROM generate_series(DATE '2026-02-01', DATE '2026-02-28', INTERVAL '1 day') AS g(d)
)
SELECT days.day, COUNT(o.id) AS orders
FROM days LEFT JOIN orders o ON o.order_date = days.day
GROUP BY days.day ORDER BY days.day;
```
*Rows returned on DataShop: 28*

</details>

### 50. Each customer's longest gap (days) between consecutive orders.

<details><summary>Solution</summary>

```sql
WITH g AS (
    SELECT customer_id,
           order_date - LAG(order_date) OVER (PARTITION BY customer_id ORDER BY order_date) AS gap
    FROM orders
)
SELECT customer_id, MAX(gap) AS longest_gap_days FROM g
WHERE gap IS NOT NULL GROUP BY customer_id ORDER BY longest_gap_days DESC;
```
*Rows returned on DataShop: 5*

</details>

### 51. Pareto: cumulative % of revenue by product (which products make 80%?).

<details><summary>Solution</summary>

```sql
WITH rev AS (
    SELECT p.name, SUM(oi.quantity * oi.unit_price * (1 - oi.discount)) AS revenue
    FROM order_items oi JOIN products p ON p.id = oi.product_id
    JOIN orders o ON o.id = oi.order_id WHERE o.status IN ('paid','shipped')
    GROUP BY p.name
)
SELECT name, ROUND(revenue, 2) AS revenue,
       ROUND(100 * SUM(revenue) OVER (ORDER BY revenue DESC ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)
             / SUM(revenue) OVER (), 1) AS cumulative_pct
FROM rev ORDER BY revenue DESC;
```
*Rows returned on DataShop: 14*

</details>

### 52. Median and 90th-percentile product price per category.

<details><summary>Solution</summary>

```sql
SELECT category_id,
       PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY price) AS median_price,
       PERCENTILE_CONT(0.9) WITHIN GROUP (ORDER BY price) AS p90_price
FROM products GROUP BY category_id ORDER BY category_id;
```
*Rows returned on DataShop: 5*

</details>

### 53. Monthly cohort table: customers active per month since their first purchase (paid + shipped).

<details><summary>Solution</summary>

```sql
WITH valid AS (SELECT * FROM orders WHERE status IN ('paid','shipped')),
first_order AS (
    SELECT customer_id, DATE_TRUNC('month', MIN(order_date)) AS cohort FROM valid GROUP BY customer_id
),
act AS (SELECT DISTINCT customer_id, DATE_TRUNC('month', order_date) AS m FROM valid)
SELECT f.cohort,
       (EXTRACT(YEAR FROM a.m) - EXTRACT(YEAR FROM f.cohort)) * 12
     + (EXTRACT(MONTH FROM a.m) - EXTRACT(MONTH FROM f.cohort)) AS month_number,
       COUNT(DISTINCT a.customer_id) AS customers
FROM first_order f JOIN act a ON a.customer_id = f.customer_id
GROUP BY 1, 2 ORDER BY 1, 2;
```
*Rows returned on DataShop: 16*

</details>

### 54. Repeat-purchase rate: % of buying customers with 2+ valid orders.

<details><summary>Solution</summary>

```sql
WITH c AS (
    SELECT customer_id, COUNT(*) AS n FROM orders
    WHERE status IN ('paid','shipped') GROUP BY customer_id
)
SELECT ROUND(100.0 * COUNT(*) FILTER (WHERE n >= 2) / COUNT(*), 1) AS repeat_rate_pct FROM c;
```
*Rows returned on DataShop: 1*

</details>

### 55. Products frequently bought together (pairs appearing in the same order).

<details><summary>Solution</summary>

```sql
SELECT a.product_id AS product_a, b.product_id AS product_b, COUNT(*) AS times
FROM order_items a
JOIN order_items b ON a.order_id = b.order_id AND a.product_id < b.product_id
GROUP BY 1, 2 ORDER BY times DESC, 1, 2 LIMIT 10;
```
*Rows returned on DataShop: 9*

</details>

### 56. Sales rep performance: revenue, orders and rank among reps.

<details><summary>Solution</summary>

```sql
WITH r AS (
    SELECT e.name, COUNT(DISTINCT o.id) AS orders,
           SUM(oi.quantity * oi.unit_price * (1 - oi.discount)) AS revenue
    FROM employees e
    JOIN orders o ON o.employee_id = e.id AND o.status IN ('paid','shipped')
    JOIN order_items oi ON oi.order_id = o.id
    GROUP BY e.name
)
SELECT name, orders, ROUND(revenue, 2) AS revenue, RANK() OVER (ORDER BY revenue DESC) AS rnk
FROM r ORDER BY rnk;
```
*Rows returned on DataShop: 3*

</details>

### 57. Employees earning more than their manager.
*(Expected in DataShop: no rows. Try again after `UPDATE employees SET salary = 7000 WHERE id = 3;`)*

<details><summary>Solution</summary>

```sql
SELECT e.name, e.salary, m.name AS manager, m.salary AS manager_salary
FROM employees e JOIN employees m ON m.id = e.manager_id
WHERE e.salary > m.salary;
```
*Rows returned on DataShop: 0*

</details>

### 58. Second highest product price (without LIMIT/OFFSET).

<details><summary>Solution</summary>

```sql
SELECT MAX(price) AS second_highest FROM products WHERE price < (SELECT MAX(price) FROM products);
```
*Rows returned on DataShop: 1*

</details>

### 59. Detect duplicate customers by first + last name (data-quality check).
*(Expected: no rows — DataShop has no duplicates. Insert a duplicate customer to test it.)*

<details><summary>Solution</summary>

```sql
SELECT first_name, last_name, COUNT(*) AS n FROM customers
GROUP BY first_name, last_name HAVING COUNT(*) > 1;
```
*Rows returned on DataShop: 0*

</details>

### 60. Share of each country in total revenue and its rank (paid + shipped).

<details><summary>Solution</summary>

```sql
WITH r AS (
    SELECT c.country, SUM(oi.quantity * oi.unit_price * (1 - oi.discount)) AS revenue
    FROM orders o JOIN customers c ON c.id = o.customer_id
    JOIN order_items oi ON oi.order_id = o.id
    WHERE o.status IN ('paid','shipped') GROUP BY c.country
)
SELECT country, ROUND(revenue, 2) AS revenue,
       ROUND(100 * revenue / SUM(revenue) OVER (), 1) AS share_pct,
       DENSE_RANK() OVER (ORDER BY revenue DESC) AS rnk
FROM r ORDER BY rnk;
```
*Rows returned on DataShop: 6*

</details>

---

## 🗣️ Interview phrases

| English | Español |
|---|---|
| *"Let me first clarify the grain of the table."* | Primero aclaro la granularidad de la tabla. |
| *"I'll start with a CTE that aggregates orders, then join it to customers."* | Empezaré con una CTE que agregue pedidos y luego la uniré a clientes. |
| *"I'm using a LEFT JOIN so that customers without orders are kept."* | Uso LEFT JOIN para conservar a los clientes sin pedidos. |
| *"To handle ties, I'd use DENSE_RANK instead of ROW_NUMBER."* | Para manejar empates usaría DENSE_RANK en vez de ROW_NUMBER. |
| *"On a large table, I'd add an index on customer_id and order_date."* | En una tabla grande añadiría un índice en customer_id y order_date. |
| *"Let me double-check edge cases: NULLs, duplicates and empty groups."* | Reviso casos límite: NULLs, duplicados y grupos vacíos. |

🏠 [[00 - SQL Roadmap]]

#sql #practice