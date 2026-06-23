<div align="center">

```
███████╗ ██████╗ ██╗         ██╗     ███████╗ █████╗ ██████╗ ███╗   ██╗██╗███╗   ██╗ ██████╗ 
██╔════╝██╔═══██╗██║         ██║     ██╔════╝██╔══██╗██╔══██╗████╗  ██║██║████╗  ██║██╔════╝ 
███████╗██║   ██║██║         ██║     █████╗  ███████║██████╔╝██╔██╗ ██║██║██╔██╗ ██║██║  ███╗
╚════██║██║▄▄ ██║██║         ██║     ██╔══╝  ██╔══██║██╔══██╗██║╚██╗██║██║██║╚██╗██║██║   ██║
███████║╚██████╔╝███████╗    ███████╗███████╗██║  ██║██║  ██║██║ ╚████║██║██║ ╚████║╚██████╔╝
╚══════╝ ╚══▀▀═╝ ╚══════╝    ╚══════╝╚══════╝╚═╝  ╚═╝╚═╝  ╚═╝╚═╝  ╚═══╝╚═╝╚═╝  ╚═══╝ ╚═════╝
```

### 🗄️ SQL Learning Journey — Complete Repository

> A structured, end-to-end SQL learning repo covering database fundamentals, real-world case studies  
> (Zomato & Flights Dashboard), Python + AWS integration, EDA with Datetime & UDFs,  
> Database Design, and Transactions — from beginner to production-ready.

<br/>

![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-150458?style=for-the-badge&logo=pandas&logoColor=white)
![Streamlit](https://img.shields.io/badge/Streamlit-FF4B4B?style=for-the-badge&logo=streamlit&logoColor=white)
![AWS](https://img.shields.io/badge/AWS_RDS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-F37626?style=for-the-badge&logo=jupyter&logoColor=white)
![Status](https://img.shields.io/badge/Status-In_Progress-blue?style=for-the-badge)

</div>

---

## 📌 About

This repository is a complete SQL learning journey organized module by module — from fundamentals to real-world industry use. It covers:

1. **Core SQL** — DDL, DML, Sorting, JOINs, Subqueries, Window Functions  
2. **Zomato Case Study** — Multi-table SQL analytics on a food delivery schema  
3. **Flights Dashboard** — Python + SQL + AWS RDS end-to-end pipeline with EDA  
4. **Datetime & UDFs** — Datetime analysis with pandas and MySQL User Defined Functions  
5. **Database Design** — Normalization, ER diagrams, indexing  
6. **Transactions** — COMMIT, ROLLBACK, SAVEPOINT with ACID guarantees  
7. **Practice Questions** — 4 full question sets with answer keys  

---

## 🗺️ Repository Structure

```
sql-learning/
│
├── 📁 1_database_fundamentals/       # RDBMS, keys, normalization, ACID
├── 📁 2_SQL_DDL_COMMANDS/            # CREATE, ALTER, DROP, TRUNCATE, constraints
├── 📁 3_SQL_DML_COMMANDS/            # INSERT, UPDATE, DELETE, SELECT, functions
├── 📁 4_SORTING_GROUPBY_HAVING/      # ORDER BY, GROUP BY, HAVING, query order
├── 📁 5_SQL_JOINS_SET_OPERATIONS/    # INNER, LEFT, RIGHT, SELF, CROSS JOIN
├── 📁 6_SQL_SUBQUERY/                # Scalar, row, correlated, nested subqueries
├── 📁 7_WINDOW_FUNCTIONS/            # RANK, DENSE_RANK, LAG, LEAD, running totals
├── 📁 case_study/                    # Zomato + Flights real-world case studies
├── 📁 practice_questions/            # Question sets + answer keys (PDF)
└── 📄 README.md
```

---

## 📚 Learning Roadmap

```
Database Fundamentals
        ↓
  DDL → DML → Sorting & Grouping
        ↓
    JOINs & Set Operations
        ↓
        Subqueries
        ↓
    Window Functions
        ↓
Case Studies: Zomato + Flights
        ↓
EDA · Datetime · UDFs · Transactions
        ↓
    Python + AWS Integration
```

---

## 🧩 Module Breakdown

### 1. 🗄️ Database Fundamentals
- What is SQL and RDBMS
- Relational model: tables, rows, columns, keys
- Primary Key, Foreign Key, Composite Key
- Normalization: 1NF → 2NF → 3NF → BCNF
- ACID properties
- ER Diagrams and schema design

---

### 2. 🔧 DDL — Data Definition Language

| Command | Purpose |
|---|---|
| `CREATE` | Create databases, tables, indexes |
| `ALTER` | Add / drop / modify columns and constraints |
| `DROP` | Permanently delete objects |
| `TRUNCATE` | Remove all rows, keep structure |

**Key Topics:** named constraints, `ON DELETE CASCADE`, `ALTER TABLE`, `TRUNCATE` vs `DELETE`, schema versioning best practices

---

### 3. ✏️ DML — Data Manipulation Language

| Command | Purpose |
|---|---|
| `INSERT` | Add new rows (single, bulk, from SELECT) |
| `UPDATE` | Modify existing rows |
| `DELETE` | Remove specific rows |
| `SELECT` | Query and retrieve data |

**Key Topics:** aggregate functions (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`, `STD`, `VARIANCE`), scalar functions (`ROUND`, `FLOOR`, `CEIL`, `ABS`), GROUP BY + HAVING

---

### 4. 📊 Sorting, Grouping & HAVING

```sql
-- SQL Query Execution Order (always remember this!)
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```

- `ORDER BY` — single and multi-column sorting
- `GROUP BY` — aggregate per group
- `HAVING` — filter after grouping (vs `WHERE` which filters before)

---

### 5. 🔗 JOINs & Set Operations

| JOIN Type | Returns |
|---|---|
| `INNER JOIN` | Only matching rows in both tables |
| `LEFT JOIN` | All rows from left + matched from right |
| `RIGHT JOIN` | All rows from right + matched from left |
| `FULL JOIN` | All rows from both tables |
| `SELF JOIN` | Table joined with itself |
| `CROSS JOIN` | Cartesian product |

**Set Operations:** `UNION`, `UNION ALL`, `INTERSECT`, `EXCEPT`

---

### 6. 🔍 Subqueries

| Type | Description |
|---|---|
| Scalar | Returns a single value |
| Row | Returns a single row |
| Table | Returns a full table (used in FROM) |
| Correlated | References outer query — runs once per row |

```sql
-- Find movies with above-average score
SELECT * FROM movies
WHERE score > (SELECT AVG(score) FROM movies);
```

---

### 7. 🪟 Window Functions

```sql
function_name() OVER (
    PARTITION BY col    -- divide into groups
    ORDER BY col        -- row order within group
    ROWS BETWEEN ...    -- define frame
)
```

| Function | Use |
|---|---|
| `ROW_NUMBER()` | Unique row number per row |
| `RANK()` | Rank with gaps on ties |
| `DENSE_RANK()` | Rank without gaps on ties |
| `LEAD()` / `LAG()` | Next / previous row value |
| `SUM() OVER(...)` | Running / cumulative total |
| `AVG() OVER(...)` | Running average |

---

## 🍕 Case Study 1 — Zomato SQL

> Real-world SQL analytics on a food delivery database — modeled on Zomato's production schema.

### Schema Diagram

```
users ←── user_id ──→ orders ←── partner_id ──→ delivery_partner
                          ↕
                      order_id
                          ↕
                    order_details
                          ↕
                        f_id
                          ↕
restaurants ←── r_id ──→ menu ←── f_id ──→ food
```

| Table | Description |
|---|---|
| `users` | Customer information |
| `restaurants` | Restaurant details |
| `food` | Food item catalog |
| `menu` | Restaurant ↔ food mapping with prices |
| `orders` | Orders placed by customers |
| `order_details` | Individual food items per order |
| `delivery_partner` | Delivery partner info |

### Key Queries Practiced
- Find customers who have **never ordered** (NULL handling)
- Find the **favourite food** of each customer (most ordered item per customer)
- Find restaurants with **revenue > X** in a given month
- Delivery partner compensation: `(deliveries × 100) + (1000 × avg_rating)`
- **Correlation** between delivery time and total rating
- Find all **veg-only restaurants** using subquery + NOT IN
- Top 3 most ordered food items overall
- Customers whose total spending is above average

---

## ✈️ Case Study 2 — Flights Dashboard (Python + SQL + AWS)

> End-to-end data pipeline: raw CSV → EDA → cleaned dataset → MySQL/AWS RDS → Streamlit dashboard.

### Pipeline Architecture

```
Raw CSV (laptop)
      ↓
Python EDA (pandas + seaborn)
      ↓
Data Cleaning & Datetime Analysis
      ↓
MySQL DB (local) / AWS RDS (cloud)
      ↓
SQL Queries via Python
      ↓
Streamlit Flights Dashboard
```

### Datetime Analysis (Python)

```python
import pandas as pd

df['fl_date'] = pd.to_datetime(df['fl_date'])
df['month']       = df['fl_date'].dt.month
df['day_of_week'] = df['fl_date'].dt.day_name()
df['hour']        = df['fl_date'].dt.hour

# Monthly flight volume trend
df.groupby('month')['flight_id'].count().plot(kind='bar', title='Flights per Month')

# Average delay by hour of day
df.groupby('hour')['arr_delay'].mean().plot(title='Avg Delay by Hour')
```

### User Defined Functions — Python

```python
def classify_delay(minutes):
    if minutes <= 0:      return 'On Time'
    elif minutes <= 30:   return 'Minor Delay'
    elif minutes <= 120:  return 'Major Delay'
    else:                 return 'Severe Delay'

df['delay_category'] = df['arr_delay'].apply(classify_delay)
```

### User Defined Functions — MySQL

```sql
DELIMITER $$
CREATE FUNCTION classify_delay(minutes INT)
RETURNS VARCHAR(20) DETERMINISTIC
BEGIN
    IF      minutes <= 0   THEN RETURN 'On Time';
    ELSEIF  minutes <= 30  THEN RETURN 'Minor Delay';
    ELSEIF  minutes <= 120 THEN RETURN 'Major Delay';
    ELSE                        RETURN 'Severe Delay';
    END IF;
END$$
DELIMITER ;

-- Usage
SELECT flight_id, classify_delay(arr_delay) AS status FROM flights;
```

### Python + AWS RDS Integration

```python
import mysql.connector, pandas as pd

conn = mysql.connector.connect(
    host     = 'your-rds-endpoint.amazonaws.com',
    user     = 'admin',
    password = 'your_password',
    database = 'flights_db'
)

query = """
    SELECT origin, dest, AVG(arr_delay) AS avg_delay
    FROM flights
    WHERE MONTH(fl_date) = 6
    GROUP BY origin, dest
    ORDER BY avg_delay DESC
    LIMIT 10;
"""
df = pd.read_sql(query, conn)
conn.close()
```

---

## 🗃️ Database Design

### Normalization

| Form | Rule |
|---|---|
| 1NF | Atomic values, no repeating groups |
| 2NF | No partial dependency on composite PK |
| 3NF | No transitive dependency |
| BCNF | Every determinant is a candidate key |

### Indexing Strategy

```sql
-- Index on frequently filtered columns
CREATE INDEX idx_orders_user ON orders(user_id);
CREATE INDEX idx_orders_date ON orders(date);

-- Composite index for multi-column WHERE
CREATE INDEX idx_orders_user_date ON orders(user_id, date);
```

---

## 🔄 Transactions (TCL)

```sql
-- Full transaction: transfer money between accounts
START TRANSACTION;
    UPDATE accounts SET balance = balance - 5000 WHERE account_id = 1;
    UPDATE accounts SET balance = balance + 5000 WHERE account_id = 2;
COMMIT;

-- Rollback on failure
START TRANSACTION;
    UPDATE accounts SET balance = balance - 5000 WHERE account_id = 1;
    -- error occurs...
ROLLBACK;

-- Savepoint: partial rollback
START TRANSACTION;
    INSERT INTO orders (...) VALUES (...);
    SAVEPOINT sp1;
    UPDATE inventory SET stock = stock - 1 WHERE item_id = 101;
    ROLLBACK TO sp1;   -- undo only the UPDATE
COMMIT;
```

**ACID Properties:**

| Property | Meaning |
|---|---|
| **Atomicity** | All-or-nothing — partial success not allowed |
| **Consistency** | DB moves from one valid state to another |
| **Isolation** | Concurrent transactions don't interfere |
| **Durability** | Committed data survives crashes |

---

## 📝 Practice Questions

| Topic | Questions | Answers |
|---|---|---|
| DDL + DML + Functions | `SQL_Question_Set.pdf` | `SQL_Answer_Key.pdf` |
| Subqueries | `SQL_Subquery_Question_Set.pdf` | `SQL_Subquery_Answer_Key.pdf` |
| Window Functions | `Window_Functions_Questions.pdf` | `Window_Functions_Answers.pdf` |
| Zomato Case Study | `Zomato_Case_Study_Questions.pdf` | `Zomato_Case_Study_Answers.pdf` |

---

## 🛠️ Tech Stack

| Tool | Purpose |
|---|---|
| 🐬 **MySQL** | Primary database engine |
| 🐍 **Python** | EDA, data cleaning, DB integration |
| 🐼 **Pandas** | Data manipulation and datetime analysis |
| 📊 **Matplotlib / Seaborn** | Visualization |
| 🚀 **Streamlit** | Flights dashboard frontend |
| 🔌 **mysql-connector-python** | Python ↔ MySQL bridge |
| ☁️ **AWS RDS** | Cloud-hosted MySQL for Flights project |
| 📓 **Jupyter Notebook** | EDA workflow |

---

## ▶️ Run Locally

```bash
# 1. Clone the repo
git clone https://github.com/amit-0333/sql-learning.git
cd sql-learning

# 2. Set up MySQL and load Zomato schema
mysql -u root -p
# Run: case_study/zomato_schema.sql

# 3. Flights dashboard setup
pip install pandas mysql-connector-python matplotlib streamlit seaborn
python case_study/flights/load_data.py      # Load CSV into MySQL
streamlit run case_study/flights/app.py     # Launch dashboard
```

---

## 👨‍💻 Author

**Amit Kumar**

[![GitHub](https://img.shields.io/badge/GitHub-amit--0333-181717?style=flat&logo=github)](https://github.com/amit-0333)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Amit%20Kumar-0077B5?style=flat&logo=linkedin)](https://www.linkedin.com/in/amit-kumar-a62a3640a/)

---

<div align="center">

> 📝 *Built as part of my Data Science and SQL learning journey.*

⭐ **Star this repo if you found it useful!**

</div>
