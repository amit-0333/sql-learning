# MySQL Views & User Defined Functions — Complete Reference

> A practical guide covering SQL views, materialized views, user-defined functions (UDFs), and real-world usage patterns.

---

## Table of Contents

1. [Views](#1-views)
2. [User Defined Functions (UDFs)](#2-user-defined-functions-udfs)
3. [Views vs UDFs vs Stored Procedures](#3-views-vs-udfs-vs-stored-procedures)
4. [Pro Tips & Patterns](#4-pro-tips--patterns)

---

## 1. Views

### What is a View?

A view is a **virtual table** — it stores no data itself but presents a customized SELECT query as if it were a table. Think of it as a saved query with a name.

- Changes in the underlying table are **automatically reflected** in the view
- It's a **logical table**, not a physical table
- Can be used exactly like a real table in queries

```
Table (physical) → View (logical/virtual) → SQL queries → results
```

---

### Types of Views

| Type | Description |
|------|-------------|
| **Simple View** | Based on a single table, no aggregates |
| **Complex View** | Based on multiple tables using JOINs, subqueries, aggregates |

---

### Create, Use, Update, Drop

```sql
-- Create a simple view
CREATE VIEW indigo AS
SELECT * FROM flights
WHERE airline = 'Indigo';

-- Use it like a normal table
SELECT * FROM indigo;
SELECT * FROM indigo WHERE source = 'Delhi';

-- Replace/update a view
CREATE OR REPLACE VIEW indigo AS
SELECT flight_id, source, destination, price
FROM flights
WHERE airline = 'Indigo';

-- Drop a view
DROP VIEW IF EXISTS indigo;

-- Show all views in a database
SHOW FULL TABLES WHERE TABLE_TYPE = 'VIEW';

-- See view definition
SHOW CREATE VIEW indigo;
```

---

### Simple View Examples

```sql
-- View: only active users
CREATE VIEW active_users AS
SELECT id, name, email FROM users
WHERE status = 'active';

-- View: hide sensitive columns (security)
CREATE VIEW public_employees AS
SELECT id, name, department, job_title
FROM employees;
-- salary, ssn, bank_details columns are hidden

-- View: recent orders
CREATE VIEW recent_orders AS
SELECT * FROM orders
WHERE order_date >= DATE_SUB(NOW(), INTERVAL 30 DAY);
```

---

### Complex View Examples

```sql
-- View: order summary with customer and product info
CREATE VIEW order_summary AS
SELECT
    o.order_id,
    c.name         AS customer_name,
    p.name         AS product_name,
    oi.quantity,
    o.total_amount,
    o.order_date
FROM orders o
JOIN customers c  ON o.customer_id = c.id
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p   ON oi.product_id = p.id;

-- Now use it simply
SELECT * FROM order_summary WHERE customer_name = 'Nitish';

-- View: monthly revenue
CREATE VIEW monthly_revenue AS
SELECT
    MONTHNAME(order_date) AS month,
    YEAR(order_date)      AS year,
    SUM(total_amount)     AS revenue,
    COUNT(*)              AS total_orders
FROM orders
GROUP BY YEAR(order_date), MONTH(order_date);
```

---

### Read-only vs Updatable Views

**Read-only views** — can only SELECT, cannot INSERT/UPDATE/DELETE.

**Updatable views** — allow modifying data in the underlying table. Conditions:
- Must NOT contain GROUP BY, DISTINCT, aggregate functions
- Must NOT contain subqueries in SELECT
- Must NOT use UNION
- Must be based on a single table (or 1:1 join)

```sql
-- ✅ Updatable view (single table, no aggregates)
CREATE VIEW active_users AS
SELECT id, name, email, status FROM users
WHERE status = 'active';

-- UPDATE goes to the actual users table
UPDATE active_users SET email = 'new@email.com' WHERE id = 1;
INSERT INTO active_users (name, email, status) VALUES ('Raj', 'raj@email.com', 'active');
DELETE FROM active_users WHERE id = 5;

-- ❌ NOT updatable (has aggregate)
CREATE VIEW user_count AS
SELECT status, COUNT(*) AS total FROM users GROUP BY status;
-- Cannot UPDATE/INSERT/DELETE this view
```

---

### WITH CHECK OPTION

Prevents inserting/updating rows through a view that would make them disappear from the view.

```sql
CREATE VIEW active_users AS
SELECT * FROM users WHERE status = 'active'
WITH CHECK OPTION;

-- ❌ This fails — row wouldn't be visible in the view after insert
INSERT INTO active_users VALUES (10, 'John', 'inactive');

-- ✅ This works
INSERT INTO active_users VALUES (10, 'John', 'active');
```

---

### Materialized Views

Unlike regular views (store only SQL query), a **materialized view stores actual result data** on disk — faster but needs manual refresh.

| | Regular View | Materialized View |
|-|-------------|------------------|
| Stores | SQL query only | Actual result data |
| Speed | Slower (re-runs query each time) | Faster (pre-computed) |
| Data freshness | Always current | Needs manual refresh |
| MySQL support | ✅ Native | ❌ Not native (simulate manually) |

```sql
-- MySQL doesn't support materialized views natively
-- Simulate by creating a real table

-- Create materialized view table
CREATE TABLE mat_monthly_revenue AS
SELECT
    MONTH(order_date) AS month,
    YEAR(order_date)  AS year,
    SUM(total_amount) AS revenue
FROM orders
GROUP BY YEAR(order_date), MONTH(order_date);

-- Refresh manually (e.g. via scheduled event or procedure)
TRUNCATE TABLE mat_monthly_revenue;
INSERT INTO mat_monthly_revenue
SELECT
    MONTH(order_date),
    YEAR(order_date),
    SUM(total_amount)
FROM orders
GROUP BY YEAR(order_date), MONTH(order_date);
```

> In PostgreSQL and Oracle, materialized views are natively supported with `REFRESH MATERIALIZED VIEW`.

---

### Advantages of Views

| Advantage | Description |
|-----------|-------------|
| **No physical storage** | Saves disk space — just a stored query |
| **Security** | Hide sensitive columns/rows from certain users |
| **Simplicity** | Wrap complex JOINs into a simple table name |
| **Consistency** | Everyone queries the same pre-defined logic |
| **Abstraction** | App code doesn't need to know underlying table structure |

---

## 2. User Defined Functions (UDFs)

### What are UDFs?

Functions created by users to perform specific tasks. Work exactly like built-in functions (UPPER, LENGTH, etc.) — take input, perform operations, return a single value.

**Benefits:**
- Simplifies complex or repeated SQL logic
- Reusability — write once, use in any query
- Enhances readability of queries

---

### Syntax

```sql
DELIMITER $$

CREATE FUNCTION function_name(
    param1 DATATYPE,
    param2 DATATYPE
)
RETURNS return_datatype
[NOT] DETERMINISTIC
BEGIN
    -- function body
    RETURN return_value;
END $$

DELIMITER ;

-- Use in a query
SELECT function_name(args);
SELECT function_name(column) FROM table;
```

---

### DETERMINISTIC vs NOT DETERMINISTIC

| | DETERMINISTIC | NOT DETERMINISTIC |
|-|--------------|------------------|
| Meaning | Same input → always same output | Output can vary |
| Examples | Math, string operations | NOW(), RAND(), DB queries |
| Performance | Better (MySQL can cache) | Slower |

```sql
-- DETERMINISTIC — pure calculation, no side effects
CREATE FUNCTION add_tax(price DECIMAL(10,2))
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    RETURN price * 1.18;  -- 18% GST
END;

-- NOT DETERMINISTIC — depends on DB state or time
CREATE FUNCTION get_order_count(user_id INT)
RETURNS INT
NOT DETERMINISTIC
BEGIN
    DECLARE cnt INT;
    SELECT COUNT(*) INTO cnt FROM orders WHERE user_id = user_id;
    RETURN cnt;
END;
```

---

### Example 1 — Hello World (Non-Parameterized)

```sql
DELIMITER $$
CREATE FUNCTION hello_world()
RETURNS VARCHAR(50)
DETERMINISTIC
BEGIN
    RETURN 'Hello World';
END $$
DELIMITER ;

SELECT hello_world();   -- 'Hello World'
```

---

### Example 2 — Calculate Age

```sql
DELIMITER $$
CREATE FUNCTION calculate_age(dob DATE)
RETURNS INT
DETERMINISTIC
BEGIN
    RETURN FLOOR(DATEDIFF(NOW(), dob) / 365);
END $$
DELIMITER ;

SELECT calculate_age('1995-08-15');                    -- 29
SELECT name, calculate_age(birth_date) AS age FROM users;
```

---

### Example 3 — Conditional Title (IF/ELSEIF)

```sql
DELIMITER $$
CREATE FUNCTION greet(name VARCHAR(100), gender VARCHAR(10))
RETURNS VARCHAR(100)
DETERMINISTIC
BEGIN
    IF gender = 'male' THEN
        RETURN CONCAT('Mr. ', name);
    ELSEIF gender = 'female' THEN
        RETURN CONCAT('Ms. ', name);
    ELSE
        RETURN CONCAT('Dear ', name);
    END IF;
END $$
DELIMITER ;

SELECT greet('Nitish', 'male');    -- 'Mr. Nitish'
SELECT greet('Priya', 'female');   -- 'Ms. Priya'
SELECT greet('Alex', 'other');     -- 'Dear Alex'

-- Use on a column
SELECT greet(name, gender) AS greeting FROM users;
```

---

### Example 4 — Date Formatting

```sql
DELIMITER $$
CREATE FUNCTION format_date(dt DATETIME)
RETURNS VARCHAR(50)
DETERMINISTIC
BEGIN
    RETURN DATE_FORMAT(dt, '%d %b %Y');
END $$
DELIMITER ;

SELECT format_date(NOW());               -- '27 Jun 2026'
SELECT format_date(order_date) FROM orders;
```

---

### Example 5 — Flights Between 2 Cities (Not Deterministic)

```sql
DELIMITER $$
CREATE FUNCTION flights_between(city1 VARCHAR(100), city2 VARCHAR(100))
RETURNS INT
NOT DETERMINISTIC
BEGIN
    DECLARE total INT;
    SELECT COUNT(*) INTO total
    FROM flights
    WHERE source = city1 AND destination = city2;
    RETURN total;
END $$
DELIMITER ;

SELECT flights_between('Delhi', 'Mumbai');   -- e.g. 12
SELECT flights_between('Bangalore', 'Pune');
```

---

### Example 6 — Tax Calculator with CASE

```sql
DELIMITER $$
CREATE FUNCTION apply_tax(price DECIMAL(10,2), category VARCHAR(50))
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    DECLARE tax_rate DECIMAL(5,2);

    CASE category
        WHEN 'food'        THEN SET tax_rate = 0.05;   -- 5%
        WHEN 'electronics' THEN SET tax_rate = 0.18;   -- 18%
        WHEN 'luxury'      THEN SET tax_rate = 0.28;   -- 28%
        ELSE                    SET tax_rate = 0.12;   -- 12% default
    END CASE;

    RETURN price + (price * tax_rate);
END $$
DELIMITER ;

SELECT apply_tax(1000, 'electronics');  -- 1180.00
SELECT apply_tax(500, 'food');          -- 525.00
```

---

### Manage Functions

```sql
-- Show all functions in a database
SHOW FUNCTION STATUS WHERE db = 'your_database';

-- View function code
SHOW CREATE FUNCTION calculate_age;

-- Drop a function
DROP FUNCTION IF EXISTS calculate_age;
```

---

## 3. Views vs UDFs vs Stored Procedures

| Feature | View | UDF | Stored Procedure |
|---------|------|-----|------------------|
| Returns | Result set (table) | Single value | Nothing (or OUT params) |
| Used in SELECT | ✅ Yes | ✅ Yes | ❌ No |
| Called with | SELECT/FROM | SELECT | CALL |
| Can have logic | ❌ No | ✅ Yes | ✅ Yes |
| Can modify data | Updatable views only | ❌ No | ✅ Yes |
| Transactions | ❌ No | ❌ No | ✅ Yes |
| Performance | Query runs each time | Cached if deterministic | Precompiled |

---

## 4. Pro Tips & Patterns

### Use Views for Security

```sql
-- Create role-based views
-- HR can see salary, regular employees cannot

CREATE VIEW employee_public AS
SELECT id, name, department, job_title FROM employees;

CREATE VIEW employee_hr AS
SELECT id, name, department, job_title, salary, ssn FROM employees;

-- Grant access to specific views
GRANT SELECT ON mydb.employee_public TO 'app_user'@'localhost';
GRANT SELECT ON mydb.employee_hr TO 'hr_user'@'localhost';
```

### Use Views to Simplify Reporting Queries

```sql
-- Without view — complex repeated query
SELECT
    u.name, COUNT(o.id) AS orders, SUM(o.amount) AS total
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.name;

-- With view — simple
CREATE VIEW user_order_summary AS
SELECT
    u.name, COUNT(o.id) AS orders, SUM(o.amount) AS total
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.name;

SELECT * FROM user_order_summary WHERE total > 10000;
```

### Chain UDFs for Clean Queries

```sql
-- Without UDFs — messy
SELECT
    CONCAT(
        CASE WHEN gender = 'male' THEN 'Mr. ' ELSE 'Ms. ' END,
        name
    ),
    FLOOR(DATEDIFF(NOW(), birth_date) / 365),
    price * 1.18
FROM users;

-- With UDFs — clean and readable
SELECT
    greet(name, gender),
    calculate_age(birth_date),
    add_tax(price)
FROM users;
```

### Refresh Strategy for Simulated Materialized Views

```sql
-- Create a scheduled event to refresh every night at midnight
CREATE EVENT refresh_monthly_revenue
ON SCHEDULE EVERY 1 DAY
STARTS '2023-01-01 00:00:00'
DO
BEGIN
    TRUNCATE TABLE mat_monthly_revenue;
    INSERT INTO mat_monthly_revenue
    SELECT MONTH(order_date), YEAR(order_date), SUM(total_amount)
    FROM orders GROUP BY YEAR(order_date), MONTH(order_date);
END;
```

---

## Useful Links

| Topic | Link |
|-------|------|
| MySQL Views | https://dev.mysql.com/doc/refman/8.0/en/views.html |
| Stored Functions | https://dev.mysql.com/doc/refman/8.0/en/stored-routines.html |
| CREATE VIEW | https://dev.mysql.com/doc/refman/8.0/en/create-view.html |

> 🔍 **Search:** `MySQL CREATE VIEW syntax` · `MySQL user defined functions` · `MySQL materialized view workaround`

---

*Reference based on MySQL 8.0 documentation + real-world usage patterns.*
