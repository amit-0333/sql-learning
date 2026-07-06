# MySQL Stored Procedures & Transactions — Complete Reference

> A practical guide covering stored procedures, transactions, savepoints, autocommit, and ACID properties with real-world examples.

---

## Table of Contents

1. [Stored Procedures](#1-stored-procedures)
2. [Transactions](#2-transactions)
3. [Autocommit](#3-autocommit)
4. [ACID Properties](#4-acid-properties)
5. [Pro Tips & Patterns](#5-pro-tips--patterns)

---

## 1. Stored Procedures

A stored procedure is a **named block of SQL statements and procedural logic** stored in a database, executed by a user or application using `CALL`.

Used to encapsulate business logic — data validation, data processing, database updates. Separates application logic from the DB layer.

### Benefits

| Benefit | Description |
|---------|-------------|
| **Performance** | Precompiled and optimized, reduces network traffic |
| **Security** | Granted specific permissions, limits access to sensitive data |
| **Encapsulation** | Complex business logic in one place, easy to maintain |
| **Consistency** | DB operations always performed the same way |
| **Reduced traffic** | Logic runs on DB server, less data sent over network |

---

### Syntax

```sql
DELIMITER $$

CREATE PROCEDURE procedure_name(
    IN  input_param  DATATYPE,   -- input only
    OUT output_param DATATYPE,   -- output only
    INOUT both_param DATATYPE    -- input and output
)
BEGIN
    -- SQL statements
    -- procedural logic (IF, DECLARE, SET, etc.)
END $$

DELIMITER ;

-- Call a procedure
CALL procedure_name(args);
```

> **UDF vs Stored Procedure:**
> - UDF → uses `SELECT`, returns a value, used inside queries
> - Procedure → uses `CALL`, can use OUT params, no return value required

---

### Parameters

| Mode | Description |
|------|-------------|
| `IN` | Input only — value passed into procedure |
| `OUT` | Output only — value returned from procedure |
| `INOUT` | Both — passed in and returned modified |

---

### Example 1 — Hello World (no parameters)

```sql
DELIMITER $$
CREATE PROCEDURE hello_world()
BEGIN
    SELECT 'Hello World';
END $$
DELIMITER ;

CALL hello_world();
-- Output: 'Hello World'
```

---

### Example 2 — Add New User with Error Message

```sql
DELIMITER $$
CREATE PROCEDURE add_user(
    IN  input_name  VARCHAR(255),
    IN  input_email VARCHAR(255),
    OUT message     VARCHAR(255)
)
BEGIN
    DECLARE user_count INTEGER;

    -- Check if email already exists
    SELECT COUNT(*) INTO user_count
    FROM users WHERE email = input_email;

    IF user_count = 0 THEN
        INSERT INTO users (name, email)
        VALUES (input_name, input_email);
        SET message = 'User inserted';
    ELSE
        SET message = 'Email already exists';
    END IF;
END $$
DELIMITER ;

-- Call the procedure
SET @message = '';
CALL add_user('Ankit', 'ankit1234@gmail.com', @message);
SELECT @message;   -- 'User inserted' or 'Email already exists'
```

---

### Example 3 — Show Orders of a Single User

```sql
DELIMITER $$
CREATE PROCEDURE user_orders(IN input_user_id INT)
BEGIN
    SELECT * FROM orders
    WHERE user_id = input_user_id;
END $$
DELIMITER ;

CALL user_orders(1);
```

---

### Example 4 — Place an Order (Real World Complex)

```sql
DELIMITER $$
CREATE PROCEDURE place_order(
    IN  input_user_id INT,
    IN  input_r_id    INT,
    IN  input_f_ids   VARCHAR(255),  -- comma-separated food IDs e.g. '3,4'
    OUT message       VARCHAR(255)
)
BEGIN
    DECLARE new_order_id INTEGER;
    DECLARE f_id1        INTEGER;
    DECLARE f_id2        INTEGER;
    DECLARE total_amount DECIMAL(10,2);

    -- Extract food IDs from string
    SET f_id1 = SUBSTRING_INDEX(input_f_ids, ',', 1);
    SET f_id2 = SUBSTRING_INDEX(input_f_ids, ',', -1);

    -- Generate new order ID
    SELECT MAX(order_id) + 1 INTO new_order_id FROM orders;

    -- Calculate total price
    SELECT SUM(price) INTO total_amount
    FROM menu
    WHERE r_id = input_r_id
    AND f_id IN (f_id1, f_id2);

    -- Insert into orders table
    INSERT INTO orders (order_id, user_id, r_id, amount, date)
    VALUES (new_order_id, input_user_id, input_r_id,
            total_amount, DATE(NOW()));

    -- Insert into order_details table
    INSERT INTO order_details (order_id, f_id)
    VALUES (new_order_id, f_id1),
           (new_order_id, f_id2);

    SET message = 'Order placed successfully';
END $$
DELIMITER ;

-- Call
SET @msg = '';
CALL place_order(1, 2, '3,4', @msg);
SELECT @msg;
```

---

### Manage Procedures

```sql
-- Show all procedures in a database
SHOW PROCEDURE STATUS WHERE db = 'your_database';

-- View procedure code
SHOW CREATE PROCEDURE add_user;

-- Drop a procedure
DROP PROCEDURE IF EXISTS add_user;
```

---

### Procedural Logic Inside Procedures

```sql
-- DECLARE variables
DECLARE var_name DATATYPE DEFAULT value;

-- SET variable
SET var_name = value;

-- Store query result into variable
SELECT COUNT(*) INTO var_name FROM table;

-- IF / ELSEIF / ELSE
IF condition THEN
    -- statements
ELSEIF condition THEN
    -- statements
ELSE
    -- statements
END IF;

-- WHILE loop
WHILE condition DO
    -- statements
END WHILE;

-- CASE
CASE variable
    WHEN value1 THEN statement1;
    WHEN value2 THEN statement2;
    ELSE statement3;
END CASE;
```

---

## 2. Transactions

A **database transaction** is a sequence of SQL operations treated as a **single logical unit of work**. Follows the principle of **all or none** — if any step fails, all steps are rolled back.

---

### Transaction Commands

| Command | Purpose |
|---------|---------|
| `START TRANSACTION` | Begin a new transaction |
| `COMMIT` | Permanently save all changes |
| `ROLLBACK` | Undo all changes since transaction started |
| `SAVEPOINT name` | Mark a checkpoint within a transaction |
| `ROLLBACK TO name` | Undo only to that savepoint |
| `RELEASE SAVEPOINT name` | Remove a savepoint |

---

### Example 1 — Basic Transaction

```sql
START TRANSACTION;

UPDATE person SET balance = 40000 WHERE id = 1;
UPDATE person SET balance = 25000 WHERE id = 4;

COMMIT;   -- both updates saved permanently
```

---

### Example 2 — Rollback

```sql
START TRANSACTION;

UPDATE person SET balance = 30000 WHERE id = 1;
UPDATE person SET balance = 35000 WHERE id = 4;

ROLLBACK;  -- both updates undone, back to original values
```

---

### Example 3 — Commit + Rollback Together

```sql
START TRANSACTION;

UPDATE person SET balance = 30000 WHERE id = 1;
COMMIT;   -- ✅ this update is saved permanently

UPDATE person SET balance = 35000 WHERE id = 4;
ROLLBACK; -- ❌ only this update is undone (after the commit)

-- Result: id=1 has 30000 (saved), id=4 unchanged (rolled back)
```

---

### Example 4 — Savepoint (Partial Rollback)

```sql
START TRANSACTION;

SAVEPOINT A;
UPDATE person SET balance = 30000 WHERE id = 1;

SAVEPOINT B;
UPDATE person SET balance = 35000 WHERE id = 4;

ROLLBACK TO B;
-- Only undoes changes AFTER savepoint B
-- Changes after savepoint A are still active

COMMIT;  -- saves the first update only
```

---

## 3. Autocommit

By default MySQL **autocommits** every SQL statement immediately — each statement is its own transaction.

```sql
-- Check current autocommit status
SELECT @@autocommit;   -- 1 = on, 0 = off

-- Disable autocommit
SET autocommit = 0;

-- Now nothing is saved until you COMMIT manually
INSERT INTO person (name) VALUES ('rishabh');
SELECT * FROM person;   -- you can see it in this session

-- Another session won't see it yet — not committed
COMMIT;   -- now permanently saved

-- Re-enable autocommit
SET autocommit = 1;
```

> When you use `START TRANSACTION`, autocommit is automatically disabled for that transaction only.

---

## 4. ACID Properties

ACID is a set of properties that ensure **reliable and consistent** database transactions.

| Property | Full Name | Meaning |
|----------|-----------|---------|
| **A** | Atomicity | All or nothing — entire transaction succeeds or fully rolls back |
| **C** | Consistency | DB moves from one valid state to another, constraints always maintained |
| **I** | Isolation | Concurrent transactions don't interfere with each other |
| **D** | Durability | Once committed, changes are permanent even after system crash |

### Atomicity
```sql
-- Bank transfer example
START TRANSACTION;
  UPDATE accounts SET balance = balance - 5000 WHERE name = 'Nitish';
  UPDATE accounts SET balance = balance + 5000 WHERE name = 'Ankit';
COMMIT;

-- If system crashes after step 1 → ROLLBACK → neither update happens
-- Both happen or neither happens = ATOMICITY
```

### Consistency
```sql
-- Constraints ensure consistency
ALTER TABLE accounts ADD CONSTRAINT chk_balance CHECK (balance >= 0);

-- This will fail (rollback) if balance goes negative
-- DB stays in a valid state always
```

### Isolation
```sql
-- Session 1 starts a transaction but hasn't committed
START TRANSACTION;
UPDATE person SET balance = 99999 WHERE id = 1;

-- Session 2 reads the table → still sees old value (isolation)
SELECT balance FROM person WHERE id = 1;  -- old value, not 99999
```

### Durability
```sql
-- Once committed, it's permanent
START TRANSACTION;
UPDATE accounts SET balance = 50000 WHERE id = 1;
COMMIT;

-- Even if server restarts now, this change is saved on disk
```

> ACID is critical for **banking, finance, healthcare, e-commerce** systems where data integrity is non-negotiable.

---

## 5. Pro Tips & Patterns

### Always Use Transactions for Multi-Step Operations

```sql
-- ❌ Without transaction — partial failure leaves DB in bad state
UPDATE accounts SET balance = balance - 5000 WHERE id = 1;
-- (crash here — money gone but not received)
UPDATE accounts SET balance = balance + 5000 WHERE id = 2;

-- ✅ With transaction — safe
START TRANSACTION;
UPDATE accounts SET balance = balance - 5000 WHERE id = 1;
UPDATE accounts SET balance = balance + 5000 WHERE id = 2;
COMMIT;
```

### Use Transactions Inside Stored Procedures

```sql
DELIMITER $$
CREATE PROCEDURE transfer_money(
    IN from_id INT,
    IN to_id   INT,
    IN amount  DECIMAL(10,2),
    OUT message VARCHAR(100)
)
BEGIN
    DECLARE sender_balance DECIMAL(10,2);

    START TRANSACTION;

    SELECT balance INTO sender_balance
    FROM accounts WHERE id = from_id;

    IF sender_balance >= amount THEN
        UPDATE accounts SET balance = balance - amount WHERE id = from_id;
        UPDATE accounts SET balance = balance + amount WHERE id = to_id;
        COMMIT;
        SET message = 'Transfer successful';
    ELSE
        ROLLBACK;
        SET message = 'Insufficient balance';
    END IF;
END $$
DELIMITER ;
```

### Use Savepoints for Complex Multi-Step Transactions

```sql
-- Order placement with partial rollback capability
START TRANSACTION;

SAVEPOINT before_order;
INSERT INTO orders (...) VALUES (...);

SAVEPOINT before_details;
INSERT INTO order_details (...) VALUES (...);

-- If order_details fails, rollback only that part
ROLLBACK TO before_details;

-- Order insert is still active, retry details insert
INSERT INTO order_details (...) VALUES (...);

COMMIT;
```

### Procedure vs Function — When to Use What

| Situation | Use |
|-----------|-----|
| Need to return a single value | Function (UDF) |
| Need to run multiple statements | Stored Procedure |
| Need to use inside SELECT | Function |
| Need OUT parameters | Stored Procedure |
| Need transactions inside | Stored Procedure |
| Reusable calculation | Function |

---

## Useful Links

| Topic | Link |
|-------|------|
| Stored Procedures | https://dev.mysql.com/doc/refman/8.0/en/stored-routines.html |
| Transactions | https://dev.mysql.com/doc/refman/8.0/en/commit.html |
| ACID Properties | https://dev.mysql.com/doc/refman/8.0/en/mysql-acid.html |

> 🔍 **Search:** `MySQL stored procedure syntax` · `MySQL transaction ACID` · `MySQL savepoint`

---

*Reference based on MySQL 8.0 documentation + real-world usage patterns.*
