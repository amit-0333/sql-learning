<div align="center">

```
███████╗██╗     ██╗ ██████╗ ██╗  ██╗████████╗███████╗
██╔════╝██║     ██║██╔════╝ ██║  ██║╚══██╔══╝██╔════╝
█████╗  ██║     ██║██║  ███╗███████║   ██║   ███████╗
██╔══╝  ██║     ██║██║   ██║██╔══██║   ██║   ╚════██║
██║     ███████╗██║╚██████╔╝██║  ██║   ██║   ███████║
╚═╝     ╚══════╝╚═╝ ╚═════╝ ╚═╝  ╚═╝  ╚═╝   ╚══════╝
```

### ✈️ Flights Analytics Dashboard — Python + MySQL + AWS RDS + Streamlit

> An end-to-end flights data pipeline — from raw CSV to a live interactive dashboard —  
> built with Python, Pandas, MySQL, AWS RDS, and Streamlit.

<br/>

![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![AWS](https://img.shields.io/badge/AWS_RDS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-150458?style=for-the-badge&logo=pandas&logoColor=white)
![Streamlit](https://img.shields.io/badge/Streamlit-FF4B4B?style=for-the-badge&logo=streamlit&logoColor=white)
![Plotly](https://img.shields.io/badge/Plotly-3F4F75?style=for-the-badge&logo=plotly&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-F37626?style=for-the-badge&logo=jupyter&logoColor=white)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)

</div>

---

## 📌 About

This project is a full data engineering + analytics pipeline built on Indian domestic flight data. It covers the complete journey from raw data to an interactive dashboard:

1. **Extract & Transform** — Load raw CSV into AWS RDS using `pandas` + `SQLAlchemy`
2. **Database Layer** — MySQL database hosted on **AWS RDS** (cloud), queried via Python OOP class
3. **CRUD Practice** — Basic `mysql-connector` operations (INSERT, SELECT, UPDATE, DELETE)
4. **Streamlit Dashboard** — Interactive app with flight search + 3 analytics charts (Pie, Bar, Line)

---

## 🗺️ Repository Structure

```
flights-sql-app/
│
├── 📄 app.py                                    # Streamlit dashboard — main application
├── 📄 dbhelper.py                               # DB class — all SQL queries via Python OOP
├── 📄 crud.py                                   # MySQL CRUD practice with mysql-connector
├── 📓 extract_transform.ipynb                   # EDA + data loading pipeline into AWS RDS
├── 📄 flights_cleaned.csv                       # Cleaned dataset (9 columns, Indian flights)
└── 📄 README.md
```

---

## 🗃️ Dataset — `flights_cleaned.csv`

Indian domestic flight data with **9 columns**:

| Column | Description |
|---|---|
| `Airline` | Airline name (IndiGo, Jet Airways, Air India…) |
| `Date_of_Journey` | Flight date (YYYY-MM-DD) |
| `Source` | Departure city |
| `Destination` | Arrival city |
| `Route` | Full route with stops (e.g. `BLR → BOM → DEL`) |
| `Dep_Time` | Departure time (HH:MM) |
| `Duration` | Flight duration in minutes |
| `Total_Stops` | Number of stops (non-stop, 1 stop, 2 stops…) |
| `Price` | Ticket price in INR |

---

## 🏗️ Pipeline Architecture

```
flights_cleaned.csv  (local CSV)
         ↓
extract_transform.ipynb
  └── pandas read_csv
  └── SQLAlchemy engine → df.to_sql()
         ↓
AWS RDS (MySQL) — database: flights, table: flights
         ↓
dbhelper.py (DB class)
  └── fetch_city_names()
  └── fetch_all_flights()
  └── fetch_airline_frequency()
  └── busy_airport()
  └── daily_frequency()
         ↓
app.py (Streamlit)
  └── Check Flights tab (search by Source → Destination)
  └── Analytics tab (Pie + Bar + Line charts)
```

---

## 📄 File Details

### `dbhelper.py` — Database Helper Class

OOP wrapper around `mysql-connector`. All SQL queries live here — `app.py` never writes raw SQL.

```python
class DB:
    def __init__(self):
        self.conn = mysql.connector.connect(
            host     = 'your-rds-endpoint.rds.amazonaws.com',
            user     = 'admin',
            password = 'your_password',
            database = 'flights'
        )
        self.mycursor = self.conn.cursor()
```

| Method | SQL Used | Returns |
|---|---|---|
| `fetch_city_names()` | `UNION` of Source + Destination | List of all cities |
| `fetch_all_flights(src, dst)` | `WHERE Source=? AND Destination=?` | Airline, Route, Time, Duration, Price |
| `fetch_airline_frequency()` | `GROUP BY Airline` | Airline names + flight counts |
| `busy_airport()` | `UNION ALL` Source+Destination, `GROUP BY` + `ORDER BY COUNT DESC` | City + total traffic |
| `daily_frequency()` | `GROUP BY Date_of_Journey` | Date + flight count per day |

---

### `app.py` — Streamlit Dashboard

Two-mode sidebar app:

**✈️ Check Flights**
- Select **Source** and **Destination** city from dropdowns (populated dynamically from DB)
- Click **Search** → shows a live table: Airline, Route, Dep_Time, Duration, Price

**📊 Analytics**
- **Pie chart** — Airline market share by number of flights (Plotly)
- **Bar chart** — Busiest airports ranked by total traffic (source + destination combined)
- **Line chart** — Daily flight frequency over time

---

### `crud.py` — CRUD Practice

Demonstrates basic `mysql-connector` operations on an `airport` table:

```python
# Table: airport → airport_id | code | city | name

# INSERT
mycursor.execute("INSERT INTO airport VALUES (1,'DEL','New Delhi','IGIA')")

# SELECT with filter
mycursor.execute("SELECT * FROM airport WHERE airport_id > 1")

# UPDATE
mycursor.execute("UPDATE airport SET name = 'Bombay' WHERE airport_id = 3")

# DELETE
mycursor.execute("DELETE FROM airport WHERE airport_id = 3")
```

---

### `extract_transform.ipynb` — Data Pipeline Notebook

Loads the cleaned CSV into AWS RDS using `SQLAlchemy`:

```python
import pandas as pd
from sqlalchemy import create_engine

df = pd.read_csv('flights_cleaned.csv')

engine = create_engine(
    "mysql+pymysql://admin:password@your-rds-endpoint/flights"
)

df.to_sql('flights', con=engine)   # uploads entire dataframe to MySQL table
```

Also contains a **Dream11 Fantasy Score UDF** (bonus experiment):

```python
def dream11(row):
    score = row['runs'] + row['fours'] + 2 * row['sixes']
    if row['runs'] >= 100:  score += 16
    elif row['runs'] >= 50: score += 8
    elif row['runs'] >= 30: score += 4
    elif row['runs'] == 0:  score -= 2
    return score

final_df['score'] = final_df.apply(dream11, axis=1)
```

---

## 🛠️ Tech Stack

| Tool | Purpose |
|---|---|
| 🐍 **Python** | Core language |
| 🐬 **MySQL** | Relational database |
| ☁️ **AWS RDS** | Cloud-hosted MySQL instance |
| 🐼 **Pandas** | Data loading, cleaning, transformation |
| 🔌 **mysql-connector-python** | Python ↔ MySQL bridge |
| ⚗️ **SQLAlchemy** | ORM engine for `df.to_sql()` bulk upload |
| 🚀 **Streamlit** | Interactive dashboard frontend |
| 📈 **Plotly** | Pie, bar, and line chart visualizations |
| 📓 **Jupyter Notebook** | EDA + pipeline workflow |

---

## ▶️ Run Locally

```bash
# 1. Clone the repo
git clone https://github.com/amit-0333/flights-sql-app.git
cd flights-sql-app

# 2. Install dependencies
pip install streamlit pandas mysql-connector-python sqlalchemy pymysql plotly

# 3. Set up AWS RDS or local MySQL
#    Update host / user / password in dbhelper.py

# 4. Load the dataset into MySQL
#    Open extract_transform.ipynb and run all cells

# 5. Launch the dashboard
streamlit run app.py
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
