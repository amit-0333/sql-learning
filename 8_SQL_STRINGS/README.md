# MySQL String Data Types & Functions — Complete Reference

> A practical guide covering string types, wildcards, and all string functions with real-world examples used by professionals.

---

## Table of Contents

1. [String Data Types](#1-string-data-types)
2. [Wildcards](#2-wildcards)
3. [String Functions](#3-string-functions)
4. [Pro Tips & Patterns](#4-pro-tips--patterns)

---

## 1. String Data Types

| Type | Max Size | Storage | Use Case |
|------|----------|---------|----------|
| `CHAR(n)` | Fixed n chars | Always n bytes | PIN, country code, phone |
| `VARCHAR(n)` | Up to n chars | Actual + 1-2 bytes | Names, emails, addresses |
| `TEXT` | 65,535 chars | Variable | Blog posts, comments |
| `MEDIUMTEXT` | 16,777,215 chars | Variable | Articles, legal docs |
| `LONGTEXT` | 4,294,967,295 chars | Variable | Books, logs, huge content |

### CHAR — Fixed Length

```sql
-- Always reserves exactly n characters, pads shorter strings with spaces
CREATE TABLE users (country_code CHAR(2));

INSERT INTO users VALUES ('IN');  -- stored as 'IN'
INSERT INTO users VALUES ('U');   -- stored as 'U ' (padded)
```

> ✅ Best for data that is **always the same length** — ISO codes, OTPs, PIN codes, phone numbers.

### VARCHAR — Variable Length

```sql
-- Only uses space the string actually needs
CREATE TABLE users (email VARCHAR(100));

INSERT INTO users VALUES ('a@b.com');
-- Stored as 7 bytes + 1 length byte = 8 bytes total
-- CHAR(100) would always use 100 bytes
```

> ✅ Best for **varying length** data — names, emails, descriptions.

### TEXT Types

```sql
-- Don't use VARCHAR(5000) for large content — use TEXT
CREATE TABLE articles (
    title   VARCHAR(255),   -- short, indexable
    summary VARCHAR(500),   -- medium
    body    TEXT,           -- full article
    raw_log LONGTEXT        -- massive dumps/logs
);
```

> ⚠️ TEXT columns **cannot** have DEFAULT values and **cannot** be fully indexed.

### CHAR vs VARCHAR Summary

| | CHAR | VARCHAR |
|-|------|---------|
| Space used | Always fixed | Only what's needed |
| Speed | Faster (fixed position) | Slightly slower |
| Best for | Same-length values | Varying-length values |

> 💡 **Pro Rule:** Use `CHAR` when length never changes. Use `VARCHAR` for everything else.

---

## 2. Wildcards

Used with `LIKE` operator for pattern matching in WHERE clauses.

| Symbol | Meaning |
|--------|---------|
| `%` | Zero, one, or more characters |
| `_` | Exactly one character |

```sql
-- Starts with 'A', exactly 5 characters total
SELECT name FROM movies WHERE name LIKE 'A____';

-- Contains "man" anywhere
SELECT name FROM movies WHERE name LIKE '%man%';

-- Ends with "man"
SELECT name FROM movies WHERE name LIKE '%man';

-- Starts with "The"
SELECT name FROM movies WHERE name LIKE 'The%';

-- Second character is 'a'
SELECT name FROM users WHERE name LIKE '_a%';
```

### Case Sensitivity

```sql
-- LIKE is case-insensitive by default
SELECT * FROM users WHERE name LIKE 'john%';
-- Matches: John, john, JOHN

-- Use BINARY for case-sensitive match
SELECT * FROM users WHERE name LIKE BINARY 'John%';
-- Matches only: John
```

### Escape Special Characters

```sql
-- Search for literal '%' sign in data
SELECT * FROM products
WHERE description LIKE '%50\%%' ESCAPE '\';
-- Finds rows containing "50%"
```

---

## 3. String Functions

### 3.1 Case Conversion

```sql
SELECT UPPER('hello');   -- 'HELLO'
SELECT LOWER('HELLO');   -- 'hello'

-- Real use: case-insensitive login
SELECT * FROM users
WHERE LOWER(email) = LOWER('User@Email.COM');
```

---

### 3.2 Concatenation

```sql
-- CONCAT: join strings manually with separator
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM users;
-- 'John Doe'

-- CONCAT_WS: separator as first arg, skips NULLs automatically
SELECT CONCAT_WS(', ', city, state, country) AS address FROM locations;
-- If state is NULL → 'Mumbai, India'  (not 'Mumbai, , India')
```

> 💡 Always prefer `CONCAT_WS` over `CONCAT` when columns can be NULL.

---

### 3.3 Substring Extraction

```sql
-- SUBSTR(string, start_position, length)  — position starts at 1
SELECT SUBSTR('Hello World', 1, 5);    -- 'Hello'
SELECT SUBSTR('Hello World', 7);       -- 'World'
SELECT SUBSTR('Hello World', -5);      -- 'World'  (from end)
SELECT SUBSTR('Hello World', -5, 3);   -- 'Wor'

-- LEFT and RIGHT — simpler alternative
SELECT LEFT('Hello World', 5);         -- 'Hello'
SELECT RIGHT('Hello World', 5);        -- 'World'

-- Real use: extract domain from email
SELECT SUBSTR(email, LOCATE('@', email) + 1) AS domain FROM users;
-- 'user@gmail.com' → 'gmail.com'
```

---

### 3.4 Replace & Insert

```sql
-- REPLACE(string, find, replace_with)
SELECT REPLACE('Hello World', 'World', 'MySQL');   -- 'Hello MySQL'

-- Real use: clean dirty data
UPDATE laptops SET ram    = REPLACE(ram, 'GB', '');
UPDATE laptops SET weight = REPLACE(weight, 'kg', '');
UPDATE products SET price = REPLACE(price, '$', '');

-- INSERT(string, position, chars_to_remove, new_string)
SELECT INSERT('Hello World', 7, 5, 'MySQL');    -- 'Hello MySQL'
SELECT INSERT('Hello World', 7, 0, 'MySQL ');   -- 'Hello MySQL World' (insert only, no removal)
```

---

### 3.5 Reverse & Palindrome

```sql
SELECT REVERSE('Hello');   -- 'olleH'

-- Find palindromes in a table
SELECT name FROM words WHERE name = REVERSE(name);
-- Matches: 'racecar', 'level', 'madam'
```

---

### 3.6 Length

```sql
SELECT LENGTH('café');        -- 5 bytes  (é = 2 bytes in UTF-8)
SELECT CHAR_LENGTH('café');   -- 4 chars

-- Find rows with multi-byte characters (non-ASCII)
SELECT * FROM products
WHERE LENGTH(name) != CHAR_LENGTH(name);
```

---

### 3.7 Trim

```sql
SELECT TRIM('   hello   ');                     -- 'hello'
SELECT LTRIM('   hello   ');                    -- 'hello   '
SELECT RTRIM('   hello   ');                    -- '   hello'

-- Remove custom character
SELECT TRIM(BOTH '.' FROM '...hello...');       -- 'hello'
SELECT TRIM(LEADING '0' FROM '000123');         -- '123'
SELECT TRIM(TRAILING '/' FROM 'path/to/dir/');  -- 'path/to/dir'

-- Real use: clean user input before storing
UPDATE users SET username = TRIM(LOWER(username));
```

---

### 3.8 Pad

```sql
SELECT LPAD('42', 5, '0');      -- '00042'
SELECT RPAD('hello', 10, '.');  -- 'hello.....'

-- Real use: format invoice/order numbers
SELECT LPAD(invoice_id, 8, '0') AS formatted_id FROM invoices;
-- 123 → '00000123'
```

---

### 3.9 Repeat

```sql
SELECT REPEAT('ha', 3);    -- 'hahaha'
SELECT REPEAT('-', 30);    -- '------------------------------'  (divider line)
```

---

### 3.10 Locate

```sql
-- Returns position of substring (0 if not found)
SELECT LOCATE('@', 'user@email.com');      -- 5
SELECT LOCATE('o', 'Hello World', 6);     -- 8  (search starts from pos 6)

-- Real use: check if value contains a pattern
SELECT * FROM users WHERE LOCATE('@', email) > 0;
```

---

### 3.11 Substring Index

```sql
-- Split string by delimiter
SELECT SUBSTRING_INDEX('www.google.com', '.', 1);    -- 'www'
SELECT SUBSTRING_INDEX('www.google.com', '.', -1);   -- 'com'
SELECT SUBSTRING_INDEX('www.google.com', '.', 2);    -- 'www.google'

-- Real use: extract filename from path
SELECT SUBSTRING_INDEX(file_path, '/', -1) AS filename FROM uploads;
-- '/home/user/docs/file.pdf' → 'file.pdf'

-- Extract CPU brand from 'Intel Core i7 2.5GHz'
SELECT SUBSTRING_INDEX(cpu, ' ', 1) AS brand FROM laptops;
-- → 'Intel'
```

---

### 3.12 STRCMP

```sql
-- Compare two strings: returns -1, 0, or 1
SELECT STRCMP('apple', 'banana');   -- -1  (a < b)
SELECT STRCMP('banana', 'apple');   -- 1   (b > a)
SELECT STRCMP('hello', 'HELLO');    -- 0   (case-insensitive, equal)
```

---

### Quick Reference Table

| Function | Purpose | Output |
|----------|---------|--------|
| `UPPER / LOWER` | Change case | `'HELLO'` / `'hello'` |
| `CONCAT` | Join strings | `'John Doe'` |
| `CONCAT_WS` | Join with separator, NULL-safe | `'John, Doe'` |
| `SUBSTR / LEFT / RIGHT` | Extract part of string | `'Hello'` |
| `REPLACE` | Swap text | `'Hello MySQL'` |
| `INSERT` | Insert at position | `'Hello MySQL'` |
| `REVERSE` | Flip string | `'olleH'` |
| `LENGTH` | Length in bytes | `5` |
| `CHAR_LENGTH` | Length in characters | `4` |
| `TRIM / LTRIM / RTRIM` | Remove whitespace/chars | `'hello'` |
| `LPAD / RPAD` | Pad string | `'00042'` |
| `REPEAT` | Repeat string | `'hahaha'` |
| `LOCATE` | Find position | `5` |
| `SUBSTRING_INDEX` | Split by delimiter | `'gmail'` |
| `STRCMP` | Compare strings | `-1 / 0 / 1` |

---

## 4. Pro Tips & Patterns

### NULL-safe Concatenation

```sql
-- CONCAT returns NULL if ANY argument is NULL
SELECT CONCAT('Hello', NULL, 'World');   -- NULL ❌

-- CONCAT_WS skips NULLs safely
SELECT CONCAT_WS(' ', first_name, middle_name, last_name);
-- If middle_name is NULL → 'John Smith' ✅

-- Or use IFNULL
SELECT CONCAT(first_name, ' ', IFNULL(last_name, '')) FROM users;
```

### Phone Number Storage

```sql
-- ✅ Use CHAR or VARCHAR — not INT (preserves leading zeros)
CREATE TABLE contacts (
    phone_in  CHAR(10),     -- fixed Indian numbers
    phone_int VARCHAR(20)   -- international (varying length)
);
```

### Indexing String Columns

```sql
-- Full index on VARCHAR (up to 767 bytes)
CREATE INDEX idx_email ON users(email);

-- Prefix index for long strings (faster, less storage)
CREATE INDEX idx_title ON articles(title(50));

-- ❌ TEXT columns cannot be fully indexed — use prefix index
CREATE INDEX idx_body ON articles(body(100));
```

### Data Cleaning Pipeline

```sql
-- Step 1: Remove units
UPDATE laptops SET ram    = REPLACE(ram, 'GB', '');
UPDATE laptops SET weight = REPLACE(weight, 'kg', '');

-- Step 2: Standardize text
UPDATE users SET email    = LOWER(TRIM(email));
UPDATE users SET username = TRIM(username);

-- Step 3: Split combined columns
ALTER TABLE laptops ADD COLUMN gpu_brand VARCHAR(100);
UPDATE laptops SET gpu_brand = SUBSTRING_INDEX(gpu, ' ', 1);
```

### Extract Parts from Complex Strings

```sql
-- 'Intel Core i7 2.5GHz'
SELECT
  SUBSTRING_INDEX(cpu, ' ', 1) AS brand,             -- 'Intel'
  REPLACE(SUBSTRING_INDEX(cpu, ' ', -1), 'GHz', '')  -- '2.5'
    AS speed
FROM laptops;

-- Extract username from email
SELECT SUBSTRING_INDEX(email, '@', 1) AS username FROM users;
-- 'user@gmail.com' → 'user'
```

---

## Useful Links

| Topic | Link |
|-------|------|
| MySQL String Functions | https://dev.mysql.com/doc/refman/8.0/en/string-functions.html |
| MySQL Data Types | https://dev.mysql.com/doc/refman/8.0/en/data-types.html |

> 🔍 **Search:** `MySQL string functions` or `MySQL VARCHAR vs CHAR`

---

*Reference based on MySQL 8.0 documentation + real-world usage patterns.*
