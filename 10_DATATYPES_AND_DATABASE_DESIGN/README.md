# MySQL Datatypes, Normalization & ER Diagram — Complete Reference

> A practical guide covering all MySQL data types, database normalization (1NF, 2NF, 3NF), and ER diagrams with real-world examples.

---

## Table of Contents

1. [MySQL Datatypes Overview](#1-mysql-datatypes-overview)
2. [Numeric Types](#2-numeric-types)
3. [ENUM and SET](#3-enum-and-set)
4. [BLOB](#4-blob)
5. [Spatial Datatypes](#5-spatial-datatypes)
6. [JSON](#6-json)
7. [Database Normalization](#7-database-normalization)
8. [Normal Forms](#8-normal-forms)
9. [ER Diagram](#9-er-diagram)
10. [Pro Tips](#10-pro-tips)

---

## 1. MySQL Datatypes Overview

```
MySQL Datatypes
├── Numeric       → TINYINT, SMALLINT, MEDIUMINT, INT, BIGINT, FLOAT, DOUBLE, DECIMAL
├── Text          → CHAR, VARCHAR, TEXT, MEDIUMTEXT, LONGTEXT, ENUM, SET, BLOB
├── Datetime      → DATE, TIME, DATETIME, YEAR, TIMESTAMP
└── Misc          → JSON, GEOMETRY (Spatial)
```

---

## 2. Numeric Types

### Integer Types

| Type | Storage | Min (Signed) | Max (Signed) | Max (Unsigned) | Use Case |
|------|---------|-------------|-------------|----------------|----------|
| `TINYINT` | 1 byte | -128 | 127 | 255 | Boolean, age, small flags |
| `SMALLINT` | 2 bytes | -32,768 | 32,767 | 65,535 | Quantities, item counts |
| `MEDIUMINT` | 3 bytes | -8,388,608 | 8,388,607 | 16,777,215 | Website visitors, followers |
| `INT` | 4 bytes | -2,147,483,648 | 2,147,483,647 | 4,294,967,295 | IDs, order numbers |
| `BIGINT` | 8 bytes | -2⁶³ | 2⁶³-1 | 2⁶⁴-1 | Revenue totals, YouTube views |

```sql
-- Signed vs Unsigned
-- Signed:   can be negative (default)
-- Unsigned: only positive, doubles max range

CREATE TABLE users (
    id      INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    age     TINYINT UNSIGNED,      -- 0 to 255
    balance BIGINT                  -- can be negative
);
```

### Decimal Types

| Type | Precision | Use Case |
|------|-----------|---------|
| `FLOAT` | ~7 digits | Prices, temperatures (approximate) |
| `DOUBLE` | ~15 digits | Scientific values, very large/small numbers |
| `DECIMAL(p,s)` | Exact | Financial values, bank balances |

```sql
-- DECIMAL(p, s) — p = total digits, s = decimal places
CREATE TABLE products (
    price      DECIMAL(10, 2),   -- up to 99999999.99
    tax_rate   FLOAT,            -- approximate, ok for display
    exact_cost DECIMAL(15, 4)    -- exact, for calculations
);

-- ❌ Never use FLOAT for money — rounding errors
SELECT 0.1 + 0.2;   -- 0.30000000000000004 (float error)

-- ✅ Use DECIMAL for money
SELECT CAST(0.1 AS DECIMAL(10,2)) + CAST(0.2 AS DECIMAL(10,2));  -- 0.30
```

---

## 3. ENUM and SET

### ENUM — Store One Value from a Predefined List

```sql
-- Column can only hold ONE of the listed values
CREATE TABLE users (
    gender ENUM('male', 'female', 'other'),
    status ENUM('active', 'inactive', 'banned') DEFAULT 'active'
);

INSERT INTO users VALUES ('male', 'active');    -- ✅
INSERT INTO users VALUES ('xyz', 'active');     -- ❌ error

-- Filter by ENUM value
SELECT * FROM users WHERE gender = 'female';
```

### SET — Store Multiple Values from a Predefined List

```sql
-- Column can hold ONE OR MORE of the listed values
CREATE TABLE users (
    hobbies SET('reading', 'gaming', 'cooking', 'travel', 'music')
);

INSERT INTO users VALUES ('reading,gaming');            -- ✅
INSERT INTO users VALUES ('cooking,travel,reading');    -- ✅

-- Find users who have 'gaming' as a hobby
SELECT * FROM users WHERE FIND_IN_SET('gaming', hobbies);
```

> **Key difference:** ENUM = pick exactly ONE. SET = pick ONE or MANY.

| | ENUM | SET |
|-|------|-----|
| Values allowed | Only 1 | 1 or more |
| Max options | 65,535 | 64 |
| Use case | Gender, status, category | Tags, hobbies, permissions |

---

## 4. BLOB

**Binary Large Object** — stores raw binary data like images, audio, video, files.

| Type | Max Size | Use Case |
|------|----------|---------|
| `TINYBLOB` | 255 bytes | Icons, tiny images |
| `BLOB` | 65 KB | Images, audio clips |
| `MEDIUMBLOB` | 16 MB | High-res images, longer videos |
| `LONGBLOB` | 4 GB | Full documents, large video files |

```sql
CREATE TABLE files (
    id          INT AUTO_INCREMENT PRIMARY KEY,
    filename    VARCHAR(255),
    file_data   LONGBLOB,
    uploaded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Load file into BLOB
INSERT INTO files (filename, file_data)
VALUES ('image.png', LOAD_FILE('/path/to/image.png'));
```

**Pros of storing in BLOB:**
- All data in one place — simplifies backup and restore
- Access controlled through DB user permissions
- No need to manage external file storage

**Cons of storing in BLOB:**
- Large files slow down DB performance
- Hard to share files with external apps
- Increases DB size significantly

> 💡 **Pro tip:** In production, store files on **S3 / cloud storage** and save only the **URL/path** as VARCHAR in the database. BLOB is rarely used in modern systems.

```sql
-- ✅ Better approach in production
CREATE TABLE files (
    id       INT AUTO_INCREMENT PRIMARY KEY,
    filename VARCHAR(255),
    file_url VARCHAR(500)   -- 'https://s3.amazonaws.com/bucket/file.png'
);
```

---

## 5. Spatial Datatypes

Used to store **geographic and geometric data** — coordinates, areas, routes.

```sql
-- GEOMETRY: generic type — stores points, lines, polygons
CREATE TABLE locations (
    id      INT AUTO_INCREMENT PRIMARY KEY,
    name    VARCHAR(100),
    coords  GEOMETRY,
    point   POINT    -- specific point type
);

-- Insert a point (longitude, latitude)
INSERT INTO locations (name, point)
VALUES ('Mumbai', ST_GeomFromText('POINT(72.8777 19.0760)'));

-- Spatial functions
ST_ASTEXT(geometry)     -- convert to readable text 'POINT(72.8 19.0)'
ST_X(point)             -- get X (longitude)
ST_Y(point)             -- get Y (latitude)
ST_Distance(p1, p2)     -- distance between two points
```

> Used in: maps, delivery tracking, geofencing, ride-hailing apps.

---

## 6. JSON

Stores structured JSON data directly in a column. Useful when data structure varies per row.

```sql
CREATE TABLE users (
    id      INT AUTO_INCREMENT PRIMARY KEY,
    name    VARCHAR(100),
    profile JSON
);

-- Insert JSON
INSERT INTO users (name, profile)
VALUES ('Nitish', '{"gender": "male", "city": "Delhi", "skills": ["SQL", "Python"]}');

-- Query JSON fields
SELECT profile->>'$.gender' FROM users;         -- 'male'
SELECT profile->>'$.city'   FROM users;         -- 'Delhi'
SELECT profile->'$.skills'  FROM users;         -- ["SQL","Python"]
SELECT profile->>'$.skills[0]' FROM users;      -- 'SQL'

-- Filter by JSON value
SELECT * FROM users WHERE profile->>'$.city' = 'Delhi';

-- Update JSON field
UPDATE users
SET profile = JSON_SET(profile, '$.city', 'Mumbai')
WHERE id = 1;
```

> ⚠️ Use JSON sparingly — it breaks normalization. Use it when data is truly variable per row (like product attributes, user preferences).

---

## 7. Database Normalization

### Why Not One Big Table?

Storing everything in one table causes **data redundancy**.

**Example — Single orders table:**

| order_id | date | amount | name | phone | address |
|---------|------|--------|------|-------|---------|
| 1 | 18 Mar | 300 | Nitish | - | Gurgaon |
| 2 | 15 Mar | 450 | Mukul | - | Delhi |
| 3 | 21 Mar | 700 | Nitish | - | Gurgaon |

Customer "Nitish" appears in multiple rows → **redundancy**.

**Problems (Anomalies):**

| Anomaly | Description |
|---------|-------------|
| **Insertion** | Can't add a customer without placing an order |
| **Deletion** | Deleting an order deletes the customer info too |
| **Update** | Changing address requires updating multiple rows |

**Solution → Normalization** — split into multiple tables, use JOINs.

> **Normalization** = process of organizing data to reduce redundancy and dependency by ensuring each piece of data is stored in exactly one place.

---

## 8. Normal Forms

### 1NF — First Normal Form

**A table is in 1NF if:**
- Each column contains **atomic (single, indivisible) values**
- No repeating groups or arrays in a column
- Each column has a **unique name**
- Data type of each column does not change
- Order of rows does not matter

**Violation:**

| EmpID | Name | Skills |
|-------|------|--------|
| 1 | John | Programming, Database |

Skills column has multiple values → **NOT 1NF**.

**Fixed (1NF):**

| EmpID | First Name | Last Name | Skill |
|-------|-----------|-----------|-------|
| 1 | John | Smith | Programming |
| 1 | John | Smith | Database Management |

```sql
-- ❌ Violates 1NF — multiple values in one column
CREATE TABLE employees (
    id     INT,
    name   VARCHAR(100),
    skills VARCHAR(500)   -- 'SQL, Python, Java'
);

-- ✅ 1NF compliant
CREATE TABLE employee_skills (
    emp_id INT,
    skill  VARCHAR(100),
    PRIMARY KEY (emp_id, skill)
);
```

---

### 2NF — Second Normal Form

**A table is in 2NF if:**
1. It is already in 1NF
2. No **partial dependency** exists

> **Partial dependency** = a non-key column depends on only PART of a composite primary key, not the whole key.

**Violation** (PK = OrderID + ProductID):

| OrderID | ProductID | Product Name | Quantity | Price/unit |
|---------|-----------|-------------|---------|-----------|
| 100 | P1 | Phone case | 2 | 10 |

`Product Name` depends only on `ProductID`, not on `OrderID + ProductID` → partial dependency.

**Fix — Split into 2 tables:**

```sql
-- Orders table (PK = order_id + product_id)
CREATE TABLE order_items (
    order_id   INT,
    product_id VARCHAR(10),
    quantity   INT,
    PRIMARY KEY (order_id, product_id)
);

-- Products table (no partial dependency)
CREATE TABLE products (
    product_id   VARCHAR(10) PRIMARY KEY,
    product_name VARCHAR(100),
    price        DECIMAL(10,2)
);
```

---

### 3NF — Third Normal Form

**A table is in 3NF if:**
1. It is already in 2NF
2. No **transitive dependency** exists

> **Transitive dependency** = a non-key column depends on another non-key column (not directly on the primary key).

**Violation** (PK = CustomerID):

| CustomerID | Name | City | State |
|-----------|------|------|-------|
| 001 | John | New York | New York |

`State` depends on `City`, not on `CustomerID` → transitive dependency.

**Fix — Split:**

```sql
-- Customers table
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name        VARCHAR(100),
    city_id     INT   -- foreign key
);

-- Cities table
CREATE TABLE cities (
    city_id INT PRIMARY KEY,
    city    VARCHAR(100),
    state   VARCHAR(100)
);
```

---

### Normal Forms Summary

| Normal Form | Requirement |
|-------------|-------------|
| **1NF** | Atomic values, unique column names, consistent data types |
| **2NF** | 1NF + no partial dependency (non-key depends on whole PK) |
| **3NF** | 2NF + no transitive dependency (non-key depends only on PK) |
| **BCNF** | Stricter version of 3NF |
| **4NF, 5NF** | Advanced — rare in practice |

> 💡 In real-world production databases, **3NF is the standard target**. Going beyond 3NF can sometimes hurt performance (too many JOINs).

---

## 9. ER Diagram

**Entity-Relationship Diagram** — a graphical representation of entities, their attributes, and relationships. Used in database design to plan the schema.

### Components

| Symbol | Represents |
|--------|-----------|
| Rectangle | Entity (table) |
| Ellipse | Attribute (column) |
| Diamond | Relationship |
| Line | Connection |

### Relationship Types

| Type | Symbol | Meaning | Example |
|------|--------|---------|---------|
| **One-to-One** | 1:1 | One entity ↔ exactly one other | Person ↔ Passport |
| **One-to-Many** | 1:N | One entity → many others | Customer → Orders |
| **Many-to-Many** | N:M | Many ↔ many on both sides | Students ↔ Courses |

```sql
-- 1:1 Example — Person and Passport
CREATE TABLE person (id INT PRIMARY KEY, name VARCHAR(100));
CREATE TABLE passport (
    id        INT PRIMARY KEY,
    person_id INT UNIQUE,           -- UNIQUE enforces 1:1
    number    VARCHAR(20),
    FOREIGN KEY (person_id) REFERENCES person(id)
);

-- 1:N Example — Customer and Orders
CREATE TABLE customers (id INT PRIMARY KEY, name VARCHAR(100));
CREATE TABLE orders (
    id          INT PRIMARY KEY,
    customer_id INT,                -- many orders per customer
    amount      DECIMAL(10,2),
    FOREIGN KEY (customer_id) REFERENCES customers(id)
);

-- N:M Example — Students and Courses (needs junction table)
CREATE TABLE students (id INT PRIMARY KEY, name VARCHAR(100));
CREATE TABLE courses  (id INT PRIMARY KEY, title VARCHAR(100));
CREATE TABLE enrollments (           -- junction/bridge table
    student_id INT,
    course_id  INT,
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES students(id),
    FOREIGN KEY (course_id)  REFERENCES courses(id)
);
```

> N:M relationships are always resolved with a **junction/bridge table** in the actual database.

---

## 10. Pro Tips

### Choosing the Right Numeric Type

```sql
-- ✅ Use smallest type that fits your data
age         TINYINT UNSIGNED    -- 0 to 255 (not INT)
quantity    SMALLINT            -- saves storage vs INT
user_id     INT UNSIGNED        -- standard for IDs
revenue     BIGINT              -- large numbers
price       DECIMAL(10,2)       -- always DECIMAL for money, never FLOAT
```

### ENUM vs VARCHAR

```sql
-- ✅ ENUM for fixed known values — faster, smaller storage
status ENUM('active','inactive','banned')

-- ✅ VARCHAR when values can change/grow over time
category VARCHAR(50)   -- might add new categories later
```

### Normalization vs Performance

```sql
-- Sometimes denormalization is intentional for performance
-- Analytics tables often store redundant data to avoid complex JOINs

-- Normalized (3 JOINs needed for a report)
SELECT o.id, c.name, p.name, oi.quantity
FROM orders o
JOIN customers c ON o.customer_id = c.id
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id;

-- Denormalized analytics table (fast reads, no JOINs)
CREATE TABLE order_report (
    order_id      INT,
    customer_name VARCHAR(100),
    product_name  VARCHAR(100),
    quantity      INT,
    report_date   DATE
);
```

---

## Useful Links

| Topic | Link |
|-------|------|
| MySQL Data Types | https://dev.mysql.com/doc/refman/8.0/en/data-types.html |
| JSON Functions | https://dev.mysql.com/doc/refman/8.0/en/json-functions.html |
| Spatial Types | https://dev.mysql.com/doc/refman/8.0/en/spatial-types.html |

> 🔍 **Search:** `MySQL data types` · `database normalization 1NF 2NF 3NF` · `ER diagram relationships`

---

*Reference based on MySQL 8.0 documentation + real-world usage patterns.*
