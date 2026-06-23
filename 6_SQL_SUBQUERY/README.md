# 🗄️ SQL — Subqueries Complete Guide

> A complete guide to SQL Subqueries — from fundamentals to real-world usage across SELECT, INSERT, UPDATE, DELETE.

---

## 📚 Table of Contents

- [What is a Subquery?](#what-is-a-subquery)
- [Types of Subqueries](#types-of-subqueries)
- [Where Can Subqueries Be Used?](#where-can-subqueries-be-used)
- [Independent Subqueries](#independent-subqueries)
- [Correlated Subquery](#correlated-subquery)
- [Usage with SELECT](#usage-with-select)
- [Usage with FROM](#usage-with-from)
- [Usage with HAVING](#usage-with-having)
- [Subquery in INSERT](#subquery-in-insert)
- [Subquery in UPDATE](#subquery-in-update)
- [Subquery in DELETE](#subquery-in-delete)
- [How to Solve Any Subquery Question](#how-to-solve-any-subquery-question)
- [Industry Best Practices](#industry-best-practices)

---

## What is a Subquery?

A subquery is a **query nested inside another query** — a SELECT statement embedded inside another SELECT, INSERT, UPDATE, or DELETE statement. The inner query executes **first**, and its result is used by the outer query as a value, list, or table.

```sql
SELECT * FROM movies
WHERE score = (SELECT MAX(score) FROM movies);
--             ↑ inner query runs first, returns one value
-- outer query uses that value to filter rows
```

> ⚠️ This topic needs practice — the difficulty isn't the syntax, it's correctly identifying what the inner query should calculate.

---

## Types of Subqueries

Subqueries are classified in **two ways:**

### By Result Returned

| Type | Returns | Used With |
|---|---|---|
| **Scalar** | 1 row, 1 column (single value) | `=`, `>`, `<` |
| **Row** | Multiple rows, 1 column | `IN`, `NOT IN` |
| **Table** | Multiple rows, multiple columns | `FROM`, `JOIN`, `(col1,col2) IN` |

### By Working

| Type | Behavior |
|---|---|
| **Independent** | Inner query runs once, completely on its own |
| **Correlated** | Inner query references the outer query — runs once **per row** of the outer query |

---

## Where Can Subqueries Be Used?

```
INSERT
SELECT  →  WHERE / SELECT / FROM / HAVING
UPDATE
DELETE
```

---

## Independent Subqueries

### Scalar Subquery
Returns one single value → use `=`, `>`, `<`.

```sql
-- Find movie with highest profit
SELECT * FROM movies
WHERE (gross - budget) = (SELECT MAX(gross - budget) FROM movies);

-- Find movies with above-average rating
SELECT * FROM movies
WHERE score > (SELECT AVG(score) FROM movies);

-- Find highest rated movie of year 2000
SELECT * FROM movies
WHERE release_year = 2000
AND score = (SELECT MAX(score) FROM movies WHERE release_year = 2000);
```

### Row Subquery
Returns multiple rows, one column → use `IN` / `NOT IN`.

```sql
-- Find all users who never ordered
SELECT * FROM users
WHERE user_id NOT IN (SELECT DISTINCT user_id FROM orders);

-- Find movies by top 3 directors (by total gross)
SELECT * FROM movies
WHERE director IN (
    SELECT director FROM movies
    GROUP BY director
    ORDER BY SUM(gross) DESC
    LIMIT 3
);
```

### Table Subquery
Returns multiple rows AND multiple columns → use `(col1, col2) IN (subquery)`.

```sql
-- Find most profitable movie of each year
SELECT * FROM movies
WHERE (release_year, gross - budget) IN (
    SELECT release_year, MAX(gross - budget)
    FROM movies
    GROUP BY release_year
);

-- Find highest rated movie of each genre (votes > 25000)
SELECT * FROM movies
WHERE (genre, score) IN (
    SELECT genre, MAX(score)
    FROM movies
    WHERE votes > 25000
    GROUP BY genre
);
```

---

## Correlated Subquery

The inner query **references a column from the outer query** — it re-executes once for every row the outer query processes.

```sql
-- Find movies with rating higher than avg of their own genre
SELECT * FROM movies m1
WHERE score > (
    SELECT AVG(score) FROM movies m2
    WHERE m2.genre = m1.genre  -- ← references outer query's current row
);
```

> `m1.genre` is the key — it ties the inner query to whatever row the outer query is currently looking at. This is what makes it "correlated."

---

## Usage with SELECT

Subquery sits inside the SELECT column list — computes a per-row value.

```sql
-- Percentage of votes each movie holds vs total votes
SELECT name,
       (votes / (SELECT SUM(votes) FROM movies)) * 100 AS vote_pct
FROM movies;

-- Each movie's score alongside the avg score of its genre
SELECT name, genre, score,
       (SELECT AVG(score) FROM movies m2 WHERE m2.genre = m1.genre) AS avg_genre_score
FROM movies m1;
```

> ⚠️ **Why this is inefficient:** the correlated subquery in SELECT re-runs once per row. On 1000 movies, that's 1000 separate subquery executions. Prefer `FROM` subquery approach for large tables.

---

## Usage with FROM

Subquery acts as a **temporary virtual table** — wrap it in `FROM`, give it an alias, then query/join it like a normal table.

```sql
-- Avg restaurant rating, then join to get restaurant name
SELECT r2.r_name, t1.avg_rating
FROM (
    SELECT r_id, AVG(restaurant_rating) AS avg_rating
    FROM orders
    GROUP BY r_id
) t1
JOIN restaurants r2 ON t1.r_id = r2.r_id;
```

> The inner query pre-aggregates **once**, then the outer query works with that smaller result — much more efficient than a correlated subquery in SELECT.

---

## Usage with HAVING

Compare an aggregated group value against another aggregated value from a subquery.

```sql
-- Find genres with avg score above overall avg score
SELECT genre, AVG(score)
FROM movies
GROUP BY genre
HAVING AVG(score) > (SELECT AVG(score) FROM movies);
```

---

## Subquery in INSERT

Instead of typing values manually, use SELECT to pull/compute data to insert. No `VALUES` keyword needed.

```sql
-- Populate loyal_users with customers who ordered more than 3 times
INSERT INTO loyal_users (user_id, name)
SELECT t1.user_id, t2.name
FROM orders t1
JOIN users t2 ON t1.user_id = t2.user_id
GROUP BY t1.user_id
HAVING COUNT(*) > 3;
```

---

## Subquery in UPDATE

Update a column using a value calculated from another table via subquery.

```sql
-- Give 10% of total order value as loyalty money (correlated)
UPDATE loyal_users
SET money = (
    SELECT SUM(amount) * 0.1
    FROM orders
    WHERE orders.user_id = loyal_users.user_id
);
```

---

## Subquery in DELETE

Delete rows based on a condition computed via subquery.

```sql
-- Delete customers who never ordered
DELETE FROM users
WHERE user_id NOT IN (
    SELECT DISTINCT user_id FROM orders
);
```

---

## How to Solve Any Subquery Question

### The 4-Step Framework

**Step 1: Identify the action word**
- "Find/show/get" → `SELECT`
- "Add/populate" → `INSERT`
- "Change/update/give" → `UPDATE`
- "Remove/delete" → `DELETE`

**Step 2: Identify the target table**

**Step 3: Break the sentence — find filter words**
- "who/that have..." → `WHERE`
- "of each / per..." → `GROUP BY`
- "more than X..." → comparison
- "compared to / vs / than the overall average..." → 🚨 subquery needed

**Step 4: Is the comparison value fixed or calculated?**

| Phrase | Subquery needed? |
|---|---|
| "price > 50000" | ❌ No — fixed number |
| "price > average price" | ✅ Yes — must calculate |
| "more than 3 orders" | ❌ No — fixed number |
| "more than the avg orders per customer" | ✅ Yes — must calculate |

### Which Type of Subquery?

```
Does inner query return ONE value?
  → Scalar → use =, >, <

Multiple rows, ONE column?
  → Row → use IN, NOT IN

Multiple rows, MULTIPLE columns?
  → Table → use (col1, col2) IN (...) or FROM

Does inner query reference outer query's column?
  → YES: Correlated (re-runs per row)
  → NO: Independent (runs once)
```

---

## Industry Best Practices

### Prefer FROM subquery over correlated SELECT subquery
```sql
-- ❌ Slow: correlated, runs once per row
SELECT name, (SELECT AVG(score) FROM movies m2 WHERE m2.genre = m1.genre)
FROM movies m1;

-- ✅ Fast: aggregates once, then joins
SELECT m.name, g.avg_score
FROM movies m
JOIN (SELECT genre, AVG(score) AS avg_score FROM movies GROUP BY genre) g
ON m.genre = g.genre;
```

### Be careful with NOT IN and NULLs
```sql
-- ⚠️ If orders.user_id has NULLs, NOT IN returns zero rows silently
WHERE user_id NOT IN (SELECT user_id FROM orders)

-- ✅ Safer alternative
WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.user_id)
```

### Name your subquery aliases clearly
```sql
-- ❌ Hard to read
FROM (SELECT ...) t1

-- ✅ Self-documenting
FROM (SELECT ...) genre_averages
```

### Use CTEs (WITH clause) for complex subqueries
Instead of nesting multiple subqueries (hard to read), use CTEs:
```sql
WITH loyal AS (
    SELECT user_id, COUNT(*) AS order_count
    FROM orders
    GROUP BY user_id
    HAVING COUNT(*) > 3
)
SELECT u.name, loyal.order_count
FROM users u
JOIN loyal ON u.user_id = loyal.user_id;
```

---

*Notes prepared from SQL Subquery lecture — DSMP 2022-23*
