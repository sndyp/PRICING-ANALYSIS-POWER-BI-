# Multi-Year Retail Sales & Profitability Analytics

## Project Overview
This project focuses on building a robust data pipeline and interactive dashboard to analyze sales performance over a three-year period (2023, 2024, and 2025). The project demonstrates advanced SQL techniques for ETL (Extract, Transform, Load) and Power BI for visualizing business KPIs such as total revenue, profit margins, and regional growth.

## Business Problem
The raw data was partitioned into yearly files with some inconsistencies in revenue reporting. The objective was to:
1. Consolidate sales records from multiple years.
2. Clean and handle missing revenue data.
3. Analyze profitability across different product categories and regions (East, West, South).

## Technical Stack
- **Database:** SQL Server (T-SQL)
- **Data Visualization:** Power BI
- **Tools:** Excel/CSV for raw data storage

## Data Architecture & SQL Implementation
The core of this project is a sophisticated SQL script that merges datasets and performs row-level calculations.

### Key SQL Features:
- **Table Consolidation:** Used `UNION ALL` to merge `Orders_2023`, `Orders_2024`, and `Orders_2025`.
- **Data Cleaning:** Implemented a logic to fill null revenue values by multiplying `Price * Quantity`.
- **Time Intelligence:** Calculated `WeekDate` to allow for granular weekly trend analysis.
- **Relational Joins:** Connected transactional data with `Customers` and `Products` tables to fetch regional and category metadata.

### The SQL Script:
```sql
-- Consolidating all orders from 2023 to 2025
with all_orders as (
    select OrderID, CustomerID, ProductID, OrderDate, Quantity, Revenue, COGS from Orders_2023
    union all
    select OrderID, CustomerID, ProductID, OrderDate, Quantity, Revenue, COGS from Orders_2024
    union all
    select OrderID, CustomerID, ProductID, OrderDate, Quantity, Revenue, COGS from Orders_2025
)

select
    a.OrderID,
    a.CustomerID,
    c.Region,
    a.ProductID,
    a.OrderDate,
    c.CustomerJoinDate,
    DATEADD(WEEK, DATEDIFF(WEEK, 0, a.OrderDate), 0) as WeekDate,
    a.Quantity,
    a.Revenue,
    -- Handling Null Revenue
    case 
        when a.Revenue is null then p.Price * a.Quantity 
        else a.Revenue 
    end as CleanedRevenue,
    a.Revenue - a.COGS as Profit,
    a.COGS,
    p.ProductName,
    p.ProductCategory,
    p.Price,
    p.Base_Cost
from all_orders a
left join customers c on a.CustomerID = c.CustomerID
left join products p on a.ProductID = p.ProductID
where a.CustomerID is not NULL;
