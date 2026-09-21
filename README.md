# assignment_1_InezaLisaGabrielle-28527
# Assignment 1 — Sunrise Supermarket Sales Analysis (SQL)

**Name:** INEZA Lisa Gabrielle
**Student ID:** _28527_
**DBMS used:** **Oracle Database 21c XE**, scripts written and run in Oracle SQL Developer (they also run in SQL\*Plus).
**Repository name:** `assignment_1_InezaLisaGabrielle_28527`

---

## 1. Summary of what I did

I designed and loaded a small sales database for a fictional shop, **Sunrise Supermarket**, and then wrote SQL to answer management's questions about customers, products and sales trends.

Concretely:

1. Created the four tables given in the assignment (`customers`, `products`, `orders`, `order_items`) exactly as specified, using Oracle data types (`NUMBER`, `VARCHAR2`, `DATE`).<img width="701" height="338" alt="image" src="https://github.com/user-attachments/assets/2fe92223-614c-49d3-bbb6-bab29a9e8318" />

2. Populated them with realistic sample data: **5 customers, 8 products across 4 categories, 15 orders and 29 order items**, spread across January–March 2025.
3. Wrote **3 JOIN queries** (INNER JOIN, JOIN, LEFT JOIN), **1 CTE query** (customers spending above average) and **4 window-function queries** (RANK, ROW\_NUMBER, running SUM, LAG).
4. Ran every query, captured the results, and wrote a business interpretation of what the numbers say about Sunrise Supermarket.

All amounts are in **Rwandan Francs (RWF)**.

---

## 2. How to run it

### Option A — Oracle SQL Developer (what I used)

1. Clone the repository:
   ```bash
   git clone https://github.com/<your-username>/assignment_1_gaby-<your_id>.git
   cd assignment_1_gaby-<your_id>
   ```
2. Open SQL Developer and connect to your Oracle database (I used the local `XEPDB1` service with a normal user account, not SYS).
3. Open and run the scripts **in this order**, using *Run Script* (F5) so that every statement executes:

   | Order | File | What it does |
   |---|---|---|
   | 1 | `sql/01_schema.sql` | Drops (if present) and creates the four tables |
   | 2 | `sql/02_insert_data.sql` | Inserts all sample rows and commits |
   | 3 | `sql/03_join_queries.sql` | The three JOIN queries |
   | 4 | `sql/04_cte_query.sql` | The CTE query |
   | 5 | `sql/05_window_queries.sql` | The four window-function queries |

   > The first time you run `01_schema.sql`, the four `DROP TABLE` lines will report "table or view does not exist". That is expected — the tables have not been created yet. Keep going; the `CREATE TABLE` statements below them will succeed.

4. In files 3–5, each query is separated by comments. You can run the whole file, or highlight one query and press **Ctrl + Enter** to run just that one.

### Option B — SQL\*Plus / command line

```bash
sqlplus username/password@localhost:1521/XEPDB1
SQL> @sql/01_schema.sql
SQL> @sql/02_insert_data.sql
SQL> @sql/03_join_queries.sql
SQL> @sql/04_cte_query.sql
SQL> @sql/05_window_queries.sql
```

Before running the query files in SQL\*Plus, these settings make the output readable:

```sql
SET LINESIZE 200
SET PAGESIZE 100
```

### Running on another DBMS

The queries are standard SQL apart from three Oracle-specific details. To run them on PostgreSQL, MySQL 8+ or SQL Server, change:

* `VARCHAR2` → `VARCHAR`, `NUMBER` → `INT` / `NUMERIC(10,2)`
* `DATE '2025-01-05'` literals → the equivalent date literal for that engine
* `order_date - previous_order_date` (Oracle returns a number of days) → `(order_date - previous_order_date)` in PostgreSQL returns an interval, so use `DATE_PART('day', ...)`; in MySQL use `DATEDIFF(order_date, previous_order_date)`; in SQL Server use `DATEDIFF(DAY, previous_order_date, order_date)`.

---

## 3. Business scenario

**Sunrise Supermarket** is a neighbourhood supermarket. It sells everyday products — groceries, dairy, household items and beverages — to registered customers. A customer places an **order** on a given date, and each order contains one or more **order items**, where each item is a product and the quantity bought of it. The unit price is stored on the product, so the value of an order line is `quantity × price`.

Management does not currently have any reporting. They want to answer three questions from the data they already collect:

1. **Who are our customers?** Where do they live, who buys the most, and is anyone registered but never buying?
2. **What do they buy?** Which products and which categories generate the revenue?
3. **How are sales trending?** Is revenue growing over time, and how often do customers come back?

The database has four tables:

```
customers (customer_id PK, customer_name, email, city)
products  (product_id PK, product_name, category, price)
orders    (order_id PK, customer_id FK → customers, order_date)
order_items (order_item_id PK, order_id FK → orders, product_id FK → products, quantity)
```

`customers` → `orders` is one-to-many (one customer places many orders).
`orders` → `order_items` is one-to-many (one order contains many lines).
`products` → `order_items` is one-to-many (one product appears on many lines).

### Data loaded

**Customers (5)** — Aline Uwase (Kigali), Eric Mugisha (Huye), Claudine Mukamana (Musanze), Patrick Habimana (Kigali), Denise Ingabire (Rubavu).
Denise is registered but has **never placed an order** — she is there on purpose, so the LEFT JOIN query has something real to show.

**Products (8 in 4 categories)**

| ID | Product | Category | Price (RWF) |
|---|---|---|---|
| 1 | Rice 5kg | Groceries | 8,500.00 |
| 2 | Cooking Oil 2L | Groceries | 6,200.00 |
| 3 | Sugar 1kg | Groceries | 1,500.00 |
| 4 | Fresh Milk 1L | Dairy | 1,200.00 |
| 5 | Yoghurt 500ml | Dairy | 2,000.00 |
| 6 | Bar Soap | Household | 900.00 |
| 7 | Detergent 1kg | Household | 3,500.00 |
| 8 | Mineral Water 5L | Beverages | 2,500.00 |

**Orders:** 15 orders from 5 January 2025 to 28 March 2025.
**Order items:** 29 lines across those 15 orders.

---

## 4. JOIN queries

### Query 1 — Every order with the customer's name, city and order date (INNER JOIN)

```sql
SELECT o.order_id,
       c.customer_name,
       c.city,
       TO_CHAR(o.order_date, 'YYYY-MM-DD') AS order_date
FROM   orders o
INNER JOIN customers c
       ON o.customer_id = c.customer_id
ORDER BY o.order_date, o.order_id;
```
<img width="740" height="434" alt="image" src="https://github.com/user-attachments/assets/b1de3d55-a1fa-4da6-8cc3-55f9f06b9721" />

**Explanation.** The `orders` table only stores `customer_id`, which is a number and means nothing to a manager reading a report. The INNER JOIN follows that foreign key into `customers` and pulls back the readable name and city. `INNER` means a row is returned only when the join condition matches on both sides — so an order without a valid customer, or a customer without orders, would not appear. Since every order must belong to a customer, all 15 orders come back.

**Result — 15 rows**

| order_id | customer_name | city | order_date |
|---|---|---|---|
| 1 | Aline Uwase | Kigali | 2025-01-05 |
| 2 | Eric Mugisha | Huye | 2025-01-08 |
| 3 | Aline Uwase | Kigali | 2025-01-15 |
| 4 | Claudine Mukamana | Musanze | 2025-01-19 |
| 5 | Patrick Habimana | Kigali | 2025-01-27 |
| 6 | Eric Mugisha | Huye | 2025-02-02 |
| 7 | Aline Uwase | Kigali | 2025-02-10 |
| 8 | Claudine Mukamana | Musanze | 2025-02-14 |
| 9 | Patrick Habimana | Kigali | 2025-02-20 |
| 10 | Eric Mugisha | Huye | 2025-02-25 |
| 11 | Aline Uwase | Kigali | 2025-03-03 |
| 12 | Claudine Mukamana | Musanze | 2025-03-09 |
| 13 | Patrick Habimana | Kigali | 2025-03-15 |
| 14 | Eric Mugisha | Huye | 2025-03-21 |
| 15 | Aline Uwase | Kigali | 2025-03-28 |

---

### Query 2 — Every order item with product name, category, price and quantity (JOIN)

```sql
SELECT oi.order_item_id,
       oi.order_id,
       p.product_name,
       p.category,
       p.price,
       oi.quantity,
       p.price * oi.quantity AS line_total
FROM   order_items oi
INNER JOIN products p
       ON oi.product_id = p.product_id
ORDER BY oi.order_item_id;
```
<img width="943" height="400" alt="image" src="https://github.com/user-attachments/assets/eb405381-3da2-4985-b3a7-1e2e8a447110" />

**Explanation.** Same idea one level down. `order_items` stores only `product_id` and `quantity`; the name, category and price live in `products`. Joining on `product_id` turns each line into something readable. I also added a calculated column `line_total = price × quantity`, because that multiplication is the basis of every revenue figure later in this report.

**Result — 29 rows**

| order_item_id | order_id | product_name | category | price | quantity | line_total |
|---|---|---|---|---|---|---|
| 1 | 1 | Rice 5kg | Groceries | 8,500.00 | 2 | 17,000.00 |
| 2 | 1 | Fresh Milk 1L | Dairy | 1,200.00 | 3 | 3,600.00 |
| 3 | 2 | Sugar 1kg | Groceries | 1,500.00 | 4 | 6,000.00 |
| 4 | 2 | Bar Soap | Household | 900.00 | 5 | 4,500.00 |
| 5 | 3 | Cooking Oil 2L | Groceries | 6,200.00 | 1 | 6,200.00 |
| 6 | 3 | Yoghurt 500ml | Dairy | 2,000.00 | 2 | 4,000.00 |
| 7 | 3 | Mineral Water 5L | Beverages | 2,500.00 | 3 | 7,500.00 |
| 8 | 4 | Rice 5kg | Groceries | 8,500.00 | 1 | 8,500.00 |
| 9 | 4 | Detergent 1kg | Household | 3,500.00 | 2 | 7,000.00 |
| 10 | 5 | Fresh Milk 1L | Dairy | 1,200.00 | 6 | 7,200.00 |
| 11 | 5 | Sugar 1kg | Groceries | 1,500.00 | 2 | 3,000.00 |
| 12 | 6 | Cooking Oil 2L | Groceries | 6,200.00 | 3 | 18,600.00 |
| 13 | 6 | Bar Soap | Household | 900.00 | 3 | 2,700.00 |
| 14 | 7 | Rice 5kg | Groceries | 8,500.00 | 1 | 8,500.00 |
| 15 | 7 | Mineral Water 5L | Beverages | 2,500.00 | 2 | 5,000.00 |
| 16 | 8 | Yoghurt 500ml | Dairy | 2,000.00 | 4 | 8,000.00 |
| 17 | 8 | Detergent 1kg | Household | 3,500.00 | 1 | 3,500.00 |
| 18 | 9 | Sugar 1kg | Groceries | 1,500.00 | 5 | 7,500.00 |
| 19 | 9 | Fresh Milk 1L | Dairy | 1,200.00 | 4 | 4,800.00 |
| 20 | 10 | Rice 5kg | Groceries | 8,500.00 | 2 | 17,000.00 |
| 21 | 10 | Cooking Oil 2L | Groceries | 6,200.00 | 1 | 6,200.00 |
| 22 | 11 | Detergent 1kg | Household | 3,500.00 | 2 | 7,000.00 |
| 23 | 11 | Mineral Water 5L | Beverages | 2,500.00 | 4 | 10,000.00 |
| 24 | 12 | Bar Soap | Household | 900.00 | 6 | 5,400.00 |
| 25 | 12 | Yoghurt 500ml | Dairy | 2,000.00 | 3 | 6,000.00 |
| 26 | 13 | Rice 5kg | Groceries | 8,500.00 | 2 | 17,000.00 |
| 27 | 14 | Fresh Milk 1L | Dairy | 1,200.00 | 5 | 6,000.00 |
| 28 | 15 | Cooking Oil 2L | Groceries | 6,200.00 | 2 | 12,400.00 |
| 29 | 15 | Sugar 1kg | Groceries | 1,500.00 | 3 | 4,500.00 |

Total of the `line_total` column: **224,600 RWF** — this is total revenue for the period, and every later revenue figure adds up to it.

---

### Query 3 — All customers and their orders, including customers with no orders (LEFT JOIN)

```sql
SELECT c.customer_id,
       c.customer_name,
       c.city,
       o.order_id,
       TO_CHAR(o.order_date, 'YYYY-MM-DD') AS order_date
FROM   customers c
LEFT JOIN orders o
       ON c.customer_id = o.customer_id
ORDER BY c.customer_id, o.order_date;
```
<img width="842" height="373" alt="image" src="https://github.com/user-attachments/assets/810ee049-21c5-483b-9034-b08f4a3d0bd8" />

**Explanation.** A LEFT JOIN keeps **every** row from the left table (`customers`) whether or not a match exists on the right. Where a customer has no orders, Oracle fills `order_id` and `order_date` with NULL instead of dropping the customer. That is exactly the difference from Query 1: an INNER JOIN here would return 15 rows and hide Denise completely. The LEFT JOIN returns 16 rows and makes the inactive customer visible — which is precisely the row management needs to see.

**Result — 16 rows** (blank cells are NULL)

| customer_id | customer_name | city | order_id | order_date |
|---|---|---|---|---|
| 1 | Aline Uwase | Kigali | 1 | 2025-01-05 |
| 1 | Aline Uwase | Kigali | 3 | 2025-01-15 |
| 1 | Aline Uwase | Kigali | 7 | 2025-02-10 |
| 1 | Aline Uwase | Kigali | 11 | 2025-03-03 |
| 1 | Aline Uwase | Kigali | 15 | 2025-03-28 |
| 2 | Eric Mugisha | Huye | 2 | 2025-01-08 |
| 2 | Eric Mugisha | Huye | 6 | 2025-02-02 |
| 2 | Eric Mugisha | Huye | 10 | 2025-02-25 |
| 2 | Eric Mugisha | Huye | 14 | 2025-03-21 |
| 3 | Claudine Mukamana | Musanze | 4 | 2025-01-19 |
| 3 | Claudine Mukamana | Musanze | 8 | 2025-02-14 |
| 3 | Claudine Mukamana | Musanze | 12 | 2025-03-09 |
| 4 | Patrick Habimana | Kigali | 5 | 2025-01-27 |
| 4 | Patrick Habimana | Kigali | 9 | 2025-02-20 |
| 4 | Patrick Habimana | Kigali | 13 | 2025-03-15 |
| 5 | Denise Ingabire | Rubavu | *(null)* | *(null)* |

---

## 5. CTE query — Customers who spend above average

```sql
WITH customer_totals AS (
    SELECT c.customer_id,
           c.customer_name,
           c.city,
           SUM(oi.quantity * p.price) AS total_spend
    FROM   customers c
    JOIN   orders      o  ON c.customer_id = o.customer_id
    JOIN   order_items oi ON o.order_id    = oi.order_id
    JOIN   products    p  ON oi.product_id = p.product_id
    GROUP BY c.customer_id, c.customer_name, c.city
),
average_spend AS (
    SELECT AVG(total_spend) AS avg_spend
    FROM   customer_totals
)
SELECT ct.customer_id,
       ct.customer_name,
       ct.city,
       ct.total_spend,
       ROUND(a.avg_spend, 2) AS avg_spend_all_customers
FROM   customer_totals ct
CROSS JOIN average_spend a
WHERE  ct.total_spend > a.avg_spend
ORDER BY ct.total_spend DESC;
```

**Explanation, step by step.**

* A **CTE** (Common Table Expression) is the `WITH name AS (...)` block. It gives a name to a query result so the rest of the statement can use it like a table. It exists only while the statement runs.
* `customer_totals` walks the full chain — customer → order → order item → product — because the price lives in `products` and the quantity lives in `order_items`. `SUM(oi.quantity * p.price)` multiplies each line first, then adds the lines up per customer. The `GROUP BY` is what makes it "per customer".
* `average_spend` then averages those four totals. This is the key reason a CTE is needed: you cannot compare a `SUM` to the `AVG` of that same `SUM` in one flat query, because the average only exists **after** the per-customer grouping is done. The CTE lets me finish step one, then use its output in step two.
* The final `SELECT` cross-joins the single average row onto every customer row and keeps only those above it. I display the average as well so the comparison is auditable.

Note that the average here is the average **across customers who bought something** (4 customers), not across all 5 registered customers — Denise has no rows to join, so the inner joins drop her.

**Result — 2 rows**

| customer_id | customer_name | city | total_spend | avg_spend_all_customers |
|---|---|---|---|---|
| 1 | Aline Uwase | Kigali | 85,700.00 | 56,150.00 |
| 2 | Eric Mugisha | Huye | 61,000.00 | 56,150.00 |

Total revenue 224,600 ÷ 4 buying customers = **56,150 RWF average**. Aline and Eric are above it; Patrick (39,500) and Claudine (38,400) are below.

---

## 6. Window-function queries

A **window function** computes a value across a set of rows related to the current row, but — unlike `GROUP BY` — it does **not** collapse the rows. Every input row still comes back, with an extra calculated column attached. `PARTITION BY` splits the rows into groups, `ORDER BY` inside `OVER (...)` decides the sequence used for the calculation.

### Window Query 1 — Rank customers by total spend, highest first

```sql
WITH customer_totals AS (
    SELECT c.customer_id,
           c.customer_name,
           SUM(oi.quantity * p.price) AS total_spend
    FROM   customers c
    JOIN   orders      o  ON c.customer_id = o.customer_id
    JOIN   order_items oi ON o.order_id    = oi.order_id
    JOIN   products    p  ON oi.product_id = p.product_id
    GROUP BY c.customer_id, c.customer_name
)
SELECT customer_id,
       customer_name,
       total_spend,
       RANK() OVER (ORDER BY total_spend DESC) AS spend_rank
FROM   customer_totals
ORDER BY spend_rank;
```

**Explanation.** The CTE produces one row per customer with their total. `RANK() OVER (ORDER BY total_spend DESC)` then numbers those rows from the biggest spender down. There is no `PARTITION BY`, so the window is the whole result set — one single ranking. I chose `RANK()` rather than `ROW_NUMBER()` because if two customers tied on spend, `RANK()` would give them the same rank (and then skip the next number), which is the honest representation of a tie; `ROW_NUMBER()` would arbitrarily put one ahead of the other.

**Result — 4 rows**

| customer_id | customer_name | total_spend | spend_rank |
|---|---|---|---|
| 1 | Aline Uwase | 85,700.00 | 1 |
| 2 | Eric Mugisha | 61,000.00 | 2 |
| 4 | Patrick Habimana | 39,500.00 | 3 |
| 3 | Claudine Mukamana | 38,400.00 | 4 |

---

### Window Query 2 — Number each customer's orders in the order placed

```sql
SELECT c.customer_name,
       o.order_id,
       TO_CHAR(o.order_date, 'YYYY-MM-DD') AS order_date,
       ROW_NUMBER() OVER (PARTITION BY o.customer_id
                          ORDER BY o.order_date, o.order_id) AS order_sequence
FROM   orders o
JOIN   customers c ON c.customer_id = o.customer_id
ORDER BY c.customer_name, order_sequence;
```

**Explanation.** `PARTITION BY o.customer_id` restarts the counter for every customer, and `ORDER BY o.order_date` decides the sequence. So each customer's earliest order is numbered 1, the next is 2, and so on. `order_id` is added as a tie-breaker in case a customer ever placed two orders on the same date, so the numbering is deterministic. This is the standard way to find a customer's first purchase (`order_sequence = 1`) or their most recent one.

**Result — 15 rows**

| customer_name | order_id | order_date | order_sequence |
|---|---|---|---|
| Aline Uwase | 1 | 2025-01-05 | 1 |
| Aline Uwase | 3 | 2025-01-15 | 2 |
| Aline Uwase | 7 | 2025-02-10 | 3 |
| Aline Uwase | 11 | 2025-03-03 | 4 |
| Aline Uwase | 15 | 2025-03-28 | 5 |
| Claudine Mukamana | 4 | 2025-01-19 | 1 |
| Claudine Mukamana | 8 | 2025-02-14 | 2 |
| Claudine Mukamana | 12 | 2025-03-09 | 3 |
| Eric Mugisha | 2 | 2025-01-08 | 1 |
| Eric Mugisha | 6 | 2025-02-02 | 2 |
| Eric Mugisha | 10 | 2025-02-25 | 3 |
| Eric Mugisha | 14 | 2025-03-21 | 4 |
| Patrick Habimana | 5 | 2025-01-27 | 1 |
| Patrick Habimana | 9 | 2025-02-20 | 2 |
| Patrick Habimana | 13 | 2025-03-15 | 3 |

---

### Window Query 3 — Running total of revenue over time

```sql
WITH daily_revenue AS (
    SELECT o.order_date,
           SUM(oi.quantity * p.price) AS day_revenue
    FROM   orders o
    JOIN   order_items oi ON o.order_id    = oi.order_id
    JOIN   products    p  ON oi.product_id = p.product_id
    GROUP BY o.order_date
)
SELECT TO_CHAR(order_date, 'YYYY-MM-DD') AS order_date,
       day_revenue,
       SUM(day_revenue) OVER (ORDER BY order_date
             ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total
FROM   daily_revenue
ORDER BY order_date;
```

**Explanation.** The CTE first collapses the order lines into one revenue figure per date. Then `SUM(...) OVER (ORDER BY order_date ...)` adds up every day's revenue from the first date through to the current row — that is what makes it *running* rather than a single grand total. The frame clause `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` states that explicitly: start at the very first row, stop at this one. I wrote the frame out instead of relying on the default because the default (`RANGE`) treats rows with equal dates as a single group; being explicit removes any ambiguity.

**Result — 15 rows**

| order_date | day_revenue | running_total |
|---|---|---|
| 2025-01-05 | 20,600.00 | 20,600.00 |
| 2025-01-08 | 10,500.00 | 31,100.00 |
| 2025-01-15 | 17,700.00 | 48,800.00 |
| 2025-01-19 | 15,500.00 | 64,300.00 |
| 2025-01-27 | 10,200.00 | 74,500.00 |
| 2025-02-02 | 21,300.00 | 95,800.00 |
| 2025-02-10 | 13,500.00 | 109,300.00 |
| 2025-02-14 | 11,500.00 | 120,800.00 |
| 2025-02-20 | 12,300.00 | 133,100.00 |
| 2025-02-25 | 23,200.00 | 156,300.00 |
| 2025-03-03 | 17,000.00 | 173,300.00 |
| 2025-03-09 | 11,400.00 | 184,700.00 |
| 2025-03-15 | 17,000.00 | 201,700.00 |
| 2025-03-21 | 6,000.00 | 207,700.00 |
| 2025-03-28 | 16,900.00 | 224,600.00 |

The final running total, 224,600 RWF, matches the sum of all order lines from Query 2 — a useful check that nothing was double-counted by the joins.

---

### Window Query 4 — Days between a customer's current and previous order

---sql
WITH order_gaps AS (
    SELECT c.customer_id,
           c.customer_name,
           o.order_id,
           o.order_date,
           LAG(o.order_date) OVER (PARTITION BY o.customer_id
                                   ORDER BY o.order_date) AS previous_order_date,
           COUNT(*) OVER (PARTITION BY o.customer_id)      AS orders_by_customer
    FROM   orders o
    JOIN   customers c ON c.customer_id = o.customer_id
)
SELECT customer_name,
       order_id,
       TO_CHAR(order_date, 'YYYY-MM-DD')          AS order_date,
       TO_CHAR(previous_order_date, 'YYYY-MM-DD') AS previous_order_date,
       order_date - previous_order_date           AS days_since_previous_order
FROM   order_gaps
WHERE  orders_by_customer > 1
ORDER BY customer_name, order_date;
```

**Explanation.** `LAG()` looks backwards one row inside the window and returns a value from it — here, the previous order's date for that same customer. `PARTITION BY o.customer_id` is essential: without it, `LAG` would grab the previous order of a *different* customer and the gaps would be meaningless. In Oracle, subtracting one `DATE` from another returns a number of days directly, so `order_date - previous_order_date` gives the gap.

`COUNT(*) OVER (PARTITION BY o.customer_id)` counts each customer's orders without collapsing the rows, and the outer `WHERE orders_by_customer > 1` uses it to keep only customers with more than one order — the condition the question asks for. This filter has to be applied outside the CTE, because a window function is not allowed in a `WHERE` clause in the same query block where it is computed.

Each customer's **first** order shows NULL, correctly — there is no earlier order to compare it to.

**Result — 15 rows**

| customer_name | order_id | order_date | previous_order_date | days_since_previous_order |
|---|---|---|---|---|
| Aline Uwase | 1 | 2025-01-05 | *(null)* | *(null)* |
| Aline Uwase | 3 | 2025-01-15 | 2025-01-05 | 10 |
| Aline Uwase | 7 | 2025-02-10 | 2025-01-15 | 26 |
| Aline Uwase | 11 | 2025-03-03 | 2025-02-10 | 21 |
| Aline Uwase | 15 | 2025-03-28 | 2025-03-03 | 25 |
| Claudine Mukamana | 4 | 2025-01-19 | *(null)* | *(null)* |
| Claudine Mukamana | 8 | 2025-02-14 | 2025-01-19 | 26 |
| Claudine Mukamana | 12 | 2025-03-09 | 2025-02-14 | 23 |
| Eric Mugisha | 2 | 2025-01-08 | *(null)* | *(null)* |
| Eric Mugisha | 6 | 2025-02-02 | 2025-01-08 | 25 |
| Eric Mugisha | 10 | 2025-02-25 | 2025-02-02 | 23 |
| Eric Mugisha | 14 | 2025-03-21 | 2025-02-25 | 24 |
| Patrick Habimana | 5 | 2025-01-27 | *(null)* | *(null)* |
| Patrick Habimana | 9 | 2025-02-20 | 2025-01-27 | 24 |
| Patrick Habimana | 13 | 2025-03-15 | 2025-02-20 | 23 |

---

## 7. Business interpretation

**Revenue.** Sunrise Supermarket took **224,600 RWF** across 15 orders in the first quarter of 2025, an average basket of about **14,973 RWF**.

**Who the customers are.** Only 4 of the 5 registered customers ever bought anything. Two of them, Aline Uwase (85,700) and Eric Mugisha (61,000), account for **65% of all revenue** between them. That concentration is a risk as much as a strength: if Aline alone stopped shopping here, the shop would lose over a third of its revenue. Management should treat the top two as accounts worth protecting — a loyalty discount or a courtesy call costs far less than replacing them.

**The inactive customer.** The LEFT JOIN surfaced Denise Ingabire, registered in Rubavu with zero orders. An INNER JOIN would have hidden her. She is the cheapest growth opportunity the shop has: she already gave her contact details, so a single re-engagement message costs almost nothing. It is also worth checking whether Rubavu is simply too far to serve — if so, the registration is a signal that a delivery option might unlock customers outside Kigali.

**By city.** Kigali generates 125,200 RWF (56%), Huye 61,000 and Musanze 38,400. The two Kigali customers behave quite differently from one another, so this is more a reflection of where the buyers happen to live than proof of a strong Kigali market — with only 5 customers, city is not yet a reliable segment.

**What they buy.** Groceries dominate at **132,400 RWF (59%)**, then Dairy (39,600), Household (30,100) and Beverages (22,500). The single biggest earner is **Rice 5kg at 68,000 RWF (30% of all revenue)**, followed by Cooking Oil 2L at 43,400. Notice the difference between volume and value: Fresh Milk 1L moves the most units of anything (18) but brings in only 21,600 RWF, and Bar Soap moves 14 units for 12,600 RWF — while rice sells just 8 units for 68,000. Stock decisions should follow value as well as movement — running out of rice costs far more than running out of soap. The practical action is to never let rice and cooking oil go out of stock, and to consider bundling low-value household items with the grocery staples that customers are already coming in for.

**How sales are trending.** The running total climbs steadily, but not evenly: January 74,500 → February 81,800 → March 68,300. February was the strongest month; March fell back about 17%, mostly because of one weak day (21 March, a single small 6,000 RWF order). With only three months of data this is a fluctuation, not a trend — but it is exactly the kind of dip that should be watched in April rather than explained away. The running total is also the right chart to show management: it makes the pace of revenue growth visible at a glance, with flat stretches revealing quiet weeks.

**How often customers return.** The `LAG` analysis shows a strikingly consistent rhythm: almost every customer returns after **21–26 days**, roughly monthly, which fits a household restocking on payday. The average gap is about 23 days. This gives the shop a concrete, testable rule: if a regular customer has not been seen in **30+ days**, they are overdue and worth a reminder. Aline is the exception — she also has a 10-day gap early on and has ordered 5 times versus 3–4 for the others, so she shops both monthly and for top-ups. She is the closest thing the shop has to a habitual customer, and understanding what brings her in mid-cycle would be worth more than any of these totals.

---

## 8. Challenges and how I resolved them

**1. Double-counting revenue in the joins.**
My first attempt at total spend joined customers → orders → order_items → products and also tried to count orders in the same query. Because one order has several item lines, the order rows get repeated by the join, and `COUNT(o.order_id)` came back inflated. I fixed it by keeping the aggregation to one level of grain per query — `SUM(quantity * price)` over the item lines, and counting orders separately with `COUNT(DISTINCT o.order_id)`. I then verified the whole thing by checking that the final running total (224,600) equals the sum of all 29 line totals in Query 2. That cross-check is what confirmed the joins were clean.

**2. Comparing a total to the average of totals.**
For the CTE question I first wrote `WHERE SUM(oi.quantity * p.price) > AVG(SUM(...))` and Oracle rejected it. The problem is ordering: the average cannot exist until the per-customer totals have been computed. The CTE solves it directly — `customer_totals` finishes the grouping, `average_spend` runs on top of that finished result, and the main query compares them. This was the clearest lesson of the assignment in *why* CTEs exist, rather than just how to type one.

**3. Window function inside WHERE.**
In Window Query 4 I tried `WHERE COUNT(*) OVER (PARTITION BY customer_id) > 1` and got `ORA-30483: window functions are not allowed here`. Window functions are evaluated after `WHERE`, so they cannot be used to filter in the same query block. I wrapped the calculation in a CTE and applied the filter in the outer query, where the column already exists.

**4. LAG crossing customer boundaries.**
My first `LAG(o.order_date) OVER (ORDER BY o.order_date)` had no `PARTITION BY`, so it returned the previous order in the *whole shop* — Aline's gap was being measured against Eric's order. The numbers looked plausible, which is what made it dangerous. Adding `PARTITION BY o.customer_id` restricted the window to one customer at a time. Checking that each customer's first order returns NULL was the test that proved it was right.

**5. Date arithmetic and display.**
Oracle's default date format (`05-JAN-25`) made the output hard to read and hard to sort mentally. I used `TO_CHAR(order_date, 'YYYY-MM-DD')` for display, but kept the arithmetic on the raw `DATE` column, since `order_date - previous_order_date` only returns a day count if both sides are real dates, not strings. Subtracting two `TO_CHAR` results would have failed.

**6. Rerunning the scripts.**
Re-running `02_insert_data.sql` produced `ORA-00001: unique constraint violated`, because the primary keys already existed. Rather than deleting rows by hand each time, I put the `DROP TABLE` statements at the top of `01_schema.sql` in reverse dependency order — `order_items` first, then `orders`, then `products` and `customers` — so the child tables go before the parents they reference. Now the whole set of scripts can be run from scratch any number of times.

---

## 9. Repository structure

```
assignment_1_InezaLisaGabrielle-28527/
├── README.md                   <- this file
└── sql/
    ├── 01_schema.sql           <- table definitions
    ├── 02_insert_data.sql      <- sample data
    ├── 03_join_queries.sql     <- JOIN queries 1-3
    ├── 04_cte_query.sql        <- CTE query
    └── 05_window_queries.sql   <- window-function queries 1-4
```

### Pushing to GitHub

```bash
cd assignment_1_gaby-<your_id>
git init
git add .
git commit -m "Assignment 1: Sunrise Supermarket SQL analysis"
git branch -M main
git remote add origin https://github.com/inezalisagabrielle-gif/assignment_1_InezaLisaGabrielle-28527.git
git push -u origin main
```
