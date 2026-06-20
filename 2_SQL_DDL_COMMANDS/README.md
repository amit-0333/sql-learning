# 🗄️ SQL DDL — Complete Guide

> A comprehensive guide to SQL Data Definition Language (DDL) covering fundamentals to real-world industry practices.

---

## 📚 Table of Contents

- [What is SQL?](#what-is-sql)
- [Types of SQL Commands](#types-of-sql-commands)
- [DDL Commands for Databases](#ddl-commands-for-databases)
- [DDL Commands for Tables](#ddl-commands-for-tables)
- [Data Integrity](#data-integrity)
- [Constraints in MySQL](#constraints-in-mysql)
- [ALTER TABLE](#alter-table)
- [TRUNCATE vs DELETE](#truncate-vs-delete)
- [Industry Best Practices](#industry-best-practices)

---

## What is SQL?

**SQL (Structured Query Language)** is a standard language used to communicate with relational databases. It is used to create, read, update, and delete data stored in tables.

---

## Types of SQL Commands

SQL commands are grouped into 4 categories:

### DDL — Data Definition Language
Defines and manages the **structure** of database objects.

| Command | Description |
|---|---|
| `CREATE` | Create new database objects (DB, table, index) |
| `ALTER` | Modify existing database objects |
| `DROP` | Permanently delete database objects |
| `TRUNCATE` | Remove all rows from a table (keeps structure) |

### DML — Data Manipulation Language
Manipulates the **data** inside tables.

| Command | Description |
|---|---|
| `INSERT` | Add new rows |
| `UPDATE` | Modify existing rows |
| `DELETE` | Remove specific rows |
| `SELECT` | Retrieve/query data |

### DCL — Data Control Language
Controls **access and permissions**.

| Command | Description |
|---|---|
| `GRANT` | Give access rights to a user |
| `REVOKE` | Remove access rights from a user |

### TCL — Transaction Control Language
Manages **transactions** (groups of operations).

| Command | Description |
|---|---|
| `COMMIT` | Save all changes permanently |
| `ROLLBACK` | Undo changes not yet committed |
| `SAVEPOINT` | Set a point to rollback to |

---

## DDL Commands for Databases

```sql
-- Create a database
CREATE DATABASE campusx;

-- Create only if it doesn't exist (safe approach)
CREATE DATABASE IF NOT EXISTS campusx;

-- Use a database
USE campusx;

-- Drop a database
DROP DATABASE campusx;

-- Alter database character set and collation
ALTER DATABASE campusx CHARACTER SET utf8mb4;
ALTER DATABASE campusx COLLATE utf8mb4_unicode_ci;

-- Show all databases
SHOW DATABASES;
```

> ⚠️ **Note:** In MySQL, `ALTER DATABASE` only supports changing character set and collation. To rename a database, you must dump and restore it.

---

## DDL Commands for Tables

```sql
-- Create a table
CREATE TABLE users (
  user_id   INTEGER NOT NULL,
  name      VARCHAR(255) NOT NULL,
  email     VARCHAR(255) NOT NULL,
  password  VARCHAR(255) NOT NULL
);

-- Create table if not exists
CREATE TABLE IF NOT EXISTS users (...);

-- Show all tables in current DB
SHOW TABLES;

-- View table structure
DESCRIBE users;

-- Drop a table
DROP TABLE users;

-- Drop only if it exists
DROP TABLE IF EXISTS users;
```

---

## Data Integrity

**Data Integrity** refers to the accuracy, completeness, and consistency of data stored in a database. It ensures data is reliable and protected from errors, corruption, or unauthorized changes.

### Methods to Ensure Data Integrity

**Constraints** — Rules that data must satisfy before being inserted, updated, or deleted. They prevent inconsistent or corrupted data.

**Transactions** — A sequence of operations treated as a single unit of work. Either all operations succeed or none do (ACID properties).

**Normalization** — A design technique that organizes tables to minimize redundancy and ensure data consistency.

---

## Constraints in MySQL

Constraints are rules enforced on columns to maintain data integrity. They can be defined **inline** (on the column) or as **named constraints** using the `CONSTRAINT` keyword.

### 1. NOT NULL
Ensures a column cannot have a NULL (empty) value.

```sql
CREATE TABLE users (
  name VARCHAR(255) NOT NULL
);
```

### 2. UNIQUE
Ensures all values in a column are different. Unlike PRIMARY KEY, a UNIQUE column can have NULL.

```sql
-- Inline
email VARCHAR(255) NOT NULL UNIQUE,

-- Named constraint (preferred in production)
CONSTRAINT users_email_unique UNIQUE (email)

-- Composite unique (combination must be unique)
CONSTRAINT uc_name_email UNIQUE (name, email)
```

### 3. PRIMARY KEY
Combination of NOT NULL + UNIQUE. Uniquely identifies each row. A table can have only **one** primary key.

```sql
-- Inline
user_id INTEGER PRIMARY KEY,

-- Named constraint
CONSTRAINT users_pk PRIMARY KEY (user_id)

-- Composite primary key
CONSTRAINT pk PRIMARY KEY (col1, col2)
```

### 4. AUTO_INCREMENT
Automatically generates a unique incrementing number. Always used with PRIMARY KEY.

```sql
student_id INTEGER PRIMARY KEY AUTO_INCREMENT
```

> 💡 When a row is deleted, AUTO_INCREMENT does **not** reuse the deleted ID.

### 5. CHECK
Validates data against a custom condition before inserting/updating.

```sql
-- Inline
age INTEGER CHECK (age > 6 AND age < 25),

-- Named constraint
CONSTRAINT students_age_check CHECK (age > 6 AND age < 25)
```

### 6. DEFAULT
Sets a default value for a column if no value is provided.

```sql
-- Static default
status VARCHAR(50) DEFAULT 'active',

-- Dynamic default (current timestamp)
created_at DATETIME DEFAULT CURRENT_TIMESTAMP
```

### 7. FOREIGN KEY
Links a column in one table to the PRIMARY KEY of another table. Enforces **referential integrity**.

```sql
CREATE TABLE orders (
  order_id   INTEGER PRIMARY KEY AUTO_INCREMENT,
  cid        INTEGER NOT NULL,
  order_date DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,

  CONSTRAINT orders_fk FOREIGN KEY (cid) REFERENCES customers(cid)
);
```

#### Referential Actions
What happens to child rows when the parent row is updated/deleted:

| Action | Behavior |
|---|---|
| `RESTRICT` | Blocks the delete/update of parent if child rows exist (default) |
| `CASCADE` | Automatically deletes/updates child rows |
| `SET NULL` | Sets the FK column to NULL |
| `SET DEFAULT` | Sets the FK column to its default value |

```sql
-- CASCADE example
CONSTRAINT orders_fk FOREIGN KEY (cid) REFERENCES customers(cid)
ON DELETE CASCADE
ON UPDATE CASCADE

-- SET NULL example
CONSTRAINT orders_fk FOREIGN KEY (cid) REFERENCES customers(cid)
ON DELETE SET NULL
```

### Full Example
```sql
CREATE TABLE users (
  user_id   INTEGER NOT NULL,
  name      VARCHAR(255) NOT NULL,
  email     VARCHAR(255) NOT NULL,
  password  VARCHAR(255) NOT NULL,

  CONSTRAINT users_pk          PRIMARY KEY (user_id),
  CONSTRAINT users_email_unique UNIQUE (email)
);

CREATE TABLE students (
  student_id INTEGER PRIMARY KEY AUTO_INCREMENT,
  name       VARCHAR(50) NOT NULL,
  age        INTEGER,

  CONSTRAINT students_age_check CHECK (age > 6 AND age < 25)
);

CREATE TABLE orders (
  order_id   INTEGER PRIMARY KEY AUTO_INCREMENT,
  cid        INTEGER NOT NULL,
  order_date DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,

  CONSTRAINT orders_fk FOREIGN KEY (cid) REFERENCES customers(cid)
  ON DELETE CASCADE
);
```

---

## ALTER TABLE

The `ALTER TABLE` statement modifies the **structure** of an existing table without losing data.

### Add a Column
```sql
ALTER TABLE customers ADD COLUMN password VARCHAR(255) NOT NULL;

-- Add at a specific position
ALTER TABLE customers ADD COLUMN surname VARCHAR(255) NOT NULL AFTER name;

-- Add at the beginning
ALTER TABLE customers ADD COLUMN code INT FIRST;
```

### Add Multiple Columns
```sql
ALTER TABLE customers
  ADD COLUMN pan_number  VARCHAR(255) AFTER surname,
  ADD COLUMN joining_date DATETIME;
```

### Drop a Column
```sql
ALTER TABLE customers DROP COLUMN pan_number;
```

### Modify a Column
```sql
-- Change data type or constraints
ALTER TABLE customers MODIFY COLUMN age TINYINT NOT NULL;

-- Rename and change type (MySQL 8+)
ALTER TABLE customers RENAME COLUMN surname TO last_name;
```

### Add / Drop Constraints
```sql
-- Add a CHECK constraint
ALTER TABLE customers ADD CONSTRAINT customer_age_check CHECK (age > 13);

-- Add a UNIQUE constraint
ALTER TABLE customers ADD CONSTRAINT unique_email UNIQUE (email);

-- Drop a constraint
ALTER TABLE customers DROP CONSTRAINT customer_age_check;

-- Drop a foreign key
ALTER TABLE orders DROP FOREIGN KEY orders_fk;

-- Drop primary key
ALTER TABLE users DROP PRIMARY KEY;
```

### Rename a Table
```sql
ALTER TABLE old_name RENAME TO new_name;
```

---

## TRUNCATE vs DELETE

```sql
-- Truncate (fast, removes all rows)
TRUNCATE TABLE students;

-- Delete all rows (slow, logged)
DELETE FROM students;

-- Delete specific rows
DELETE FROM students WHERE age < 10;
```

| Feature | TRUNCATE | DELETE |
|---|---|---|
| Removes all rows | ✅ | ✅ (without WHERE) |
| WHERE clause | ❌ | ✅ |
| Rollback possible | ❌ (in MySQL) | ✅ |
| Resets AUTO_INCREMENT | ✅ | ❌ |
| Speed | Faster | Slower |
| Triggers fired | ❌ | ✅ |
| DDL or DML | DDL | DML |

---

## Industry Best Practices

These are practices followed by experienced developers and companies in production environments.

### Naming Conventions
```sql
-- Use snake_case for all names
first_name, order_date, customer_id

-- Always name your constraints (easier to drop/modify later)
CONSTRAINT users_pk PRIMARY KEY (user_id)
CONSTRAINT orders_fk FOREIGN KEY (cid) REFERENCES customers(cid)

-- Table names: plural, lowercase
users, orders, products

-- Index names: idx_table_column
idx_users_email
```

### Always Use IF NOT EXISTS / IF EXISTS
```sql
-- Prevents errors in scripts/migrations
CREATE DATABASE IF NOT EXISTS myapp;
CREATE TABLE IF NOT EXISTS users (...);
DROP TABLE IF EXISTS temp_data;
```

### Character Set & Collation
```sql
-- Use utf8mb4 for full Unicode support (supports emojis)
CREATE DATABASE myapp CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```
> Companies use `utf8mb4` instead of `utf8` because MySQL's `utf8` does not support 4-byte characters like emojis.

### Soft Delete (Industry Standard)
Instead of deleting rows, companies mark them as deleted:
```sql
ALTER TABLE users ADD COLUMN is_deleted BOOLEAN DEFAULT FALSE;
ALTER TABLE users ADD COLUMN deleted_at DATETIME DEFAULT NULL;

-- "Delete" a user
UPDATE users SET is_deleted = TRUE, deleted_at = NOW() WHERE user_id = 5;

-- Query active users only
SELECT * FROM users WHERE is_deleted = FALSE;
```

### Audit Columns
Almost every production table has these columns:
```sql
created_at  DATETIME DEFAULT CURRENT_TIMESTAMP,
updated_at  DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
created_by  INTEGER,
updated_by  INTEGER
```

### Migrations (Version Control for DB)
In companies, database changes are tracked using **migration files** — never run raw ALTER on production directly. Tools used:
- **Flyway**
- **Liquibase**
- **Django migrations** (Python)
- **Knex.js** (Node.js)

### Indexing Foreign Keys
```sql
-- Always index FK columns for faster JOINs
CREATE INDEX idx_orders_cid ON orders(cid);
```

### Avoid DROP in Production
```sql
-- Never run this on production without backup
DROP TABLE users;  -- ❌ Dangerous

-- Always take a backup first, or rename
ALTER TABLE users RENAME TO users_backup_20240101;  -- ✅ Safer
```

### Use Transactions for Schema Changes
```sql
START TRANSACTION;
  ALTER TABLE users ADD COLUMN phone VARCHAR(15);
  ALTER TABLE users ADD CONSTRAINT chk_phone CHECK (LENGTH(phone) >= 10);
COMMIT;
-- ROLLBACK if something goes wrong
```

---

## 🛠️ Tools Used

- **MySQL** with **phpMyAdmin** (local development via XAMPP)
- **MySQL Workbench** (used in companies for visual schema design)
- **DBeaver** (popular open-source DB client in industry)

---

## 📝 Quick Reference

```sql
-- Database
CREATE DATABASE IF NOT EXISTS db_name;
DROP DATABASE IF EXISTS db_name;
ALTER DATABASE db_name CHARACTER SET utf8mb4;

-- Table
CREATE TABLE table_name (...);
DROP TABLE IF EXISTS table_name;
TRUNCATE TABLE table_name;

-- Column operations
ALTER TABLE t ADD COLUMN col datatype constraints;
ALTER TABLE t DROP COLUMN col;
ALTER TABLE t MODIFY COLUMN col new_datatype;
ALTER TABLE t RENAME COLUMN old TO new;
ALTER TABLE t RENAME TO new_table_name;

-- Constraints
ALTER TABLE t ADD CONSTRAINT name TYPE (col);
ALTER TABLE t DROP CONSTRAINT name;
```

---

*Notes prepared from SQL DDL lecture — DSMP 2022-23*
