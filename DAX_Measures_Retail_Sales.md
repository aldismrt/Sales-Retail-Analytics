# DAX Measures — Retail Sales Portfolio (Power BI)

Full reference of DAX measures & calculated columns for the `retail_sales_dataset.xlsx` dataset
(tables: `Transactions`, `Customers`, `Products`, `Stores`).

---

## 0. Data Model Setup

Before creating measures, set up the following first:

1. **Create a Date table** (avoid using Power BI's built-in auto date/time):
```dax
DateTable = CALENDAR(MIN(Transactions[Date]), MAX(Transactions[Date]))
```
Then add supporting columns:
```dax
Year = YEAR(DateTable[Date])
Month = FORMAT(DateTable[Date], "MMM")
MonthNumber = MONTH(DateTable[Date])
Quarter = "Q" & FORMAT(DateTable[Date], "Q")
```

2. **Relationships**: connect `DateTable[Date]` → `Transactions[Date]`,
   `Customers[CustomerID]` → `Transactions[CustomerID]`,
   `Products[ProductID]` → `Transactions[ProductID]`,
   `Stores[StoreID]` → `Transactions[StoreID]` (all one-to-many, single direction).

3. Mark `DateTable` as a **Date Table** (Table tools → Mark as Date Table) so time intelligence functions work correctly.

---

## 1. Calculated Columns (base tables)

**In the `Customers` table:**
```dax
Age = DATEDIFF(Customers[BirthDate], TODAY(), YEAR)

Tenure (Years) = DATEDIFF(Customers[JoinDate], TODAY(), YEAR)

FullName = Customers[FirstName] & " " & Customers[LastName]
```

**In the `Products` table:**
```dax
Margin per Unit = Products[UnitPrice] - Products[CostPrice]

Margin % = DIVIDE(Products[Margin per Unit], Products[UnitPrice])
```

> **Note:** In the final implementation, the `Sales`, `Profit`, `Cost`, `Year`, `Month`, `MonthNumber`, `Quarter` columns in the `Transactions` table, as well as `Age`, `Tenure`, and `FullName` in `Customers`, were pre-calculated using Python (pandas) before the data was imported into Power BI. The measures below follow that approach (simple `SUM` over the pre-computed columns), but pure DAX formulas are also included as a reference/alternative.

---

## 2. Core Measures (Sales & Profit)

```dax
Total Sales = SUM(Transactions[Sales])

Total Cost = SUM(Transactions[Cost])

Total Profit = SUM(Transactions[Profit])

Profit Margin % = DIVIDE([Total Profit], [Total Sales])

Total Discount Given =
SUMX(
    Transactions,
    Transactions[Quantity] * RELATED(Products[UnitPrice]) * Transactions[Discount]
)

Total Quantity Sold = SUM(Transactions[Quantity])

Total Transactions = DISTINCTCOUNT(Transactions[TransactionID])

Average Order Value = DIVIDE([Total Sales], [Total Transactions])
```

**Alternative (if Sales/Profit aren't pre-calculated in the source data, pure DAX):**
```dax
Total Sales =
SUMX(
    Transactions,
    Transactions[Quantity]
    * RELATED(Products[UnitPrice])
    * (1 - Transactions[Discount])
)

Total Cost =
SUMX(
    Transactions,
    Transactions[Quantity] * RELATED(Products[CostPrice])
)

Total Profit = [Total Sales] - [Total Cost]
```

---

## 3. Customer Measures

```dax
Total Customers = DISTINCTCOUNT(Transactions[CustomerID])

Avg Customer Age = AVERAGE(Customers[Age])

Avg Spend per Customer = DIVIDE([Total Sales], [Total Customers])

Customer Total Spend = SUM(Transactions[Sales])

New Customers =
CALCULATE(
    DISTINCTCOUNT(Customers[CustomerID]),
    FILTER(
        Customers,
        YEAR(Customers[JoinDate]) = YEAR(MAX(DateTable[Date]))
    )
)

Repeat Customers =
CALCULATE(
    DISTINCTCOUNT(Transactions[CustomerID]),
    FILTER(
        VALUES(Transactions[CustomerID]),
        CALCULATE(DISTINCTCOUNT(Transactions[TransactionID])) > 1
    )
)

Repeat Purchase Rate = DIVIDE([Repeat Customers], [Total Customers])
```

---

## 4. Time Intelligence

```dax
Sales LY =
CALCULATE([Total Sales], SAMEPERIODLASTYEAR(DateTable[Date]))

YoY Growth % = DIVIDE([Total Sales] - [Sales LY], [Sales LY])

Sales MTD = TOTALMTD([Total Sales], DateTable[Date])

Sales YTD = TOTALYTD([Total Sales], DateTable[Date])

Sales Prev Month =
CALCULATE([Total Sales], DATEADD(DateTable[Date], -1, MONTH))

MoM Growth % = DIVIDE([Total Sales] - [Sales Prev Month], [Sales Prev Month])

Running Total Sales =
CALCULATE(
    [Total Sales],
    FILTER(
        ALLSELECTED(DateTable[Date]),
        DateTable[Date] <= MAX(DateTable[Date])
    )
)
```

> **Note:** `Sales LY` compares the exact same period (not a full calendar year) — if the current year's data only covers a partial period, `Sales LY` will automatically calculate the matching partial period from the prior year. This is expected DAX behavior, not an error.

---

## 5. Product & Ranking Measures

```dax
Product Sales Rank =
RANKX(ALL(Products[ProductName]), [Total Sales], , DESC)

Best Selling Product =
CALCULATE(
    VALUES(Products[ProductName]),
    TOPN(1, ALL(Products), [Total Sales], DESC)
)

% of Total Sales by Category =
DIVIDE([Total Sales], CALCULATE([Total Sales], ALL(Products[Category])))

Top 5 Products Sales =
CALCULATE(
    [Total Sales],
    TOPN(5, ALL(Products[ProductName]), [Total Sales], DESC)
)
```

---

## 6. Store & Region Measures

```dax
Store Sales Rank =
RANKX(ALL(Stores[StoreName]), [Total Sales], , DESC)

Sales per Region =
CALCULATE([Total Sales], ALLEXCEPT(Stores, Stores[Region]))

Best Performing Store =
CALCULATE(
    VALUES(Stores[StoreName]),
    TOPN(1, ALL(Stores), [Total Sales], DESC)
)
```

---

## 7. Payment Method Measures

```dax
Sales by Payment Method =
CALCULATE([Total Sales], ALLEXCEPT(Transactions, Transactions[PaymentMethod]))

% Transactions by Payment Method =
DIVIDE(
    [Total Transactions],
    CALCULATE([Total Transactions], ALL(Transactions[PaymentMethod]))
)
```

---

## Dashboard Pages Reference

| Page | Key Visuals |
|---|---|
| **1. Overview** | KPI cards (Total Sales, Profit, AOV, Total Transactions), Year-over-Year table, Sales by Region, Sales by Category (donut), Sales by Month (area chart), Top Products table |
| **2. Product Performance** | Sortable product table (Sales, Profit, Margin %), Sales by SubCategory, Sales by Category, Scatter chart (Sales vs Profit Margin %) |
| **3. Customer Insights** | KPI cards (Total Customers, Avg Age, Avg Spend), Age distribution, Gender split (donut), Sales by Payment Method, Top Customers table |

## Formatting Notes

- `Profit Margin %`, `YoY Growth %`, `MoM Growth %`, and other percentage measures need to be manually set to **Percentage** format (select the measure → Measure tools → Format → Percentage), since `DIVIDE()` returns a decimal value (0.19) rather than automatically formatting it as a percentage (19%).
- Use **slicers** (Year, Category, Region) instead of clicking directly on visuals like tables/charts for filtering, to avoid accidental filters caused by cross-filtering between visuals.
