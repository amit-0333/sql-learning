# 🪟 SQL — Window Functions Complete Guide

> A complete guide to SQL Window Functions — from ranking to running totals, used heavily in data analytics and business reporting.

---

## 📚 Table of Contents

- [What are Window Functions?](#what-are-window-functions)
- [Syntax](#syntax)
- [RANK / DENSE_RANK / ROW_NUMBER](#rank--dense_rank--row_number)
- [PARTITION BY](#partition-by)
- [FIRST_VALUE / LAST_VALUE / NTH_VALUE](#first_value--last_value--nth_value)
- [Frames](#frames)
- [WINDOW Clause (Alias)](#window-clause-alias)
- [LEAD & LAG](#lead--lag)
- [Cumulative Sum](#cumulative-sum)
- [Cumulative Average](#cumulative-average)
- [Running Average](#running-average)
- [Percent of Total](#percent-of-total)
- [Percent Change](#percent-change)
- [Industry Use Cases](#industry-use-cases)

---

## What are Window Functions?

A window function performs a calculation across a **set of rows related to the current row** — without collapsing rows like `GROUP BY` does. Every row keeps its identity while also getting an extra calculated column.

**Key difference from GROUP BY:**

| | GROUP BY | Window Function |
|---|---|---|
| Rows in output | One per group | Same as input |
| Individual row values | Lost | Kept |
| Use case | Summarize | Rank, compare, run totals |

---

## Syntax

```sql
function_name() OVER (
    PARTITION BY col    -- optional: divide into groups
    ORDER BY col        -- optional: define row order within group
    ROWS BETWEEN ...    -- optional: define frame
)
```

- `OVER()` — required, even if empty
- `PARTITION BY` — like GROUP BY but doesn't collapse rows
- `ORDER BY` — determines the order within each partition
- `ROWS BETWEEN` — defines the frame (range of rows to include)

---

## RANK / DENSE_RANK / ROW_NUMBER

All three assign a number to each row based on ordering. They differ in how they handle **ties**.

| Function | Ties | Gaps after tie? |
|---|---|---|
| `ROW_NUMBER()` | Unique number even for ties | No gaps |
| `RANK()` | Same rank for ties | Yes — skips next rank |
| `DENSE_RANK()` | Same rank for ties | No — continues sequentially |

**Example with marks:**
| name | marks | ROW_NUMBER | RANK | DENSE_RANK |
|---|---|---|---|---|
| Deepak | 98 | 1 | 1 | 1 |
| Arjun | 95 | 2 | 2 | 2 |
| Vinay | 95 | 3 | 2 | 2 |
| Rohit | 80 | 4 | 4 | 3 |

```sql
-- Global ranking by marks
SELECT *, RANK() OVER(ORDER BY marks DESC) FROM marks;

-- Global ranking with no gaps
SELECT *, DENSE_RANK() OVER(ORDER BY marks DESC) FROM marks;

-- Simple row number (no order = arbitrary)
SELECT *, ROW_NUMBER() OVER() FROM marks;

-- Rank within each branch
SELECT *, RANK() OVER(PARTITION BY branch ORDER BY marks DESC) FROM marks;

SELECT *,
    RANK()       OVER(PARTITION BY branch ORDER BY marks DESC),
    DENSE_RANK() OVER(PARTITION BY branch ORDER BY marks DESC)
FROM marks;
```

---

## PARTITION BY

Divides rows into groups (partitions) — the window function then runs **independently within each partition**.

```sql
-- Avg marks per branch, shown alongside each student row
SELECT *, AVG(marks) OVER(PARTITION BY branch) AS branch_avg
FROM marks;

-- Find students scoring below their branch average
SELECT * FROM (
    SELECT *, AVG(marks) OVER(PARTITION BY branch) AS branch_avg
    FROM marks
) t
WHERE t.marks < t.branch_avg;
```

**Creative use — generate roll numbers per branch:**
```sql
SELECT *,
    CONCAT(branch, '-', ROW_NUMBER() OVER(PARTITION BY branch)) AS roll_number
FROM marks;
-- Output: CSE-1, CSE-2, ECE-1, ECE-2 etc.
```

**Rank customers per month by spending (Zomato):**
```sql
SELECT MONTHNAME(date), user_id, SUM(amount),
    RANK() OVER(PARTITION BY MONTHNAME(date) ORDER BY SUM(amount) DESC)
FROM orders
GROUP BY MONTHNAME(date), user_id
ORDER BY MONTH(date);
```

**Rank batsmen within each IPL team:**
```sql
SELECT BattingTeam, batter, SUM(batsman_run) AS total_runs,
    DENSE_RANK() OVER(PARTITION BY BattingTeam ORDER BY SUM(batsman_run) DESC) AS rank_within_team
FROM ipl
GROUP BY BattingTeam, batter;
```

---

## FIRST_VALUE / LAST_VALUE / NTH_VALUE

Return the value of an expression from the first, last, or Nth row of the window frame.

```sql
-- Topper name shown on every row (global)
SELECT *, FIRST_VALUE(name) OVER(ORDER BY marks DESC) FROM marks;

-- Topper of each branch shown on every row
SELECT *, FIRST_VALUE(name) OVER(PARTITION BY branch ORDER BY marks DESC) AS topper
FROM marks;

-- Last (lowest) marks in each branch — needs FRAME clause
SELECT *, LAST_VALUE(marks) OVER(
    PARTITION BY branch ORDER BY marks DESC
    ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
) FROM marks;

-- 2nd topper of each branch
SELECT *, NTH_VALUE(name, 2) OVER(
    PARTITION BY branch ORDER BY marks DESC
    ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
) FROM marks;
```

**Find branch toppers (actual rows, not just the name column):**
```sql
SELECT * FROM (
    SELECT *,
        FIRST_VALUE(name)  OVER(PARTITION BY branch ORDER BY marks DESC) AS topper_name,
        FIRST_VALUE(marks) OVER(PARTITION BY branch ORDER BY marks DESC) AS topper_marks
    FROM marks
) t
WHERE t.name = t.topper_name AND t.marks = t.topper_marks;
```

---

## Frames

A frame is a **subset of rows within the partition** that determines the scope of the window function's calculation. Defined using `ROWS BETWEEN`.

```sql
ROWS BETWEEN <start> AND <end>
```

**Keywords:**
- `UNBOUNDED PRECEDING` — from the very first row of the partition
- `CURRENT ROW` — the current row
- `UNBOUNDED FOLLOWING` — to the very last row of the partition
- `N PRECEDING` — N rows before current
- `N FOLLOWING` — N rows after current

**Common frame patterns:**

| Frame | Meaning |
|---|---|
| `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` | All rows from start up to current (default for running totals) |
| `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` | All rows in the partition (needed for LAST_VALUE to work correctly) |
| `ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING` | Current row + 1 before + 1 after |
| `ROWS BETWEEN 3 PRECEDING AND 2 FOLLOWING` | 3 rows before + current + 2 rows after |

> ⚠️ `LAST_VALUE` without a frame defaults to `CURRENT ROW` as the end — which means it only looks at the current row. Always add `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` when using `LAST_VALUE`.

---

## WINDOW Clause (Alias)

When the same window spec is repeated multiple times, name it once with `WINDOW` at the end:

```sql
-- Without WINDOW alias (repetitive)
SELECT *,
    LAST_VALUE(name)  OVER(PARTITION BY branch ORDER BY marks DESC ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING),
    LAST_VALUE(marks) OVER(PARTITION BY branch ORDER BY marks DESC ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)
FROM marks;

-- With WINDOW alias (clean)
SELECT *,
    LAST_VALUE(name)  OVER w AS last_name,
    LAST_VALUE(marks) OVER w AS last_marks
FROM marks
WINDOW w AS (PARTITION BY branch ORDER BY marks DESC ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING);
```

---

## LEAD & LAG

Access the value of a row **before or after** the current row — used for comparing current vs previous period.

```sql
LAG(col, n, default)   -- value n rows BEFORE current row
LEAD(col, n, default)  -- value n rows AFTER current row
```

```sql
-- Previous student's marks (by student_id order)
SELECT *, LAG(marks) OVER(ORDER BY student_id) FROM marks;

-- MoM revenue growth (Zomato example)
SELECT *,
    (revenue - LAG(revenue) OVER(ORDER BY month)) / LAG(revenue) OVER(ORDER BY month) * 100 AS mom_growth
FROM monthly_revenue;
```

---

## Cumulative Sum

Sum of all values from the beginning of the partition up to the current row — grows with each row.

```sql
SELECT month, sales,
    SUM(sales) OVER(ORDER BY month ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS cumulative_sum
FROM monthly_sales;
```

**V Kohli career runs example (IPL):**
```sql
SELECT * FROM (
    SELECT
        CONCAT('Match-', CAST(ROW_NUMBER() OVER(ORDER BY ID) AS CHAR)) AS match_no,
        SUM(batsman_run) AS runs_scored,
        SUM(SUM(batsman_run)) OVER w AS career_runs,
        AVG(SUM(batsman_run)) OVER w AS career_avg
    FROM ipl
    WHERE batter = 'V Kohli'
    GROUP BY ID
    WINDOW w AS (ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)
) t;
```

---

## Cumulative Average

Average of all values from the start of the partition up to the current row.

```sql
SELECT student_id, test_number, score,
    AVG(score) OVER(PARTITION BY student_id ORDER BY test_number
                    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS cumulative_avg
FROM test_scores;
```

---

## Running Average

Average over a **sliding window** of N rows — smooths out fluctuations. Common in time series and financial data.

```sql
-- 3-row moving average (current + 2 preceding)
SELECT month, sales,
    AVG(sales) OVER(ORDER BY month ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS moving_avg_3
FROM monthly_sales;
```

---

## Percent of Total

Each row's value as a percentage of the overall total.

```sql
SELECT category, total_sales,
    (total_sales / SUM(total_sales) OVER()) * 100 AS percent_of_total
FROM sales_data;
```

> `SUM(...) OVER()` with an **empty OVER()** sums across all rows — the denominator is the grand total.

---

## Percent Change

Change from one period to the next — uses LAG.

```sql
SELECT month, revenue,
    LAG(revenue) OVER(ORDER BY month) AS prev_revenue,
    ROUND((revenue - LAG(revenue) OVER(ORDER BY month)) / LAG(revenue) OVER(ORDER BY month) * 100, 2) AS pct_change
FROM monthly_revenue;
```

---

## Industry Use Cases

| Use Case | Window Function |
|---|---|
| Rank employees by salary within department | `DENSE_RANK() OVER(PARTITION BY dept ORDER BY salary DESC)` |
| Running total of sales | `SUM(sales) OVER(ORDER BY date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)` |
| Month-over-month growth | `LAG(revenue) OVER(ORDER BY month)` |
| Find top N per group | `DENSE_RANK() OVER(PARTITION BY group ORDER BY value DESC)` + `WHERE rank <= N` |
| % contribution per category | `SUM(val) OVER(PARTITION BY category) / SUM(val) OVER()` |
| Smoothed moving average | `AVG(val) OVER(ORDER BY date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW)` |
| First/last purchase per customer | `FIRST_VALUE(order_date) OVER(PARTITION BY customer_id ORDER BY order_date)` |

---

## Quick Reference

```sql
-- Ranking
RANK()        OVER(PARTITION BY col ORDER BY col DESC)
DENSE_RANK()  OVER(PARTITION BY col ORDER BY col DESC)
ROW_NUMBER()  OVER(PARTITION BY col)

-- Position-based
FIRST_VALUE(col) OVER(PARTITION BY col ORDER BY col ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)
LAST_VALUE(col)  OVER(PARTITION BY col ORDER BY col ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)
NTH_VALUE(col,n) OVER(PARTITION BY col ORDER BY col ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)

-- Shift
LAG(col, n, default)  OVER(ORDER BY col)
LEAD(col, n, default) OVER(ORDER BY col)

-- Aggregates as windows
SUM(col)  OVER(ORDER BY col ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)
AVG(col)  OVER(ORDER BY col ROWS BETWEEN N PRECEDING AND CURRENT ROW)
COUNT(col) OVER(PARTITION BY col)

-- WINDOW alias
WINDOW w AS (PARTITION BY col ORDER BY col ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)
```

---

*Notes prepared from SQL Window Functions lecture — DSMP 2022-23*
