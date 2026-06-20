# 🗄️ SQL — Joins & Set Operations

> A guide to combining data across tables using JOINs and combining query results using Set Operations.

---

## 📚 Table of Contents

- [What is a JOIN?](#what-is-a-join)
- [Types of JOINs](#types-of-joins)
- [Join on Multiple Columns](#join-on-multiple-columns)
- [Join on Multiple Tables](#join-on-multiple-tables)
- [Set Operations](#set-operations)
- [Industry Best Practices](#industry-best-practices)

---

## What is a JOIN?

A `JOIN` combines rows from two or more tables based on a related column between them. Used whenever data is spread across normalized tables (e.g., `customers` and `orders`) and needs to be queried together.

```sql
SELECT columns
FROM table1
JOIN table2 ON table1.common_col = table2.common_col;
```

---

## Types of JOINs

| JOIN Type | Returns |
|---|---|
| `INNER JOIN` | Only matching rows in both tables |
| `LEFT JOIN` | All rows from left table + matched rows from right (NULL if no match) |
| `RIGHT JOIN` | All rows from right table + matched rows from left (NULL if no match) |
| `FULL JOIN` | All rows from both tables (matched + unmatched) |
| `SELF JOIN` | Table joined with itself |
| `CROSS JOIN` | Cartesian product — every row of table1 with every row of table2 |

### INNER JOIN
Returns only rows with matching values in **both** tables. Most commonly used join.
```sql
SELECT orders.order_id, customers.name
FROM orders
INNER JOIN customers ON orders.cid = customers.cid;
```
> `JOIN` by itself defaults to `INNER JOIN` in MySQL.

### LEFT JOIN (LEFT OUTER JOIN)
Returns **all** rows from the left table, with matched rows from the right. Unmatched right-side columns return `NULL`.
```sql
SELECT customers.name, orders.order_id
FROM customers
LEFT JOIN orders ON customers.cid = orders.cid;
```
**Common use case:** finding rows with no match (e.g., customers with no orders) using `WHERE right.col IS NULL`.

### RIGHT JOIN (RIGHT OUTER JOIN)
Returns **all** rows from the right table, with matched rows from the left.
```sql
SELECT customers.name, orders.order_id
FROM customers
RIGHT JOIN orders ON customers.cid = orders.cid;
```
> Logically the same as `LEFT JOIN` with tables swapped. Most developers prefer `LEFT JOIN` for readability.

### FULL JOIN (FULL OUTER JOIN)
Returns all rows from both tables — matched where possible, `NULL` where not.
> ⚠️ **Not supported natively in MySQL.** Simulated using `UNION` of `LEFT JOIN` and `RIGHT JOIN`:
```sql
SELECT * FROM customers
LEFT JOIN orders ON customers.cid = orders.cid
UNION
SELECT * FROM customers
RIGHT JOIN orders ON customers.cid = orders.cid;
```

### SELF JOIN
A table joined with itself — useful for comparing rows within the same table (e.g., employee-manager hierarchy).
```sql
SELECT e1.name AS employee, e2.name AS manager
FROM employees e1
JOIN employees e2 ON e1.manager_id = e2.employee_id;
```
> Requires table **aliases** since the same table is referenced twice.

### CROSS JOIN
Produces the **Cartesian product** — every row from table1 combined with every row from table2. No `ON` condition needed.
```sql
SELECT * FROM sizes
CROSS JOIN colors;
```
> ⚠️ Result size = `rows(table1) × rows(table2)`. Can produce huge outputs on large tables — use carefully.

---

## Join on Multiple Columns

Used when a single column isn't enough to uniquely match rows (composite keys).

```sql
SELECT *
FROM table1
JOIN table2
  ON table1.col1 = table2.col1
  AND table1.col2 = table2.col2;
```

> Common with **composite foreign keys** — e.g., matching on `(order_id, product_id)` together instead of just one column.

---

## Join on Multiple Tables

Chain joins one after another — each new `JOIN` can reference any previously joined table.

```sql
SELECT a.col, b.col, c.col
FROM table_a a
JOIN table_b b ON a.id = b.a_id
JOIN table_c c ON b.id = c.b_id;
```

- Can mix join types (`INNER`, `LEFT`, etc.) in the same chain
- Join order generally follows the relationship chain (A→B→C)
- Always alias tables — multi-table queries get unreadable fast without them

```sql
-- Mixed join types example
SELECT o.order_id, c.name, p.product_name
FROM orders o
JOIN customers c ON o.cid = c.cid
LEFT JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id;
```

---

## Set Operations

Combine results of two or more `SELECT` queries. Both queries must have the **same number of columns** with **compatible data types**.

| Operation | Returns |
|---|---|
| `UNION` | Combines results, removes duplicates |
| `UNION ALL` | Combines results, keeps duplicates (faster) |
| `INTERSECT` | Only rows common to both queries |
| `EXCEPT` / `MINUS` | Rows in first query but not in second |

```sql
SELECT col FROM table1
UNION
SELECT col FROM table2;
```

> ⚠️ MySQL supports `UNION` and `UNION ALL` natively. `INTERSECT` and `EXCEPT` were added only in **MySQL 8.0.31+** — on older versions, simulate them with `JOIN` / `NOT IN` / `NOT EXISTS`.

**Rules:**
- Column count must match across all queries
- Result column names come from the **first** query
- Data types must be compatible across corresponding columns

**Simulating INTERSECT (older MySQL):**
```sql
SELECT a.col FROM table1 a
INNER JOIN table2 b ON a.col = b.col;
```

**Simulating EXCEPT (older MySQL):**
```sql
SELECT col FROM table1
WHERE col NOT IN (SELECT col FROM table2);
```

---

## Industry Best Practices

### Always index join columns
JOIN performance depends heavily on whether the columns used in `ON` are indexed — typically the primary key / foreign key pair. Unindexed joins on large tables cause full table scans.

### Prefer explicit JOIN syntax over implicit (comma) joins
```sql
-- ✅ Modern, explicit, readable
SELECT * FROM orders
JOIN customers ON orders.cid = customers.cid;

-- ❌ Old-style implicit join — avoid in production code
SELECT * FROM orders, customers
WHERE orders.cid = customers.cid;
```

### Use UNION ALL instead of UNION when duplicates don't matter
`UNION` runs an implicit `DISTINCT`, which requires sorting/deduplication — expensive on large result sets. If you know there won't be duplicates (or don't care), `UNION ALL` is significantly faster.

### Watch out for CROSS JOIN explosions
Accidentally missing an `ON` condition (or using a comma join with no `WHERE`) silently turns your query into a `CROSS JOIN`, multiplying row counts and potentially crashing the query.

### Use EXISTS instead of JOIN for existence checks
When you only need to check whether a related row exists (not retrieve its data), `EXISTS` is often faster than a `JOIN` since it can stop scanning as soon as one match is found.
```sql
-- ✅ Faster when you just need existence check
SELECT * FROM customers c
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.cid = c.cid);
```

### Verify join cardinality before deploying
A join that unexpectedly matches multiple rows on one side (one-to-many instead of expected one-to-one) silently duplicates rows in the result — leading to inflated `SUM()`/`COUNT()` values. Always sanity-check row counts after joins involving aggregates.

---

*Notes prepared from SQL lecture — DSMP 2022-23*
