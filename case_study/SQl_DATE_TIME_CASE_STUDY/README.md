# SQL DateTime Case Study — Flights Dataset

## Overview

This project is a case study on SQL **DateTime functions** using a real-world flights dataset. It covers a wide range of datetime operations including date parsing, time filtering, duration calculations, arrival time derivation, and pivot-style aggregations.

---

## Dataset

**Table:** `flightss` (in the `flights` database)

| Column | Type | Description |
|---|---|---|
| index | INT | Row identifier |
| Airline | VARCHAR | Name of the airline (e.g. Jet Airways, IndiGo, Air India) |
| Date_of_Journey | DATE | Date the flight departs |
| Source | VARCHAR | Departure city (e.g. Banglore, Delhi) |
| Destination | VARCHAR | Arrival city (e.g. New Delhi, Mumbai) |
| Dep_Time | TIME | Departure time in HH:MM format |
| Duration | VARCHAR | Flight duration as text (e.g. "13h 5m", "2h 30m") |
| Total_Stops | VARCHAR | Number of stops (e.g. "non-stop", "1 stop", "2 stops") |
| Additional_Info | VARCHAR | Extra info (e.g. "No info", "1 Long layover") |
| Price | INT | Ticket price in INR |

**Sample Data:**

| Airline | Date_of_Journey | Source | Destination | Dep_Time | Duration | Total_Stops | Price |
|---|---|---|---|---|---|---|---|
| Jet Airways | 2019-01-03 | Banglore | New Delhi | 11:40 | 13h 5m | 1 stop | 26890 |
| SpiceJet | 2019-01-03 | Banglore | New Delhi | 15:35 | 8h 5m | 1 stop | 7744 |
| Air India | 2019-01-03 | Banglore | New Delhi | 8:50 | 39h 5m | 2 stops | 17135 |

---

## Schema Changes (Setup for Q6 onwards)

Three new columns were added to support arrival time calculations:

```sql
ALTER TABLE flightss ADD COLUMN departure DATETIME;
ALTER TABLE flightss ADD COLUMN duration_mins INTEGER;
ALTER TABLE flightss ADD COLUMN arrival DATETIME;
```

These were populated using:
- `STR_TO_DATE()` + `CONCAT()` to build the `departure` DATETIME
- A `CASE WHEN` expression to parse the `Duration` text into `duration_mins`
- `DATE_ADD()` with `INTERVAL` to compute `arrival`

---

## Queries

### Q1. Month with most number of flights
```sql
SELECT MONTHNAME(Date_of_journey), COUNT(*)
FROM flightss
GROUP BY MONTHNAME(Date_of_journey)
ORDER BY COUNT(*) DESC LIMIT 1;
```

---

### Q2. Weekday with most costly flights
```sql
SELECT DAYNAME(Date_of_journey), AVG(price)
FROM flightss
GROUP BY DAYNAME(Date_of_journey)
ORDER BY AVG(price) DESC LIMIT 1;
```

---

### Q3. Number of IndiGo flights every month
```sql
SELECT MONTHNAME(Date_of_journey), COUNT(*) FROM flightss
WHERE Airline = 'IndiGo'
GROUP BY MONTHNAME(Date_of_journey)
ORDER BY MONTHNAME(Date_of_journey);
```

---

### Q4. Flights departing 10 AM – 2 PM from Bangalore to Delhi
```sql
SELECT * FROM flightss
WHERE Source = 'Banglore' AND Destination = 'Delhi'
AND TIME(Dep_time) BETWEEN '10:00:00' AND '14:00:00';
```

---

### Q5. Number of weekend flights from Bangalore
```sql
SELECT COUNT(*) FROM flightss
WHERE source = 'banglore'
AND DAYNAME(date_of_journey) IN ('saturday', 'sunday');
```

---

### Q6. Calculate arrival time by adding duration to departure time
```sql
-- Step 1: Create departure DATETIME
UPDATE flightss
SET departure = STR_TO_DATE(CONCAT(date_of_journey,' ',dep_time), '%Y-%m-%d %H:%i');

-- Step 2: Parse duration text into minutes
UPDATE flightss
SET duration_mins =
CASE
    WHEN duration LIKE '%h' AND duration NOT LIKE '% %'
        THEN REPLACE(duration, 'h', '') * 60
    WHEN duration LIKE '%h %m'
        THEN REPLACE(SUBSTRING_INDEX(duration,' ',1), 'h', '') * 60
           + REPLACE(SUBSTRING_INDEX(duration,' ',-1), 'm', '')
    WHEN duration LIKE '%m'
        THEN REPLACE(duration, 'm', '')
    ELSE NULL
END;

-- Step 3: Calculate arrival
UPDATE flightss
SET arrival = DATE_ADD(departure, INTERVAL duration_mins MINUTE);
```

---

### Q7. Arrival date for all flights
```sql
SELECT DATE(arrival) FROM flightss;
```

---

### Q8. Number of flights travelling on multiple dates
```sql
SELECT COUNT(*) FROM flightss
WHERE DATE(departure) != DATE(arrival);
```

---

### Q9. Average duration between city pairs in xh ym format
```sql
SELECT source, destination,
TIME_FORMAT(SEC_TO_TIME(AVG(duration_mins)*60), '%kh %im') AS avg_duration
FROM flightss
GROUP BY source, destination;
```

---

### Q10. Non-stop flights departing before midnight, arriving after midnight
```sql
SELECT * FROM flightss
WHERE total_stops = 'non-stop'
AND DATE(departure) < DATE(arrival);
```

---

### Q11. Quarter-wise number of flights per airline
```sql
SELECT airline, QUARTER(departure), COUNT(*)
FROM flightss
GROUP BY airline, QUARTER(departure);
```

---

### Q12. Average duration and price — non-stop vs with stops
```sql
WITH stop_table AS (
    SELECT *,
        CASE
            WHEN Total_stops = 'non-stop' THEN 'non-stop'
            ELSE 'with stop'
        END AS stops
    FROM flightss
)
SELECT stops,
TIME_FORMAT(SEC_TO_TIME(AVG(duration_mins)*60), '%kh %im') AS avg_duration,
AVG(price) AS avg_price
FROM stop_table
GROUP BY stops;
```

---

### Q13. Air India flights from Delhi in a date range
```sql
SELECT * FROM flightss
WHERE airline = 'Air India'
AND DATE(departure) BETWEEN '2019-03-01' AND '2019-03-10';
```

---

### Q14. Longest flight per airline
```sql
SELECT airline,
TIME_FORMAT(SEC_TO_TIME(MAX(duration_mins)*60), '%kh %im') AS max_duration
FROM flightss
GROUP BY airline
ORDER BY MAX(duration_mins) DESC;
```

---

### Q15. City pairs with average duration > 3 hours
```sql
SELECT source, destination,
TIME_FORMAT(SEC_TO_TIME(AVG(duration_mins)*60), '%kh %im') AS avg_duration
FROM flightss
GROUP BY source, destination
HAVING AVG(duration_mins) > 180;
```

---

### Q16. Weekday vs time grid — flight frequency from Bangalore & Delhi
```sql
SELECT
    DAYNAME(departure) AS weekday,
    SUM(CASE WHEN HOUR(departure) BETWEEN 0 AND 5 THEN 1 ELSE 0 END) AS '12AM - 6AM',
    SUM(CASE WHEN HOUR(departure) BETWEEN 6 AND 11 THEN 1 ELSE 0 END) AS '6AM - 12PM',
    SUM(CASE WHEN HOUR(departure) BETWEEN 12 AND 17 THEN 1 ELSE 0 END) AS '12PM - 6PM',
    SUM(CASE WHEN HOUR(departure) BETWEEN 18 AND 23 THEN 1 ELSE 0 END) AS '6PM - 12AM'
FROM flightss
WHERE source IN ('Banglore', 'Delhi')
GROUP BY DAYOFWEEK(departure), weekday
ORDER BY DAYOFWEEK(departure);
```

---

### Q17. Weekday vs time grid — avg flight price from Bangalore to Delhi
```sql
SELECT
    DAYNAME(departure) AS weekday,
    AVG(CASE WHEN HOUR(departure) BETWEEN 0 AND 5 THEN price END) AS '12AM - 6AM',
    AVG(CASE WHEN HOUR(departure) BETWEEN 6 AND 11 THEN price END) AS '6AM - 12PM',
    AVG(CASE WHEN HOUR(departure) BETWEEN 12 AND 17 THEN price END) AS '12PM - 6PM',
    AVG(CASE WHEN HOUR(departure) BETWEEN 18 AND 23 THEN price END) AS '6PM - 12AM'
FROM flightss
WHERE source = 'Banglore' AND destination = 'Delhi'
GROUP BY DAYOFWEEK(departure), DAYNAME(departure)
ORDER BY DAYOFWEEK(departure);
```

---

## Key SQL DateTime Functions Used

| Function | Purpose |
|---|---|
| `MONTHNAME()` | Extract month name from a date |
| `DAYNAME()` | Extract weekday name from a date |
| `DAYOFWEEK()` | Numeric weekday (1 = Sunday) for ordering |
| `QUARTER()` | Extract quarter (1–4) from a date |
| `HOUR()` | Extract hour from a time/datetime |
| `TIME()` | Extract time portion from a datetime |
| `DATE()` | Extract date portion from a datetime |
| `STR_TO_DATE()` | Parse a string into a DATE/DATETIME |
| `DATE_ADD()` | Add an interval to a date/datetime |
| `SEC_TO_TIME()` | Convert seconds to TIME format |
| `TIME_FORMAT()` | Format a time value as a string |
| `CONCAT()` | Combine strings (used to build datetime string) |
| `BETWEEN` | Filter values within a range |

---

## Tools & Environment

- **Database:** MySQL
- **Dataset:** Indian domestic flights (2019)
- **Concepts covered:** Date filtering, time range queries, string-to-date parsing, duration arithmetic, pivot aggregations with CASE WHEN, CTEs
