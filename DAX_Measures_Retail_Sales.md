# DAX Measures — Retail Sales Portfolio (Power BI)

Referensi lengkap DAX measure & calculated column untuk dataset `retail_sales_dataset.xlsx`
(tabel: `Transactions`, `Customers`, `Products`, `Stores`).

---

## 0. Persiapan Data Model

Sebelum bikin measure, susun dulu:

1. **Buat Date table** (jangan pakai auto date/time bawaan Power BI):
```dax
DateTable = CALENDAR(MIN(Transactions[Date]), MAX(Transactions[Date]))
```
Lalu tambahkan kolom pendukung:
```dax
Year = YEAR(DateTable[Date])
Month = FORMAT(DateTable[Date], "MMM")
MonthNumber = MONTH(DateTable[Date])
Quarter = "Q" & FORMAT(DateTable[Date], "Q")
```

2. **Relationship**: hubungkan `DateTable[Date]` → `Transactions[Date]`,
   `Customers[CustomerID]` → `Transactions[CustomerID]`,
   `Products[ProductID]` → `Transactions[ProductID]`,
   `Stores[StoreID]` → `Transactions[StoreID]` (semua one-to-many, single direction).

3. Tandai `DateTable` sebagai **Date Table** (Table tools → Mark as Date Table) supaya time intelligence berfungsi.

---

## 1. Calculated Column (di tabel dasar)

**Di tabel `Customers`:**
```dax
Age = DATEDIFF(Customers[BirthDate], TODAY(), YEAR)

Tenure (Years) = DATEDIFF(Customers[JoinDate], TODAY(), YEAR)

FullName = Customers[FirstName] & " " & Customers[LastName]
```

**Di tabel `Products`:**
```dax
Margin per Unit = Products[UnitPrice] - Products[CostPrice]

Margin % = DIVIDE(Products[Margin per Unit], Products[UnitPrice])
```

> **Catatan:** Pada implementasi final, kolom `Sales`, `Profit`, `Cost`, `Year`, `Month`, `MonthNumber`, `Quarter` di tabel `Transactions`, serta `Age`, `Tenure`, `FullName` di `Customers` sudah dihitung lebih awal menggunakan Python (pandas) sebelum data diimport ke Power BI. Measure di bawah ini disusun mengikuti pendekatan tersebut (langsung `SUM` dari kolom hasil olahan), namun formula DAX murni tetap disertakan sebagai referensi/alternatif.

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

**Alternatif (jika Sales/Profit belum dihitung di sumber data, murni DAX):**
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

> **Catatan:** `Sales LY` membandingkan periode yang sama persis (bukan full year) — jika data tahun berjalan baru mencakup sebagian bulan, `Sales LY` juga otomatis hanya menghitung periode yang sama di tahun sebelumnya. Ini perilaku standar dan bukan error.

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

| Halaman | Visual Utama |
|---|---|
| **1. Overview** | KPI cards (Total Sales, Profit, AOV, Total Transactions), Year-over-Year table, Sales by Region, Sales by Category (donut), Sales by Month (area chart), Top Products table |
| **2. Product Performance** | Sortable product table (Sales, Profit, Margin %), Sales by SubCategory, Sales by Category, Scatter chart (Sales vs Profit Margin %) |
| **3. Customer Insights** | KPI cards (Total Customers, Avg Age, Avg Spend), Age distribution, Gender split (donut), Sales by Payment Method, Top Customers table |

## Formatting Notes

- `Profit Margin %`, `YoY Growth %`, `MoM Growth %`, dan measure persentase lainnya perlu di-set format **Percentage** secara manual (klik measure → Measure tools → Format → Percentage), karena `DIVIDE()` menghasilkan angka desimal (0.19) bukan otomatis jadi persen (19%).
- Gunakan **slicer** (Year, Category, Region) alih-alih klik langsung ke visual seperti tabel/chart untuk filtering, supaya menghindari filter "tidak sengaja" akibat cross-filtering antar visual.
