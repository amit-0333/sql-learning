# 🗄️ Database Fundamentals

Before writing any SQL, it's important to understand what a database actually is, how data is organized within one, and the core concepts that every relational database is built on.

---

## 🎯 Goal of This Section

- Understand what a database and a DBMS are
- Understand the relational model (tables, rows, columns)
- Understand keys and how tables relate to each other
- Understand normalization and why it matters
- Build the conceptual foundation needed before learning DDL/DML/DQL

---

## 1️⃣ What is a Database?

A **database** is an organized collection of structured data, stored electronically, that can be easily accessed, managed, and updated.

A **DBMS (Database Management System)** is the software that lets users create, read, update, and manage databases — e.g. MySQL, PostgreSQL, SQLite, Oracle, SQL Server.

### RDBMS (Relational Database Management System)
An RDBMS organizes data into **tables** (relations) that can be linked to each other based on common data. Examples: MySQL, PostgreSQL, SQLite, Microsoft SQL Server, Oracle.

### SQL vs NoSQL
| | SQL (Relational) | NoSQL (Non-relational) |
|---|---|---|
| Structure | Tables with fixed schema | Flexible: documents, key-value, graph, wide-column |
| Examples | MySQL, PostgreSQL, SQLite | MongoDB, Redis, Cassandra |
| Best for | Structured data, complex relationships | Unstructured/semi-structured data, horizontal scaling |
| Query language | SQL | Varies by database |

---

## 2️⃣ The Relational Model

A relational database organizes data into **tables**, made up of:

- **Table (Relation)** – a collection of related data, organized in rows and columns (e.g. `students`, `orders`)
- **Row (Record / Tuple)** – a single entry in a table, representing one instance of the entity (e.g. one student)
- **Column (Field / Attribute)** – a specific property of the entity (e.g. `name`, `age`, `email`)
- **Schema** – the structure/blueprint of the database: tables, columns, data types, and relationships
- **Data Type** – defines what kind of value a column can hold (`INT`, `VARCHAR`, `DATE`, `BOOLEAN`, `DECIMAL`, etc.)

```
students
+------------+-----------+--------+----------------------+
| student_id | name      | age    | email                |
+------------+-----------+--------+----------------------+
| 1          | Alice     | 21     | alice@email.com      |
| 2          | Bob       | 22     | bob@email.com        |
+------------+-----------+--------+----------------------+
```

---

## 3️⃣ Keys

Keys are how rows are uniquely identified and how tables are connected to one another.

### Primary Key (PK)
A column (or set of columns) that **uniquely identifies** each row in a table. Cannot be NULL, must be unique.

```sql
student_id INT PRIMARY KEY
```

### Foreign Key (FK)
A column in one table that refers to the **primary key** of another table — this is how relationships between tables are established.

```sql
-- 'orders' table referencing 'students' table
student_id INT,
FOREIGN KEY (student_id) REFERENCES students(student_id)
```

### Candidate Key
Any column (or combination) that *could* qualify as a primary key — i.e. it's unique and not null. A table may have multiple candidate keys, but only one becomes the primary key.

### Composite Key
A primary key made up of **two or more columns** combined, used when no single column is unique on its own.

```sql
PRIMARY KEY (order_id, product_id)
```

### Unique Key
Similar to a primary key — enforces uniqueness — but a table can have multiple unique keys, and they can allow one NULL value (depending on the RDBMS).

### Super Key
Any set of columns that can uniquely identify a row, including extra/redundant columns beyond what's strictly needed (a candidate key is a *minimal* super key).

---

## 4️⃣ Types of Relationships

| Relationship | Example | How it's modeled |
|---|---|---|
| **One-to-One** | A person and their passport | FK with a unique constraint |
| **One-to-Many** | A customer and their orders | FK on the "many" side |
| **Many-to-Many** | Students and courses | A separate **junction/bridge table** with FKs to both |

```sql
-- Junction table example for many-to-many
CREATE TABLE student_courses (
    student_id INT,
    course_id INT,
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES students(student_id),
    FOREIGN KEY (course_id) REFERENCES courses(course_id)
);
```

---

## 5️⃣ Constraints

Rules enforced on columns to maintain data accuracy and integrity.

| Constraint | Purpose |
|---|---|
| `PRIMARY KEY` | Uniquely identifies each row |
| `FOREIGN KEY` | Enforces a link to another table's primary key |
| `NOT NULL` | Column cannot have a NULL value |
| `UNIQUE` | All values in the column must be different |
| `CHECK` | Restricts values based on a condition (e.g. `age > 0`) |
| `DEFAULT` | Sets a default value when none is provided |
| `AUTO_INCREMENT` / `SERIAL` | Automatically generates sequential values (commonly used for PKs) |

---

## 6️⃣ Normalization

**Normalization** is the process of organizing data to reduce redundancy and improve data integrity, by splitting large tables into smaller, related tables.

### Why Normalize?
- Avoid storing the same data in multiple places (redundancy)
- Avoid update/insert/delete anomalies
- Keep data consistent

### Normal Forms (Most Common)

**1NF (First Normal Form)**
- Each column contains atomic (indivisible) values
- No repeating groups or arrays within a single column

**2NF (Second Normal Form)**
- Must satisfy 1NF
- Every non-key column must depend on the *whole* primary key (relevant for composite keys) — no partial dependency

**3NF (Third Normal Form)**
- Must satisfy 2NF
- No transitive dependency — non-key columns must depend only on the primary key, not on other non-key columns

```
Unnormalized:
orders(order_id, customer_name, customer_email, product, price)

Normalized (3NF):
customers(customer_id, customer_name, customer_email)
orders(order_id, customer_id, product, price)
```

### Denormalization
Sometimes data is intentionally **denormalized** (reintroducing redundancy) to improve read performance in reporting/analytics systems — a common trade-off in real-world database design.

---

## 7️⃣ ACID Properties

Properties that guarantee reliable processing of database transactions.

| Property | Meaning |
|---|---|
| **Atomicity** | A transaction is all-or-nothing — it fully completes or fully fails |
| **Consistency** | A transaction brings the database from one valid state to another |
| **Isolation** | Concurrent transactions don't interfere with each other |
| **Durability** | Once committed, changes persist even after a system failure |

---

## 8️⃣ Categories of SQL Commands

SQL commands are grouped based on what they do:

| Category | Stands For | Examples | Purpose |
|---|---|---|---|
| **DDL** | Data Definition Language | `CREATE`, `ALTER`, `DROP`, `TRUNCATE` | Define/modify database structure |
| **DML** | Data Manipulation Language | `INSERT`, `UPDATE`, `DELETE` | Modify the actual data |
| **DQL** | Data Query Language | `SELECT` | Retrieve data |
| **DCL** | Data Control Language | `GRANT`, `REVOKE` | Manage access/permissions |
| **TCL** | Transaction Control Language | `COMMIT`, `ROLLBACK`, `SAVEPOINT` | Manage transactions |

This learning path will go through these categories one at a time, starting with **DDL** next.

---

## ✅ Checklist Before Moving to DDL

- [ ] Understand what a database, DBMS, and RDBMS are
- [ ] Understand tables, rows, columns, and schema
- [ ] Understand primary keys, foreign keys, and table relationships
- [ ] Understand normalization and why redundancy is avoided
- [ ] Understand ACID properties at a high level
- [ ] Know the 5 categories of SQL commands

---

## ➡️ Next Step

Proceed to **02_ddl** to learn how to actually create and define database structures (`CREATE`, `ALTER`, `DROP`, constraints) using SQL.
