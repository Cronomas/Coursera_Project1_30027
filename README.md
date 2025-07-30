# Coursera_Project1_30027
# Coffee_Shop

## Project Overview

The **Coffee_Shop** project is a relational database management system developed using PostgreSQL to support a retail coffee store’s operational and analytical needs. The database is designed to efficiently manage store operations, staff, customers, product inventory, and sales transactions in a normalized and scalable manner.

---

## Mission Objectives

- Design and implement a fully normalized database schema for a coffee retail chain.
- Ensure data integrity and reduce redundancy by applying normalization principles (up to 3NF).
- Enable accurate tracking of sales transactions and inventory movement.
- Support analytical views using SQL Views and Materialized Views.
- Ensure compatibility with both PostgreSQL (primary) and MySQL (secondary for replication/import).

---

## System Definition

The **Coffee_Shop** system is a transactional database that:

- Manages staff assigned to various retail outlets.
- Keeps records of customers, products, and product categories.
- Tracks sales transactions and their details (item-level).
- Facilitates reporting through derived views and materialized views.
- Normalizes entities such as `sales_detail` and `product_type` for data consistency.

---

## Database Entities and Normalization

### 1. `staff`
Represents employees and their assignments to locations.

| Column      | Data Type | Constraint       |
|-------------|-----------|------------------|
| staff_id    | SERIAL    | PRIMARY KEY      |
| first_name  | VARCHAR   | NOT NULL         |
| last_name   | VARCHAR   | NOT NULL         |
| position    | VARCHAR   |                  |
| start_date  | DATE      |                  |
| location_id | INT       | FK → `sales_outlet(location_id)` |

---

### 2. `sales_outlet`
Contains physical store and franchise details.

| Column       | Data Type | Constraint  |
|--------------|-----------|-------------|
| location_id  | SERIAL    | PRIMARY KEY |
| outlet_type  | VARCHAR   | NOT NULL    |
| address      | VARCHAR   |             |
| city         | VARCHAR   |             |
| telephone    | VARCHAR   |             |
| postal_code  | VARCHAR   |             |
| manager      | VARCHAR   |             |

---

### 3. `product_type` (Normalized)
Stores the distinct categories/types of products sold.

| Column            | Data Type | Constraint  |
|-------------------|-----------|-------------|
| product_type_id   | SERIAL    | PRIMARY KEY |
| product_type_name | VARCHAR   | NOT NULL    |

---

### 4. `product`
Contains individual product information, each linked to a type.

| Column           | Data Type | Constraint                          |
|------------------|-----------|-------------------------------------|
| product_id       | SERIAL    | PRIMARY KEY                         |
| product_name     | VARCHAR   | NOT NULL                            |
| description      | TEXT      |                                     |
| price            | NUMERIC   | NOT NULL                            |
| product_type_id  | INT       | FK → `product_type`                 |

---

### 5. `customer`
Tracks customer data for personalization and analytics.

| Column               | Data Type | Constraint  |
|----------------------|-----------|-------------|
| customer_id          | SERIAL    | PRIMARY KEY |
| customer_name        | VARCHAR   | NOT NULL    |
| customer_email       | VARCHAR   |             |
| customer_since       | DATE      |             |
| customer_card_number | VARCHAR   | UNIQUE      |
| birthdate            | DATE      |             |
| gender               | VARCHAR   |             |

---

### 6. `sales_transaction`
Represents header-level details of sales operations.

| Column           | Data Type | Constraint                          |
|------------------|-----------|-------------------------------------|
| transaction_id   | SERIAL    | PRIMARY KEY                         |
| transaction_date | DATE      | NOT NULL                            |
| transaction_time | TIME      |                                     |
| staff_id         | INT       | FK → `staff(staff_id)`              |
| customer_id      | INT       | FK → `customer(customer_id)`        |

---

### 7. `sales_detail` (Normalized)
Line-item level data representing individual items per transaction.

| Column           | Data Type | Constraint                              |
|------------------|-----------|-----------------------------------------|
| detail_id        | SERIAL    | PRIMARY KEY                             |
| transaction_id   | INT       | FK → `sales_transaction`                |
| product_id       | INT       | FK → `product(product_id)`              |
| quantity         | INT       | NOT NULL                                |
| unit_price       | NUMERIC   | NOT NULL                                |

---

## Views and Materialized Views

### `staff_locations_view`
Links staff with their assigned locations for HR and admin use.

```sql
CREATE OR REPLACE VIEW staff_locations_view AS
SELECT 
    s.staff_id,
    s.first_name || ' ' || s.last_name AS staff_name,
    so.location_id,
    so.city AS outlet_location
FROM staff s
JOIN sales_outlet so ON s.location_id = so.location_id;
