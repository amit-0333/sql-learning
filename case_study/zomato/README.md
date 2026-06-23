# 🍕 Zomato SQL Case Study

> A real-world SQL case study modeled on Zomato's data structure — covering joins, aggregations, subqueries, and business analytics queries.

---

## 📚 Table of Contents

- [Schema Overview](#schema-overview)
- [Table Setup](#table-setup)
- [Table Relationships](#table-relationships)
- [Key Concepts Used](#key-concepts-used)

---

## Schema Overview

| Table | Description |
|---|---|
| `users` | Customer information |
| `orders` | Orders placed by users at restaurants |
| `order_details` | Individual food items within each order |
| `food` | Food items available |
| `menu` | Menu linking restaurants to food items with prices |
| `restaurants` | Restaurant information |
| `delivery_partner` | Delivery partner information |

---

## Table Setup

```sql
CREATE DATABASE IF NOT EXISTS zomato;
USE zomato;

CREATE TABLE users (
    user_id   INTEGER PRIMARY KEY AUTO_INCREMENT,
    name      VARCHAR(100) NOT NULL,
    email     VARCHAR(150) NOT NULL UNIQUE,
    password  VARCHAR(255)
);

CREATE TABLE restaurants (
    r_id    INTEGER PRIMARY KEY AUTO_INCREMENT,
    r_name  VARCHAR(100) NOT NULL,
    city    VARCHAR(100)
);

CREATE TABLE food (
    f_id    INTEGER PRIMARY KEY AUTO_INCREMENT,
    f_name  VARCHAR(100) NOT NULL,
    type    VARCHAR(10) CHECK (type IN ('Veg', 'Non-Veg'))
);

CREATE TABLE menu (
    menu_id  INTEGER PRIMARY KEY AUTO_INCREMENT,
    r_id     INTEGER NOT NULL,
    f_id     INTEGER NOT NULL,
    price    DECIMAL(8,2) NOT NULL,
    CONSTRAINT menu_r_fk FOREIGN KEY (r_id) REFERENCES restaurants(r_id),
    CONSTRAINT menu_f_fk FOREIGN KEY (f_id) REFERENCES food(f_id)
);

CREATE TABLE delivery_partner (
    partner_id  INTEGER PRIMARY KEY AUTO_INCREMENT,
    partner_name VARCHAR(100) NOT NULL
);

CREATE TABLE orders (
    order_id         INTEGER PRIMARY KEY AUTO_INCREMENT,
    user_id          INTEGER NOT NULL,
    r_id             INTEGER NOT NULL,
    amount           DECIMAL(10,2) NOT NULL,
    date             DATETIME DEFAULT CURRENT_TIMESTAMP,
    partner_id       INTEGER,
    delivery_time    INTEGER,
    delivery_rating  DECIMAL(3,1),
    restaurant_rating DECIMAL(3,1),
    CONSTRAINT orders_user_fk FOREIGN KEY (user_id) REFERENCES users(user_id),
    CONSTRAINT orders_r_fk    FOREIGN KEY (r_id)    REFERENCES restaurants(r_id),
    CONSTRAINT orders_p_fk    FOREIGN KEY (partner_id) REFERENCES delivery_partner(partner_id)
);

CREATE TABLE order_details (
    order_detail_id INTEGER PRIMARY KEY AUTO_INCREMENT,
    order_id        INTEGER NOT NULL,
    f_id            INTEGER NOT NULL,
    CONSTRAINT od_order_fk FOREIGN KEY (order_id) REFERENCES orders(order_id),
    CONSTRAINT od_food_fk  FOREIGN KEY (f_id)     REFERENCES food(f_id)
);
```

---

## Table Relationships

```
users ←──── user_id ────→ orders ←──── partner_id ────→ delivery_partner
                              ↕
                          order_id
                              ↕
                        order_details
                              ↕
                            f_id
                              ↕
restaurants ←── r_id ──→ menu ←── f_id ──→ food
```

- `orders` links `users` and `restaurants` via `user_id` and `r_id`
- `order_details` links `orders` and `food` via `order_id` and `f_id`
- `menu` links `restaurants` and `food` via `r_id` and `f_id`
- `delivery_partner` links to `orders` via `partner_id`

---

## Key Concepts Used

- Basic SELECT, WHERE, GROUP BY, ORDER BY, LIMIT
- Aggregate functions: COUNT, AVG, SUM, MIN, MAX
- Multi-table JOINs (3-4 tables)
- Subqueries (Scalar, Row, Correlated)
- Date functions: MONTH(), YEAR()
- HAVING clause
- NULL handling: IS NULL, IS NOT NULL
- Formula-based computed columns

---

*Case study based on Zomato-style dataset — DSMP 2022-23*
