# SQL for Data Analysis – Car Prices Dataset

## 🎯 Objective
Analyze car sales data using **SQLite3** and demonstrate core SQL data analysis techniques, including:
- Data selection and filtering
- Aggregation and grouping
- Table joins
- Subqueries
- Views for reusable analysis
- Query optimization using indexes

---

## 🧰 Tools Used
- **SQLite3** (for database and queries)
- **DB Browser for SQLite** or SQLite command-line shell
- **Dataset:** `car_prices.csv` (converted into `car_prices.db`)

---

## 📁 Files Included
| File | Description |
|------|--------------|
| `car_prices.csv` | Original dataset |
| `car_prices.db` | SQLite database created from the CSV |
| `task4_car_analysis.sql` | SQL script with all required queries |
| `README.md` | This documentation file |
| *(Optional)* Screenshots | Query outputs for report submission |

---

## 🧩 Database Schema
The main table in the database is `car_prices` with the following columns:

| Column | Type | Description |
|---------|------|-------------|
| `year` | INTEGER | Model year of the car |
| `make` | TEXT | Manufacturer (e.g., Toyota, BMW) |
| `model` | TEXT | Model name |
| `trim` | TEXT | Trim level |
| `body` | TEXT | Body type (SUV, Sedan, etc.) |
| `transmission` | TEXT | Transmission type |
| `vin` | TEXT | Vehicle Identification Number |
| `state` | TEXT | State of sale |
| `condition` | REAL | Condition rating |
| `odometer` | REAL | Mileage reading |
| `color` | TEXT | Exterior color |
| `interior` | TEXT | Interior color/material |
| `seller` | TEXT | Dealer or seller name |
| `mmr` | REAL | Market value estimate |
| `sellingprice` | REAL | Actual selling price |
| `saledate` | TEXT | Sale date |

---

## 🧠 SQL Queries Overview

### 1. Basic Analysis
- `SELECT`, `WHERE`, `ORDER BY`, `GROUP BY`
- Example: Find average selling price by car make.

### 2. Joins
- Created helper table `car_specs`
- Performed `INNER JOIN` and `LEFT JOIN` to combine data.

### 3. Subqueries
- Found cars above average selling price.
- Retrieved highest-priced car per make.

### 4. Aggregate Functions
- `SUM`, `AVG`, `COUNT` to summarize car prices and odometer readings.

### 5. Views
- Created view `v_make_year_summary` summarizing average price per make and year.
- Accessed using:
  ```sql
  SELECT * FROM v_make_year_summary;
6. Index Optimization

Created indexes to speed up queries:

idx_make, idx_price, idx_year, idx_state

Verified using:

PRAGMA index_list('car_prices');

▶️ How to Run

Open the SQLite database:

sqlite3 car_prices.db


Load and execute the SQL script:

.read task4_car_analysis.sql


To verify created views and indexes:

SELECT name, type FROM sqlite_master WHERE type='view';
PRAGMA index_list('car_prices');


To explore data:

SELECT * FROM car_prices LIMIT 10;

📊 Example Outputs
Query	Description	Example Output
Average selling price per make	SELECT make, AVG(sellingprice)	Toyota – 21000.45
Cars above average price	WHERE sellingprice > (SELECT AVG(...))	High-end models
View query	SELECT * FROM v_make_year_summary	Summary table by make/year

-- ==========================================================
-- Task 4: SQL for Data Analysis (SQLite3)
-- Dataset: car_prices
-- ==========================================================

-- a. SELECT, WHERE, ORDER BY, GROUP BY
-------------------------------------------------------------

-- 1. View first 10 records
SELECT * 
FROM car_prices
LIMIT 10;

-- 2. Cars sold for more than $20,000
SELECT year, make, model, sellingprice
FROM car_prices
WHERE sellingprice > 20000
ORDER BY sellingprice DESC
LIMIT 10;

-- 3. Average selling price per car make
SELECT make, ROUND(AVG(sellingprice), 2) AS avg_price
FROM car_prices
GROUP BY make
ORDER BY avg_price DESC;

-- 4. Average selling price by state and make
SELECT state, make, ROUND(AVG(sellingprice), 2) AS avg_price
FROM car_prices
GROUP BY state, make
ORDER BY avg_price DESC
LIMIT 10;

-- b. JOINS (INNER, LEFT)
-------------------------------------------------------------
-- Create a helper table for join demonstration
CREATE TABLE IF NOT EXISTS car_specs AS
SELECT DISTINCT make, body, transmission
FROM car_prices;

-- 5. INNER JOIN: Match cars with their specifications
SELECT c.make, c.model, c.year, s.body, s.transmission, c.sellingprice
FROM car_prices AS c
INNER JOIN car_specs AS s
ON c.make = s.make
LIMIT 10;

-- 6. LEFT JOIN: Include all cars even if missing details
SELECT c.make, c.model, s.body, s.transmission
FROM car_prices AS c
LEFT JOIN car_specs AS s
ON c.make = s.make
LIMIT 10;

-- Note: SQLite doesn’t directly support RIGHT JOIN.
-- To simulate, swap the table order in LEFT JOIN.

-- c. Subqueries
-------------------------------------------------------------

-- 7. Cars priced above the average selling price
SELECT make, model, sellingprice
FROM car_prices
WHERE sellingprice > (SELECT AVG(sellingprice) FROM car_prices)
ORDER BY sellingprice DESC
LIMIT 10;

-- 8. Cars with highest selling price for each make
SELECT make, model, sellingprice
FROM car_prices
WHERE (make, sellingprice) IN (
    SELECT make, MAX(sellingprice)
    FROM car_prices
    GROUP BY make
);

-- d. Aggregate functions (SUM, AVG, COUNT)
-------------------------------------------------------------

-- 9. Total and average selling price of all cars
SELECT 
    COUNT(*) AS total_cars,
    ROUND(SUM(sellingprice), 2) AS total_sales,
    ROUND(AVG(sellingprice), 2) AS avg_price
FROM car_prices;

-- 10. Average odometer reading by car body type
SELECT body, ROUND(AVG(odometer), 2) AS avg_odometer
FROM car_prices
GROUP BY body
ORDER BY avg_odometer ASC;

-- e. Create views for analysis
-------------------------------------------------------------

-- 11. Create a view summarizing prices by make and year
CREATE VIEW IF NOT EXISTS v_make_year_summary AS
SELECT make, year, COUNT(*) AS num_sales, ROUND(AVG(sellingprice), 2) AS avg_price
FROM car_prices
GROUP BY make, year;

-- 12. Query the view
SELECT * FROM v_make_year_summary
ORDER BY avg_price DESC
LIMIT 10;

-- f. Optimize queries with indexes
-------------------------------------------------------------

-- 13. Create indexes to speed up common queries
CREATE INDEX IF NOT EXISTS idx_make ON car_prices(make);
CREATE INDEX IF NOT EXISTS idx_price ON car_prices(sellingprice);
CREATE INDEX IF NOT EXISTS idx_year ON car_prices(year);
CREATE INDEX IF NOT EXISTS idx_state ON car_prices(state);

-- 14. Check the created indexes
PRAGMA index_list('car_prices');

-- ==========================================================
-- End of Task 4 SQL Script
-- ==========================================================
