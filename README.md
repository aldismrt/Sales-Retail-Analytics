# 🛍️ Retail Sales Analytics — Power BI Dashboard

An end-to-end retail sales analytics dashboard built in Power BI, covering data modeling, DAX measures, and interactive visualizations to uncover sales, profitability, and customer trends across regions and product categories.

<img width="1421" height="792" alt="Screenshot 2026-08-14 131938" src="https://github.com/user-attachments/assets/876fadae-cccf-4b5c-b115-8cad8aa81908" />

---

## 📌 Project Overview

This project analyzes retail transaction data to answer key business questions:

- How is revenue trending over time, and what's driving year-over-year growth?
- Which product categories and individual products generate the most sales and profit?
- How does performance vary across regions and stores?
- Who are the customers, and how does their behavior impact revenue?

The dashboard was built following an end-to-end BI workflow: data cleaning and feature engineering in Python, relational data modeling in Power BI, DAX measure development, and interactive report design.

---

## 🗂️ Dataset

The dataset is a retail sales dataset consisting of four related tables:

| Table | Description | Key Columns |
|---|---|---|
| **Transactions** | Fact table — one row per sale | TransactionID, Date, CustomerID, ProductID, StoreID, Quantity, Discount, PaymentMethod |
| **Customers** | Customer dimension | CustomerID, FirstName, LastName, Gender, BirthDate, City, JoinDate |
| **Products** | Product dimension | ProductID, ProductName, Category, SubCategory, UnitPrice, CostPrice |
| **Stores** | Store dimension | StoreID, StoreName, City, Region |

**Source:** [Retail Sales Dataset – Kaggle](https://www.kaggle.com/datasets/buharishehu/retail-sales-dataset)

---

## 🛠️ Tools & Skills Used

- **Power BI Desktop** — data modeling, DAX, report design
- **Power Query** — data transformation and cleaning
- **DAX (Data Analysis Expressions)** — calculated columns, measures, and time intelligence
- **Python (pandas)** — data enrichment and feature engineering prior to import

---

## 🧱 Data Model

The data model follows a **star schema**, with `Transactions` as the central fact table connected to four dimension tables via one-to-many relationships. A dedicated `DateTable` (calculated table) enables accurate time intelligence calculations (YoY growth, MTD, YTD).

<img width="1919" height="848" alt="Screenshot 2026-08-14 110301" src="https://github.com/user-attachments/assets/fc0cdce3-8bb5-4fc9-ae06-8d909f0609f3" />

**Relationships:**
- `Products[ProductID]` → `Transactions[ProductID]`
- `Customers[CustomerID]` → `Transactions[CustomerID]`
- `Stores[StoreID]` → `Transactions[StoreID]`
- `DateTable[Date]` → `Transactions[Date]`

---

## ⚙️ Data Preparation

Before importing into Power BI, the raw dataset was enriched using Python (pandas) to add derived fields:

- **Customers:** `Age`, `Tenure (Years)`, `FullName`
- **Products:** `Margin per Unit`, `Margin %`
- **Transactions:** `Sales`, `Cost`, `Profit`, `Year`, `Month`, `MonthNumber`, `Quarter`

This step demonstrates the ability to combine Python-based data engineering with Power BI's native modeling capabilities.

---

## 🧮 Key DAX Measures

A selection of core measures used throughout the report:

```dax
Total Sales = SUM(Transactions[Sales])

Total Profit = SUM(Transactions[Profit])

Profit Margin % = DIVIDE([Total Profit], [Total Sales])

Average Order Value = DIVIDE([Total Sales], [Total Transactions])

Sales LY = CALCULATE([Total Sales], SAMEPERIODLASTYEAR(DateTable[Date]))

YoY Growth % = DIVIDE([Total Sales] - [Sales LY], [Sales LY])

Product Sales Rank = RANKX(ALL(Products[ProductName]), [Total Sales], , DESC)
```

📄 Full list of measures available in [`DAX_Measures_Retail_Sales.md`](DAX_Measures_Retail_Sales.md).

---

## 📊 Dashboard Pages

### 1. Overview
KPI summary, yearly sales trend, sales by category and region, and top-performing products.

<img width="1421" height="792" alt="Screenshot 2026-08-14 131938" src="https://github.com/user-attachments/assets/fd06e3c3-19c9-4252-9f2f-2460d4b12ca6" />


### 2. Product Performance
Product-level breakdown by category and sub-category, a sales-vs-margin scatter plot to spot high-volume/low-margin outliers, and a sortable product table with profit margin.

<img width="1281" height="709" alt="Screenshot 2026-08-14 131705" src="https://github.com/user-attachments/assets/404bbce5-08a4-4795-b6a4-ef154d55fe5c" />


### 3. Customer Insights
Customer demographics (age distribution, gender split), sales by payment method, and a ranked table of top customers by total spend.

<img width="1428" height="789" alt="image" src="https://github.com/user-attachments/assets/7849cca5-cf88-4fff-b2d4-764f6498b8a9" />


---

## 💡 Key Insights

**Sales & Profitability**
- **Total revenue reached 14.30M** across the observed period, generating **3.83M in profit** (~26.8% overall margin).
- **2024 saw explosive growth of +222% YoY**, followed by a slight -1% dip in 2025 (partial-year comparison, not a true full-year decline).
- **Electronics (44%) and Fashion (44%)** together account for nearly 88% of total sales, while **Groceries contributes just 12%**.
- The **East region leads all others**, generating nearly double the sales of West, North, or South individually.

**Product Performance**
- **"Book Television," "And Footwear,"** and **"Set Dairy"** are the top 3 best-selling products by revenue.
- Margin varies significantly even among top sellers — **"Set Dairy" and "And Footwear" post the highest margins (~42%)**, while high-revenue **"Like Camera" trails at just 13.9% margin**, highlighting that sales volume and profitability don't always move together.
- Average order value sits at **~2.86K**, based on roughly **5,000 transactions**.

**Customer Behavior**
- The customer base of **200 individuals** spends an average of **~71.5K each**.
- Gender split leans slightly female (**56.5% female vs. 43.5% male**).
- Sales are fairly evenly distributed across payment methods (Cash, Credit Card, Mobile Payment, Bank Transfer), with **Cash and Credit Card leading at ~3.7M each**, suggesting no single payment channel dominates customer preference.

---

## 🚀 How to Use

1. Clone or download this repository.
2. Open `Retail Sales Analytics.pbix` in Power BI Desktop.
3. Data source path may need to be updated under **Transform Data → Data Source Settings** if the original Excel file location differs.
4. Explore the report pages, slicers, and drill-through interactions.

---

## 👤 Author

Built as part of a personal Power BI portfolio project to demonstrate end-to-end BI skills: data modeling, DAX, and dashboard design.

Feel free to connect or share feedback!
