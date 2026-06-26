# MySQL Temporal Data Types & Functions — Complete Reference

> A practical guide covering date/time types, functions, formatting, arithmetic, and real-world patterns used by professionals.

---

## Table of Contents

1. [Temporal Data Types](#1-temporal-data-types)
2. [DATETIME Functions](#2-datetime-functions)
3. [Date Formatting](#3-date-formatting)
4. [Type Conversion](#4-type-conversion)
5. [DATETIME Arithmetic](#5-datetime-arithmetic)
6. [TIMESTAMP vs DATETIME](#6-timestamp-vs-datetime)
7. [Pro Tips & Patterns](#7-pro-tips--patterns)

---

## 1. Temporal Data Types

| Type | Format | Range | Storage |
|------|--------|-------|---------|
| `DATE` | YYYY-MM-DD | 1000-01-01 to 9999-12-31 | 3 bytes |
| `TIME` | HH:MM:SS | -838:59:59 to 838:59:59 | 3 bytes |
| `DATETIME` | YYYY-MM-DD HH:MM:SS | 1000-01-01 to 9999-12-31 | 8 bytes |
| `TIMESTAMP` | YYYY-MM-DD HH:MM:SS | 1970-01-01 to 2038-01-19 | 4 bytes |
| `YEAR` | YYYY or YY | 1901 to 2155 | 1 byte |

```sql
CREATE TABLE events (
    id           INT AUTO_INCREMENT PRIMARY KEY,
    event_date   DATE,        -- only the date
    start_time   TIME,        -- only the time
    created_at   DATETIME,    -- full date + time, stored as-is
    updated_at   TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                              ON UPDATE CURRENT_TIMESTAMP,
    event_year   YEAR         -- only the year
);

INSERT INTO events (event_date, start_time, created_at, event_year)
VALUES ('2023-12-25', '18:30:00', NOW(), 2023);
```

### Notes on Each Type

**DATE** — When you only care about the date, not the time.
```sql
-- Birthdays, holidays, deadlines
birth_date DATE   -- '1995-08-15'
```

**TIME** — Duration or time-of-day without a date.
```sql
-- Store duration of a race, shift start time
race_time TIME    -- '01:23:45' (1 hour, 23 min, 45 sec)
```

**DATETIME** — Full timestamp stored exactly as entered, no timezone conversion.
```sql
-- Booking dates, event schedules, historical records
booking_date DATETIME   -- '2023-08-15 14:30:00'
```

**TIMESTAMP** — Auto-managed, timezone-aware, smaller storage. Used for audit fields.
```sql
-- Auto-set on insert, auto-update on change
created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
```

**YEAR** — When only the year matters.
```sql
-- Graduation year, model year
grad_year YEAR   -- 2023
```

> ⚠️ **YEAR with 2 digits** assumes 1970–2069. `70` = 1970, `69` = 2069. Always use 4-digit year to be safe.

---

## 2. DATETIME Functions

### 2.1 Current Date/Time

```sql
SELECT CURDATE();    -- '2023-08-15'           (date only)
SELECT CURTIME();    -- '14:30:25'             (time only)
SELECT NOW();        -- '2023-08-15 14:30:25'  (full datetime)
SELECT SYSDATE();    -- same as NOW()
```

---

### 2.2 Extraction Functions

```sql
-- Given datetime: '2023-08-15 14:35:22'

SELECT DATE('2023-08-15 14:35:22');      -- '2023-08-15'
SELECT TIME('2023-08-15 14:35:22');      -- '14:35:22'

SELECT YEAR('2023-08-15');               -- 2023
SELECT MONTH('2023-08-15');              -- 8
SELECT MONTHNAME('2023-08-15');          -- 'August'

SELECT DAY('2023-08-15');                -- 15
SELECT DAYOFMONTH('2023-08-15');         -- 15  (same as DAY)
SELECT DAYNAME('2023-08-15');            -- 'Tuesday'
SELECT DAYOFWEEK('2023-08-15');          -- 3   (1=Sun, 2=Mon ... 7=Sat)
SELECT DAYOFYEAR('2023-08-15');          -- 227

SELECT QUARTER('2023-08-15');            -- 3
SELECT WEEK('2023-08-15');               -- 33
SELECT WEEKOFYEAR('2023-08-15');         -- 33

SELECT HOUR('14:35:22');                 -- 14
SELECT MINUTE('14:35:22');              -- 35
SELECT SECOND('14:35:22');              -- 22

SELECT LAST_DAY('2023-02-01');          -- '2023-02-28'
SELECT LAST_DAY('2024-02-01');          -- '2024-02-29'  (leap year)
```

---

### 2.3 Extraction in Queries

```sql
-- All orders from this month
SELECT * FROM orders
WHERE MONTH(order_date) = MONTH(NOW())
AND   YEAR(order_date)  = YEAR(NOW());

-- All orders from Q3
SELECT * FROM orders WHERE QUARTER(order_date) = 3;

-- Weekend orders only
SELECT * FROM orders
WHERE DAYOFWEEK(order_date) IN (1, 7);   -- 1=Sunday, 7=Saturday

-- Monthly revenue report
SELECT
  MONTHNAME(order_date) AS month,
  COUNT(*)              AS total_orders,
  SUM(amount)           AS revenue
FROM orders
GROUP BY MONTH(order_date), MONTHNAME(order_date)
ORDER BY MONTH(order_date);
```

---

## 3. Date Formatting

```sql
DATE_FORMAT(date, 'format_string')
TIME_FORMAT(time, 'format_string')
```

### Format Specifiers Table

| Specifier | Description | Example |
|-----------|-------------|---------|
| `%Y` | Year 4-digit | `2023` |
| `%y` | Year 2-digit | `23` |
| `%M` | Month name full | `August` |
| `%b` | Month name short | `Aug` |
| `%m` | Month number (01-12) | `08` |
| `%c` | Month number (1-12) | `8` |
| `%d` | Day (01-31) | `05` |
| `%e` | Day (1-31) | `5` |
| `%D` | Day with suffix | `5th` |
| `%W` | Weekday name full | `Tuesday` |
| `%a` | Weekday name short | `Tue` |
| `%H` | Hour 24hr (00-23) | `14` |
| `%h` | Hour 12hr (01-12) | `02` |
| `%i` | Minutes (00-59) | `35` |
| `%s` | Seconds (00-59) | `22` |
| `%p` | AM or PM | `PM` |
| `%r` | Time 12hr full | `02:35:22 PM` |
| `%T` | Time 24hr full | `14:35:22` |
| `%j` | Day of year (001-366) | `227` |
| `%U` | Week number (00-53) | `33` |

> 📖 **Full specifier list:**
> https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_date-format
>
> 🔍 **Search:** `MySQL DATE_FORMAT specifiers`

### Formatting Examples

```sql
SELECT DATE_FORMAT(NOW(), '%d %b %Y');
-- '15 Aug 2023'

SELECT DATE_FORMAT(NOW(), '%W, %d %M %Y');
-- 'Tuesday, 15 August 2023'

SELECT DATE_FORMAT(NOW(), '%d/%m/%Y');
-- '15/08/2023'  (Indian format)

SELECT DATE_FORMAT(NOW(), '%Y-%m-%dT%H:%i:%sZ');
-- '2023-08-15T14:35:22Z'  (ISO 8601 — standard for APIs)

SELECT DATE_FORMAT(NOW(), '%D %M %Y');
-- '15th August 2023'

SELECT TIME_FORMAT('14:35:22', '%h:%i %p');
-- '02:35 PM'

-- You can mix custom text in the format
SELECT DATE_FORMAT(start_time, '%d hello %b, %y') FROM uber_rides;
-- '09 hello Mar, 23'
```

---

## 4. Type Conversion

### Implicit Conversion

MySQL auto-converts compatible types when it can.

```sql
-- String automatically treated as date
SELECT * FROM orders WHERE order_date = '2023-08-15';
-- MySQL converts the string to DATE automatically
```

### Explicit — STR_TO_DATE()

Convert a string in any format to a proper DATE/DATETIME.

```sql
STR_TO_DATE(string, format)
```

```sql
SELECT STR_TO_DATE('15-08-2023', '%d-%m-%Y');
-- '2023-08-15'

SELECT STR_TO_DATE('Aug 15, 2023', '%b %d, %Y');
-- '2023-08-15'

SELECT STR_TO_DATE('15/08/2023 02:30 PM', '%d/%m/%Y %h:%i %p');
-- '2023-08-15 14:30:00'

-- Real use: fix non-standard dates during data import
UPDATE orders
SET order_date = STR_TO_DATE(raw_date_col, '%d/%m/%Y')
WHERE raw_date_col IS NOT NULL;
```

### CAST and CONVERT

```sql
-- CAST
SELECT CAST('2023-08-15' AS DATE);
SELECT CAST('14:30:00' AS TIME);
SELECT CAST(NOW() AS DATE);            -- strips time part → '2023-08-15'
SELECT CAST('3.14' AS DECIMAL(10,2));  -- '3.14'

-- CONVERT (alternative syntax)
SELECT CONVERT('2023-08-15', DATE);
SELECT CONVERT(NOW(), DATE);

-- Real use: strip time from DATETIME for date-only comparison
SELECT * FROM orders
WHERE CAST(created_at AS DATE) = '2023-08-15';
```

---

## 5. DATETIME Arithmetic

### DATEDIFF and TIMEDIFF

```sql
-- DATEDIFF: returns difference in DAYS only
SELECT DATEDIFF('2023-12-31', '2023-01-01');    -- 364
SELECT DATEDIFF(NOW(), '2023-01-01');            -- days since Jan 1

-- Age in years
SELECT FLOOR(DATEDIFF(NOW(), birth_date) / 365) AS age FROM users;

-- TIMEDIFF: returns difference as TIME value
SELECT TIMEDIFF('14:30:00', '10:00:00');         -- '04:30:00'
SELECT TIMEDIFF(NOW(), login_time) AS session_duration FROM sessions;
```

---

### DATE_ADD and DATE_SUB

```sql
-- DATE_ADD(date, INTERVAL value unit)
SELECT DATE_ADD('2023-01-01', INTERVAL 30 DAY);     -- '2023-01-31'
SELECT DATE_ADD('2023-01-01', INTERVAL 3 MONTH);    -- '2023-04-01'
SELECT DATE_ADD('2023-01-01', INTERVAL 1 YEAR);     -- '2024-01-01'
SELECT DATE_ADD(NOW(), INTERVAL 2 HOUR);            -- 2 hours from now

-- DATE_SUB(date, INTERVAL value unit)
SELECT DATE_SUB(NOW(), INTERVAL 7 DAY);     -- 1 week ago
SELECT DATE_SUB(NOW(), INTERVAL 1 MONTH);   -- 1 month ago
SELECT DATE_SUB(NOW(), INTERVAL 1 YEAR);    -- 1 year ago
```

**Available INTERVAL units:**
`MICROSECOND` · `SECOND` · `MINUTE` · `HOUR` · `DAY` · `WEEK` · `MONTH` · `QUARTER` · `YEAR`

---

### ADDTIME and SUBTIME

```sql
-- Add/subtract time from a datetime
SELECT ADDTIME('2023-08-15 10:00:00', '02:30:00');
-- '2023-08-15 12:30:00'

SELECT SUBTIME('2023-08-15 10:00:00', '01:00:00');
-- '2023-08-15 09:00:00'
```

---

### Arithmetic in Queries

```sql
-- Orders from last 7 days
SELECT * FROM orders
WHERE order_date >= DATE_SUB(NOW(), INTERVAL 7 DAY);

-- Subscription expiry date
SELECT
  user_id,
  start_date,
  DATE_ADD(start_date, INTERVAL 1 YEAR) AS expiry_date
FROM subscriptions;

-- Upcoming events in next 30 days
SELECT * FROM events
WHERE event_date BETWEEN NOW()
AND DATE_ADD(NOW(), INTERVAL 30 DAY);

-- Orders between two dates
SELECT * FROM orders
WHERE order_date BETWEEN '2023-01-01' AND '2023-03-31';
```

---

## 6. TIMESTAMP vs DATETIME

| Feature | DATETIME | TIMESTAMP |
|---------|----------|-----------|
| Range | 1000-01-01 to 9999-12-31 | 1970-01-01 to 2038-01-19 |
| Storage | 8 bytes | 4 bytes |
| Timezone | Stored as-is | Converted to UTC on store, back on read |
| Precision | Up to microseconds | Up to seconds |
| Auto-update | ❌ No | ✅ Yes (`ON UPDATE CURRENT_TIMESTAMP`) |
| NULL default | Yes | No (defaults to current time) |

```sql
-- TIMESTAMP: perfect for audit columns (auto-managed)
CREATE TABLE posts (
    id         INT AUTO_INCREMENT PRIMARY KEY,
    title      VARCHAR(255),
    content    TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                         ON UPDATE CURRENT_TIMESTAMP
    -- created_at: set once on insert, never changes
    -- updated_at: auto-updates every time row is modified
);

-- DATETIME: for business/historical dates beyond TIMESTAMP range
CREATE TABLE bookings (
    id           INT AUTO_INCREMENT PRIMARY KEY,
    booking_date DATETIME,   -- timezone-independent, wide range
    event_date   DATE        -- just the date, no time needed
);
```

> 💡 **Pro Rule:**
> - Use `TIMESTAMP` → `created_at`, `updated_at` (auto-managed audit fields)
> - Use `DATETIME` → `booking_date`, `event_date`, `birth_date` (business dates, timezone-independent)

---

## 7. Pro Tips & Patterns

### Indexing Date Columns

```sql
-- ✅ Index on date column for fast range queries
CREATE INDEX idx_order_date ON orders(order_date);

-- ❌ Avoid functions on indexed columns in WHERE — kills index usage
SELECT * FROM orders WHERE YEAR(order_date) = 2023;

-- ✅ Use range instead — index is used
SELECT * FROM orders
WHERE order_date BETWEEN '2023-01-01' AND '2023-12-31';
```

### Standard Date Filtering Patterns

```sql
-- Today
SELECT * FROM orders WHERE DATE(created_at) = CURDATE();

-- This week
SELECT * FROM orders
WHERE WEEK(created_at)  = WEEK(NOW())
AND   YEAR(created_at)  = YEAR(NOW());

-- This month
SELECT * FROM orders
WHERE MONTH(created_at) = MONTH(NOW())
AND   YEAR(created_at)  = YEAR(NOW());

-- Last 30 days
SELECT * FROM orders
WHERE created_at >= DATE_SUB(NOW(), INTERVAL 30 DAY);

-- Last 7 days
SELECT * FROM orders
WHERE created_at >= DATE_SUB(NOW(), INTERVAL 7 DAY);

-- Specific date range
SELECT * FROM orders
WHERE DATE(created_at) BETWEEN '2023-01-01' AND '2023-03-31';
```

### Audit Columns (Standard in Every Production Table)

```sql
-- Every production table should have these
CREATE TABLE any_table (
    id         BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    -- ... your columns ...
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    created_by INT,
    is_deleted TINYINT(1) DEFAULT 0   -- soft delete flag
);
```

### Format Dates for Different Outputs

```sql
-- ISO 8601 — standard for REST APIs / JSON
SELECT DATE_FORMAT(created_at, '%Y-%m-%dT%H:%i:%sZ') FROM orders;
-- '2023-08-15T14:35:22Z'

-- Human-readable for reports
SELECT DATE_FORMAT(created_at, '%D %M %Y') FROM orders;
-- '15th August 2023'

-- Indian format
SELECT DATE_FORMAT(created_at, '%d/%m/%Y') FROM orders;
-- '15/08/2023'

-- 12-hour time with AM/PM
SELECT TIME_FORMAT(start_time, '%h:%i %p') FROM events;
-- '02:35 PM'
```

### Age & Duration Calculations

```sql
-- User age
SELECT
  name,
  birth_date,
  FLOOR(DATEDIFF(NOW(), birth_date) / 365) AS age
FROM users;

-- Days until expiry
SELECT
  plan_name,
  expiry_date,
  DATEDIFF(expiry_date, NOW()) AS days_left
FROM subscriptions
WHERE expiry_date > NOW();

-- Session duration
SELECT
  user_id,
  login_time,
  logout_time,
  TIMEDIFF(logout_time, login_time) AS duration
FROM sessions;
```

### Fix Non-Standard Date Formats (Data Import)

```sql
-- When imported CSV has dates like '15-08-2023' or 'Aug 15, 2023'
ALTER TABLE orders ADD COLUMN order_date_clean DATE;

UPDATE orders
SET order_date_clean = STR_TO_DATE(order_date_raw, '%d-%m-%Y');

-- Then drop old column and rename
ALTER TABLE orders DROP COLUMN order_date_raw;
ALTER TABLE orders RENAME COLUMN order_date_clean TO order_date;
```

---

## Useful Links

| Topic | Link |
|-------|------|
| DATE_FORMAT specifiers | https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_date-format |
| Date & Time Functions | https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html |
| Temporal Data Types | https://dev.mysql.com/doc/refman/8.0/en/date-and-time-types.html |

> 🔍 **Search tips:**
> - `MySQL DATE_FORMAT specifiers` → full format codes table
> - `MySQL temporal data types` → type details and ranges
> - `MySQL date functions` → all date/time functions list

---

*Reference based on MySQL 8.0 documentation + real-world usage patterns.*
