
# Starbucks Sales Dashboard

An end-to-end data analytics project: raw CSV files → SQL Server database → Power BI dashboard.

<img width="1343" height="748" alt="Screenshot 2026-11-23 160304" src="https://github.com/user-attachments/assets/bcaa1e8b-863c-4341-b57f-406d816a471b" />



## Project Overview

The goal was to take raw sales data, load it into a properly designed relational database, clean it, and build an interactive dashboard that answers basic business questions:

- How much revenue, how many orders and how many customers do we have?
- At which hours of the day do sales peak?
- How do sales break down by item, payment method and customer type?

## Tools Used

| Area | Tool |
|---|---|
| Database | SQL Server Express, SQL Server Management Studio (SSMS) |
| Querying and cleaning | T-SQL |
| Visualization | Power BI Desktop |
| Calculations | DAX |
| Design | Figma (dashboard background) |

## Data

Three CSV files loaded into SQL Server:

| Table | Rows | Description |
|---|---|---|
| `customers` | 500 | Customer details (name, email, phone, age, gender) |
| `items` | 77 | Menu items with nutrition info (calories, fat, carb, fiber, protein, type) |
| `sales` | 10,000 | Sales transactions (store, datetime, customer, item, quantity, price, total amount, payment mode, customer type) |


## Database Design

`sales` is linked to `customers` and `items` with foreign keys.

```
customers (customer_id PK) ──┐
                             ├──< sales (sale_id PK, customer_id FK, item_id FK)
items (id PK) ───────────────┘
```

## Data Import and Cleaning

1. Created the `Starbucks` database and the three target tables with primary and foreign keys.
2. Imported each CSV into a **staging table** using SSMS *Import Flat File*, so raw data never touched the final tables directly.
3. Moved the data into the final tables with `INSERT ... SELECT`, using `TRY_CAST` to convert text columns to proper types (`INT`, `DECIMAL`, `DATETIME2`).
4. Validated the load: row counts per table, `NULL` checks on dates and amounts, and orphan-row checks against the foreign keys.
5. Dropped the staging tables.

### A data issue worth mentioning

`sales` has 10,000 rows but only **9,936 unique `transaction_id` values**, and the repeated IDs were not exact duplicates (checked with `SELECT DISTINCT`). So `transaction_id` could not be a primary key.

**Fix:** I redesigned the table with an auto-generated `sale_id` (`IDENTITY`) as the primary key, kept `transaction_id` as an indexed column, and used `DISTINCTCOUNT(transaction_id)` for order counts so orders are not over-counted.

## Power BI Dashboard

Connected Power BI to SQL Server and built a 3-page report:

| Page | Content |
|---|---|
| **Home** | Landing page with navigation |
| **Overview** | KPI cards (orders, total amount, customers, quantity) with trend lines, and hourly charts for amount, orders and quantity |
| **Detail** | Transaction-level table with item, customer type, payment mode, amount and quantity |

### Main measures

```DAX
Order Count = DISTINCTCOUNT(sales[transaction_id])
Total Amount = SUM(sales[total_amount])
Total Quantity = SUM(sales[quantity])
```

An hours table (0 to 23) is related to `sales` by hour so the charts display hours in the correct order.

## Key Insights

- **Total sales:** about 78.4K across 9,936 unique orders
- **Items sold:** about 16.5K units
- **Customers:** 500
- **Peak hour:** 9 AM is the highest hour for amount, orders and quantity. Sales stay strong until noon, then drop in the afternoon.

## Screenshots

| Home | Overview | Detail |
|---|---|---|
| <img width="1338" height="751" alt="Screenshot 2026-11-23 160208" src="https://github.com/user-attachments/assets/51104df7-ec05-4b5e-a3f6-228bc5b8b78e" />|<img width="1343" height="748" alt="Screenshot 2026-11-23 160304" src="https://github.com/user-attachments/assets/94a4b4a5-7439-4b92-9511-849a5e78b965" />|<img width="1456" height="815" alt="Screenshot 2026-11-23 160915" src="https://github.com/user-attachments/assets/accecd0d-9c0e-44c6-b3c8-128e56ea7d8e" />|
- Building DAX measures and a report with a clean data model

