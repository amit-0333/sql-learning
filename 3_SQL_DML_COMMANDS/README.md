# 🗄️ SQL DML — Complete Guide

> A comprehensive guide to SQL Data Manipulation Language (DML) — from fundamentals to real-world industry practices.

---

## 📚 Table of Contents

- [What is DML?](#what-is-dml)
- [INSERT](#insert)
- [UPDATE](#update)
- [DELETE](#delete)
- [SELECT Basics](#select-basics)
- [Types of Functions in SQL](#types-of-functions-in-sql)
  - [Aggregate Functions](#aggregate-functions)
  - [Scalar Functions](#scalar-functions)
- [GROUP BY & HAVING](#group-by--having)
- [Practice Questions](#practice-questions)
- [Industry Best Practices](#industry-best-practices)
- [Performance Tips (Experienced Dev POV)](#performance-tips-experienced-dev-pov)

---

## What is DML?

**DML (Data Manipulation Language)** — used to insert, retrieve, update, and delete data in database tables. Unlike DDL (which changes structure), DML works with the **actual data**.

| Command | Purpose |
|---|---|
| `INSERT` | Add new rows |
| `UPDATE` | Modify existing rows |
| `DELETE` | Remove rows |
| `SELECT` | Retrieve/query data |

---

## INSERT

```sql
-- Insert a single row
INSERT INTO smartphones (brand, price, rating)
VALUES ('Samsung', 25000, 4.2);

-- Insert multiple rows in one statement (faster than multiple INSERTs)
INSERT INTO smartphones (brand, price, rating)
VALUES
  ('Apple', 60000, 4.5),
  ('OnePlus', 30000, 4.3),
  ('Xiaomi', 15000, 4.0);

-- Insert data copied from another table
INSERT INTO archived_phones
SELECT * FROM smartphones WHERE price < 5000;
```

> 💡 **Tip:** Always specify column names explicitly in INSERT statements. Relying on column order breaks silently if the table schema changes.

---

## UPDATE

```sql
UPDATE smartphones
SET price = price - 1000
WHERE brand = 'Samsung';

-- Update multiple columns
UPDATE smartphones
SET price = 28000, rating = 4.4
WHERE brand = 'OnePlus' AND price = 30000;
```

> ⚠️ **Always use WHERE with UPDATE.** Forgetting it updates every row in the table — one of the most common production incidents.

---

## DELETE

```sql
-- delete all phones price > 200000
DELETE FROM smartphones WHERE price > 200000;

-- delete with multiple conditions
DELETE FROM smartphones WHERE brand = 'Nokia' AND rating < 2;
```

> ⚠️ Same rule applies — DELETE without WHERE removes **all rows**. Use `TRUNCATE` instead if you truly want all rows gone (faster, resets auto_increment).

---

## SELECT Basics

```sql
SELECT * FROM smartphones;

SELECT brand, price FROM smartphones;

SELECT * FROM smartphones WHERE price > 20000;

SELECT * FROM smartphones ORDER BY price DESC;

SELECT * FROM smartphones LIMIT 10;

-- pagination: skip first 10, get next 10
SELECT * FROM smartphones LIMIT 10 OFFSET 10;
```

---

## Types of Functions in SQL

SQL functions are broadly split into two types:

| Type | Returns | Example |
|---|---|---|
| **Aggregate Functions** | One value per group of rows | `AVG()`, `COUNT()` |
| **Scalar Functions** | One value per row | `ROUND()`, `ABS()` |

---

### Aggregate Functions

Operate on a **set of rows** and return a **single summarized value**.

| Function | Use Case |
|---|---|
| `MAX()` / `MIN()` | Find the minimum and maximum price |
| `AVG()` | Find avg rating of Apple phones |
| `SUM()` | Total of a numeric column |
| `COUNT()` | Find the number of OnePlus phones |
| `COUNT(DISTINCT col)` | Find the number of unique brands available |
| `STD()` | Find standard deviation of screen sizes |
| `VARIANCE()` | Find variance of Xiaomi phone prices |

```sql
SELECT MIN(price), MAX(price) FROM smartphones;

SELECT MAX(price) FROM smartphones WHERE brand = 'Samsung';

SELECT AVG(rating) FROM smartphones WHERE brand = 'Apple';

SELECT COUNT(*) FROM smartphones WHERE brand = 'OnePlus';

SELECT COUNT(DISTINCT brand) FROM smartphones;

SELECT STD(screen_size) FROM smartphones;

SELECT VARIANCE(price) FROM smartphones WHERE brand = 'Xiaomi';
```

> 💡 `COUNT(*)` counts all rows (including NULLs). `COUNT(column)` ignores NULLs in that column.

---

### Scalar Functions

Operate on **each row individually**, returning one output per row.

| Function | Use Case |
|---|---|
| `ABS()` | Difference from avg rating of all Samsung phones |
| `ROUND()` | Round the ppi to 1 decimal place |
| `CEIL()` / `FLOOR()` | Round rating up/down |

```sql
SELECT rating,
       ABS(rating - (SELECT AVG(rating) FROM smartphones WHERE brand = 'Samsung')) AS diff_from_avg
FROM smartphones
WHERE brand = 'Samsung';

SELECT ROUND(ppi, 1) FROM smartphones;

SELECT CEIL(rating), FLOOR(rating) FROM smartphones;
```

**Other commonly used scalar functions (industry use):**
```sql
-- String functions
SELECT UPPER(brand), LOWER(brand), LENGTH(brand) FROM smartphones;
SELECT CONCAT(brand, ' - ', model) AS full_name FROM smartphones;
SELECT TRIM(brand) FROM smartphones;

-- Date functions
SELECT NOW(), CURDATE(), CURTIME();
SELECT DATEDIFF(NOW(), created_at) AS days_old FROM smartphones;
SELECT DATE_FORMAT(created_at, '%d-%m-%Y') FROM smartphones;

-- Conditional
SELECT IF(price > 50000, 'Premium', 'Budget') AS category FROM smartphones;
SELECT IFNULL(rating, 0) FROM smartphones;
```

---

## GROUP BY & HAVING

Aggregate functions are most powerful when combined with `GROUP BY`.

```sql
-- Average price per brand
SELECT brand, AVG(price) AS avg_price
FROM smartphones
GROUP BY brand;

-- Brands with more than 10 phones listed
SELECT brand, COUNT(*) AS total
FROM smartphones
GROUP BY brand
HAVING COUNT(*) > 10;
```

> 💡 **WHERE filters rows before grouping. HAVING filters groups after aggregation.** This is one of the most commonly asked interview questions.

---

## Practice Questions

```sql
-- Average battery capacity & avg primary rear camera resolution for price >= 100000
SELECT AVG(battery_capacity), AVG(primary_camera_rear)
FROM smartphones
WHERE price >= 100000;

-- Average internal memory for refresh_rate >= 120Hz and front camera >= 20MP
SELECT AVG(internal_memory)
FROM smartphones
WHERE refresh_rate >= 120 AND primary_camera_front >= 20;

-- Number of smartphones with 5G capability
SELECT COUNT(*)
FROM smartphones
WHERE has_5g = TRUE;
```

---

## Industry Best Practices

### Never run UPDATE/DELETE without a WHERE clause check
```sql
-- Always run SELECT first to preview affected rows
SELECT * FROM smartphones WHERE brand = 'Nokia';

-- Then convert to DELETE/UPDATE once confirmed
DELETE FROM smartphones WHERE brand = 'Nokia';
```

### Use Transactions for risky operations
```sql
START TRANSACTION;
  UPDATE smartphones SET price = price * 0.9 WHERE brand = 'Samsung';
COMMIT;
-- or ROLLBACK; if results look wrong
```

### Batch large INSERT/UPDATE/DELETE operations
Running a single DELETE/UPDATE on millions of rows locks the table and can crash production. Companies batch it:
```sql
DELETE FROM logs WHERE created_at < '2023-01-01' LIMIT 1000;
-- Repeat in a loop until 0 rows affected
```

### Soft delete instead of hard delete
```sql
UPDATE smartphones SET is_deleted = TRUE WHERE id = 5;
-- instead of
DELETE FROM smartphones WHERE id = 5;
```

### Use parameterized queries (prevent SQL Injection)
```sql
-- ❌ Never build queries with string concatenation
"SELECT * FROM users WHERE email = '" + userInput + "'"

-- ✅ Use prepared statements / parameter binding
SELECT * FROM users WHERE email = ?;
```

### Avoid SELECT * in production code
```sql
-- ❌ Pulls unnecessary columns, breaks if schema changes
SELECT * FROM smartphones;

-- ✅ Select only what you need
SELECT brand, price, rating FROM smartphones;
```

---

## Performance Tips (Experienced Dev POV)

| Tip | Why |
|---|---|
| Index columns used in `WHERE`, `JOIN`, `GROUP BY` | Speeds up lookups significantly |
| Avoid functions on indexed columns in WHERE (`WHERE YEAR(date) = 2024`) | Disables index usage — use range instead (`WHERE date >= '2024-01-01'`) |
| Use `EXPLAIN` before optimizing | Shows query execution plan, identifies full table scans |
| Use `LIMIT` when testing queries on large tables | Avoids accidentally pulling millions of rows |
| Prefer `EXISTS` over `IN` for subqueries on large datasets | Generally faster, especially with NULLs |
| Use `COUNT(1)` or `COUNT(*)` instead of `COUNT(column)` for row counts | Slightly faster, no NULL-checking overhead |

```sql
-- Check query performance
EXPLAIN SELECT * FROM smartphones WHERE brand = 'Samsung';
```

---

*Notes prepared from SQL DML lecture — DSMP 2022-23*
