# 🗄️ SQL — Sorting, Grouping & HAVING

> A guide to ORDER BY, GROUP BY, HAVING, and SQL query execution order.

---

## 📚 Table of Contents

- [Sorting Data — ORDER BY](#sorting-data--order-by)
- [Grouping Data — GROUP BY](#grouping-data--group-by)
- [HAVING Clause](#having-clause)
- [Query Execution Order](#query-execution-order)
- [Industry Best Practices](#industry-best-practices)

---

## Sorting Data — ORDER BY

Used to arrange query results in a specific order.

```sql
SELECT * FROM table_name
ORDER BY column_name ASC | DESC;
```

- `ASC` — ascending (default)
- `DESC` — descending
- Can sort by **multiple columns** — first column takes priority, second breaks ties

```sql
ORDER BY col1 ASC, col2 DESC;
```

### LIMIT + OFFSET

Used together with `ORDER BY` to find "Nth highest/lowest" values.

```sql
SELECT * FROM table_name
ORDER BY column_name DESC
LIMIT 1 OFFSET 1;   -- skips top row, gets 2nd
```

| Clause | Purpose |
|---|---|
| `LIMIT n` | Restrict number of rows returned |
| `OFFSET n` | Skip first n rows |

> 💡 To find the **2nd largest**, sort `DESC` and use `OFFSET 1`. For **Nth largest**, use `OFFSET (N-1)`.

---

## Grouping Data — GROUP BY

Groups rows that share the same value(s) in specified column(s) so aggregate functions can be applied **per group** instead of across the whole table.

```sql
SELECT column_name, AGG_FUNC(column2)
FROM table_name
GROUP BY column_name;
```

**Rules:**
- Every column in `SELECT` that isn't inside an aggregate function **must** appear in `GROUP BY`
- Can group by **multiple columns** — creates a group for each unique combination
```sql
GROUP BY col1, col2;
```
- Commonly paired with `ORDER BY` + `LIMIT` to find top/bottom groups

**Typical use cases:**
- Per-category summaries (count, avg, max, min per brand/team/etc.)
- Comparing groups against each other (e.g., 5G vs non-5G phones)
- Multi-level grouping (e.g., brand + processor)

---

## HAVING Clause

Filters **groups** after aggregation — unlike `WHERE`, which filters rows **before** grouping.

```sql
SELECT column_name, AGG_FUNC(column2)
FROM table_name
WHERE row_condition
GROUP BY column_name
HAVING group_condition;
```

| Clause | Filters | Runs |
|---|---|---|
| `WHERE` | Individual rows | Before grouping |
| `HAVING` | Aggregated groups | After grouping |

**Rule of thumb:**
- Use `WHERE` to filter raw data (e.g., `WHERE has_5g = TRUE`)
- Use `HAVING` to filter based on aggregate results (e.g., `HAVING COUNT(*) > 10`)
- Both can appear in the same query, alongside `ORDER BY` and `LIMIT`

---

## Query Execution Order

SQL is **written** in one order but **executed** in another. This is one of the most asked SQL interview questions.

```
F → FROM
J → JOIN
W → WHERE
G → GROUP BY
H → HAVING
S → SELECT
O → ORDER BY
L → LIMIT
```

| Step | Clause | What happens |
|---|---|---|
| 1 | `FROM` | Identify source table(s) |
| 2 | `JOIN` | Combine with other tables |
| 3 | `WHERE` | Filter individual rows |
| 4 | `GROUP BY` | Group rows together |
| 5 | `HAVING` | Filter groups (post-aggregation) |
| 6 | `SELECT` | Pick columns/expressions to return |
| 7 | `ORDER BY` | Sort the result |
| 8 | `LIMIT` | Restrict number of rows returned |

**What this explains:**
- `WHERE` can't use aggregate functions — they don't exist at that stage yet
- `HAVING` can use aggregate functions — grouping has already happened
- Column aliases from `SELECT` generally can't be used in `WHERE`, but often can be used in `ORDER BY` (since `SELECT` runs before `ORDER BY`)
- `LIMIT` runs last — applied after sorting

---

## Industry Best Practices

### Filter early, group late
Always apply `WHERE` conditions to reduce row count **before** grouping — grouping fewer rows is significantly faster on large tables.

```sql
-- ✅ Good: filters rows before grouping
SELECT brand, AVG(price)
FROM smartphones
WHERE has_5g = TRUE
GROUP BY brand;

-- ❌ Avoid: grouping everything then trying to filter inside HAVING with row-level conditions
```

### Don't use HAVING for row-level filters
`HAVING` should only filter on aggregated values. Using it for simple row conditions (that `WHERE` could handle) hurts performance since it forces the engine to scan more rows before filtering.

### Index columns used in GROUP BY / ORDER BY
Especially important on large tables — an unindexed `GROUP BY` on millions of rows triggers a full table scan + filesort.

### Use EXPLAIN to verify
```sql
EXPLAIN SELECT brand, COUNT(*)
FROM smartphones
GROUP BY brand;
```
Check for `Using filesort` or `Using temporary` in the output — both signal potential performance issues on large datasets.

### Be careful with LIMIT + OFFSET on large tables
`OFFSET` becomes slow on huge tables because the database still scans and discards all skipped rows. For pagination at scale, companies often use **keyset pagination** (`WHERE id > last_seen_id LIMIT n`) instead.

---

*Notes prepared from SQL lecture — DSMP 2022-23*
