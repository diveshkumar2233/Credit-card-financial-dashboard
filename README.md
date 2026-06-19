# 💳 Credit Card Financial Dashboard

An end-to-end **Power BI** analytics project that turns raw credit card transaction and customer data into a two-page executive dashboard — helping a bank's management track revenue, transaction behavior, customer profiles, and acquisition cost across **5.6M+ customers / 657K transactions**.

![Status](https://img.shields.io/badge/status-complete-brightgreen)
![Tool](https://img.shields.io/badge/tool-Power%20BI-yellow)
![Database](https://img.shields.io/badge/database-PostgreSQL-blue)

---

## 📌 Table of Contents

- [Project Overview](#-project-overview)
- [Business Problem](#-business-problem)
- [Problems Identified in the Data / Dashboard](#-problems-identified-in-the-data--dashboard)
- [Solutions Implemented](#-solutions-implemented)
- [Tech Stack](#-tech-stack)
- [Dataset](#-dataset)
- [Database Schema](#-database-schema)
- [Project Architecture / Workflow](#-project-architecture--workflow)
- [How It Works (Step-by-Step Setup)](#-how-it-works-step-by-step-setup)
- [Dashboard Pages](#-dashboard-pages)
- [Key Insights](#-key-insights)
- [KPIs Tracked](#-kpis-tracked)
- [Repository Structure](#-repository-structure)
- [Future Improvements](#-future-improvements)
- [Author](#-author)

---

## 🧭 Project Overview

A bank wants to understand how its **credit card customers behave**, where **revenue comes from**, and which **customer segments and card categories** are most/least profitable. Raw data was sitting in CSV files with no easy way to slice it by quarter, card type, demographics, or spend category.

This project:
1. Loads raw customer & transaction CSVs into a **PostgreSQL** database via SQL scripts.
2. Models and cleans the data.
3. Builds a **2-page interactive Power BI dashboard** (Customer Report + Transaction Report) with cross-filtering, slicers, and KPI cards.

---

## 🎯 Business Problem

Management needed answers to questions such as:

- Which **card category** (Blue/Silver/Gold/Platinum) generates the most revenue vs. cost to acquire?
- Which **customer segments** (job, education, income, age, gender) drive the most revenue?
- Is revenue **trending up or down** quarter over quarter?
- Which **expenditure categories** and **transaction channels** (Swipe/Chip/Online) bring in the most revenue?
- Are **acquisition costs** justified by the revenue each card tier brings in?
- How satisfied are customers (CSS – Customer Satisfaction Score) and does that correlate with revenue?

Without a centralized dashboard, these questions required manual spreadsheet pulls every time — slow, error-prone, and not interactive.

---

## ⚠️ Problems Identified in the Data / Dashboard

| # | Problem | Impact |
|---|----------|--------|
| 1 | **Date import errors** — PostgreSQL `COPY` failed with `date/time field value out of range: "0"` because the CSV dates weren't in `YYYY-MM-DD` ISO format. | Blocked data load entirely. |
| 2 | **Two disconnected datasets** (`cc_detail` and `cust_detail`) — transactions and customer demographics lived in separate tables/files with only `Client_Num` in common. | Without a proper relationship, you can't slice revenue by customer attributes (income, education, job, etc.). |
| 3 | **Missing Week 53 data** — the original yearly extract was incomplete and needed a second `COPY` for an additional batch (`cc_add.csv`, `cust_add.csv`). | Risk of under-reported revenue/transactions if not merged correctly. |
| 4 | **High acquisition cost concentrated in one tier** — Blue cards alone cost **0.89M** to acquire vs. **0.01M–0.06M** for Gold/Platinum/Silver, yet Blue is the highest-volume, lower-margin tier. | Indicates inefficient spend on the lowest-tier segment relative to ROI from premium tiers. |
| 5 | **Revenue volatility** — weekly revenue (Revenue vs Gender chart) swings between ~0.36M and ~0.68M with no clear seasonal trend surfaced. | Hard for stakeholders to tell if dips are seasonal, a data issue, or an actual business problem. |
| 6 | **No single source of truth for filters** — slicers for Card Category, Income Group, Marital Status, etc. existed but weren't connected to one unified data model. | Cross-filtering between Customer Report and Transaction Report pages wasn't seamless. |
| 7 | **Channel imbalance** — Online transactions contribute only **3M** in revenue vs. **35M** from Swipe, despite the growing shift to digital payments industry-wide. | Possible missed opportunity in digital/online card usage growth. |

---

## ✅ Solutions Implemented

| Problem | Solution |
|----------|-----------|
| Date format errors on `COPY` | Set session datestyle explicitly: `SET datestyle TO 'ISO, DMY';` and validated/cleaned date columns in CSVs before import. |
| Disconnected tables | Created a **1-to-many relationship** in Power BI's data model on `Client_Num` between `cc_detail` (transaction-level, many rows per customer) and `cust_detail` (one row per customer). |
| Missing week-53 data | Used a second `COPY` command (`cc_add.csv` / `cust_add.csv`) into the **same tables** to append the missing batch instead of creating duplicate/parallel tables. |
| Acquisition cost vs. revenue mismatch | Surfaced **Customer Acq Cost by Card_Category** as its own visual so stakeholders can directly compare it next to **Revenue by Card_Category** and **Interest Earned** — making the ROI gap visible and actionable. |
| Revenue volatility | Added a **time-series line chart** (Revenue vs Gender, weekly) plus **Qtr Revenue & Transaction Count** combo chart so trend vs. noise is easy to separate at a glance. |
| Unified filtering | Built both report pages on the **same underlying model** with synced slicers (Week_Start_Date, Income Group, Card_Category) so filtering one page's context is consistent with the other. |
| Channel imbalance visibility | Added **Revenue by Use Chip** breakdown (Swipe/Chip/Online) so the digital-channel gap is a visible, trackable KPI rather than hidden in raw transactions. |

---

## 🛠 Tech Stack

| Layer | Tool |
|-------|------|
| Data Storage | PostgreSQL |
| Data Loading | SQL (`CREATE TABLE`, `COPY ... CSV HEADER`) |
| Data Modeling & Visualization | Power BI Desktop |
| Relationships | Power BI Data Model (Client_Num key) |
| Version Control | Git & GitHub |

---

## 📂 Dataset

Two core entities, joined on `Client_Num`:

### 1. `cc_detail` — Credit Card Transaction Data
Weekly transaction-level records per customer: card category, fees, credit limit, revolving balance, transaction amount/count, utilization ratio, spend channel (`Use_Chip`), expenditure type, interest earned, delinquency flag.

### 2. `cust_detail` — Customer Demographic Data
One row per customer: age, gender, dependents, education, marital status, state, zip, car/house ownership, personal loan flag, job, income, customer satisfaction score (CSS).

> 📁 Raw CSVs are not included in this repo for size/privacy reasons. See [Repository Structure](#-repository-structure) for where to place them locally.

---

## 🗄 Database Schema

```sql
CREATE DATABASE ccdb;

CREATE TABLE cc_detail (
    Client_Num INT,
    Card_Category VARCHAR(20),
    Annual_Fees INT,
    Activation_30_Days INT,
    Customer_Acq_Cost INT,
    Week_Start_Date DATE,
    Week_Num VARCHAR(20),
    Qtr VARCHAR(10),
    current_year INT,
    Credit_Limit DECIMAL(10,2),
    Total_Revolving_Bal INT,
    Total_Trans_Amt INT,
    Total_Trans_Ct INT,
    Avg_Utilization_Ratio DECIMAL(10,3),
    Use_Chip VARCHAR(10),
    Exp_Type VARCHAR(50),
    Interest_Earned DECIMAL(10,3),
    Delinquent_Acc VARCHAR(5)
);

CREATE TABLE cust_detail (
    Client_Num INT,
    Customer_Age INT,
    Gender VARCHAR(5),
    Dependent_Count INT,
    Education_Level VARCHAR(50),
    Marital_Status VARCHAR(20),
    State_cd VARCHAR(50),
    Zipcode VARCHAR(20),
    Car_Owner VARCHAR(5),
    House_Owner VARCHAR(5),
    Personal_Loan VARCHAR(5),
    Contact VARCHAR(50),
    Customer_Job VARCHAR(50),
    Income INT,
    Cust_Satisfaction_Score INT
);
```

**Relationship:** `cc_detail.Client_Num` → `cust_detail.Client_Num` (many-to-one)

---

## 🏗 Project Architecture / Workflow

```
 CSV Files (credit_card.csv, customer.csv, cc_add.csv, cust_add.csv)
              │
              ▼
   PostgreSQL Database (ccdb)
   ├─ cc_detail   (transactions)
   └─ cust_detail (customers)
              │
              ▼
   Power BI Desktop — Get Data (PostgreSQL connector)
              │
              ▼
   Data Modeling (relationship on Client_Num, calculated columns/measures)
              │
              ▼
   Dashboard Build (2 pages, slicers, KPI cards, charts)
              │
              ▼
   Published / Shared Report (.pbix)
```

---

## ⚙️ How It Works (Step-by-Step Setup)

### Step 1 — Create the database and tables
Run the provided [`SQL_Query_-_Financial_Dashboard_Data.sql`](./SQL_Query_-_Financial_Dashboard_Data.sql) script:
```sql
CREATE DATABASE ccdb;
-- then run the CREATE TABLE statements for cc_detail and cust_detail
```

### Step 2 — Fix date formatting (important!)
If you hit `ERROR: date/time field value out of range: "0"`, either:
- Clean the date columns in the CSV to strict `YYYY-MM-DD` format, **or**
- Set the session datestyle before importing:
```sql
SET datestyle TO 'ISO, DMY';
```

### Step 3 — Load the base data
```sql
COPY cc_detail   FROM 'D:\credit_card.csv' DELIMITER ',' CSV HEADER;
COPY cust_detail FROM 'D:\customer.csv'    DELIMITER ',' CSV HEADER;
```

### Step 4 — Load the supplementary (Week-53) data
```sql
COPY cc_detail   FROM 'D:\cc_add.csv'   DELIMITER ',' CSV HEADER;
COPY cust_detail FROM 'D:\cust_add.csv' DELIMITER ',' CSV HEADER;
```
> Update the file paths to match your local machine before running.

### Step 5 — Connect Power BI to PostgreSQL
1. Open Power BI Desktop → **Get Data** → **PostgreSQL database**.
2. Enter server/database (`ccdb`) credentials.
3. Load `cc_detail` and `cust_detail`.

### Step 6 — Build the data model
- Go to **Model view**.
- Create a relationship: `cust_detail[Client_Num]` (1) → `cc_detail[Client_Num]` (*).
- Add calculated columns as needed: `Age Group`, `Income Group` (Low/Mid/High), `Salary Group`.

### Step 7 — Build the visuals
- Page 1: **Credit Card Customer Report** (demographics-focused).
- Page 2: **Credit Card Transaction Report** (revenue/spend-focused).
- Add slicers for `Week_Start_Date`, `Income Group`, `Card_Category`, `Gender`, `Qtr`.
- Add KPI cards for Total Revenue, Total Interest, Total Income, CSS.

### Step 8 — Publish / Share
Save as `.pbix` and publish to Power BI Service, or export to PDF for sharing (as included in this repo).

---

## 📊 Dashboard Pages

### Page 1 — Credit Card Customer Report
![Customer Report](./Credit_Card_Financial_Dashboard-Customer.pdf)

**KPIs:** Total Revenue `55.4M` · Total Interest `7.9M` · Total Income `577M` · CSS `3.19`

**Visuals:**
- Revenue vs Gender (weekly trend line)
- Age Group breakdown (20–30, 30–40, …, 60+)
- Top 5 States by revenue (TX, NY, CA, FL, NJ)
- Salary Group (High/Mid/Low)
- Dependent Count distribution
- Marital Status (Married/Single/Unknown)
- Education Level breakdown
- Customer Job vs Revenue/Transaction Amount/Income table

**Slicers:** Week_Start_Date, Income Group (Low/Mid/High), Card_Category (Platinum/Gold/Silver/Blue), Qtr (Q1–Q4)

### Page 2 — Credit Card Transaction Report
![Transaction Report](./Credit_Card_Financial_Dashboard-Transaction.pdf)

**KPIs:** Total Revenue `55.4M` · Total Interest `7.9M` · Transaction Amount `45M` · Transaction Count `657K`

**Visuals:**
- Card_Category table: Revenue, Interest Earned, Annual Fees
- Qtr Revenue & Transaction Count (combo chart)
- Revenue by Expenditure Type (Bills, Entertainment, Fuel, Grocery, Food, Travel)
- Revenue by Education Level
- Revenue by Customer Job
- Revenue by Use Chip (Swipe/Chip/Online)
- Customer Acquisition Cost by Card_Category

**Slicers:** Week_Start_Date, Gender, Income Group, Card_Category, Qtr

---

## 💡 Key Insights

- **Blue cards dominate volume** but carry by far the highest acquisition cost (`0.89M`) — disproportionate to their per-card revenue contribution compared to Platinum/Gold.
- **Self-employed and Businessman segments** are the top two revenue-generating customer jobs (`14M` and `11M` respectively).
- **Swipe transactions (35M)** vastly outperform **Online (3M)** — a potential growth lever if the bank pushes more customers toward digital/online card usage.
- **Graduates** generate the highest revenue by education level (`22M`), nearly double the next segment.
- **Texas (TX)** leads all states in revenue contribution among the top 5.
- Revenue is **fairly stable quarter-over-quarter** (`13.4M`–`14.2M`) despite weekly volatility — meaning the swings are short-term noise, not a structural decline.
- **Female customers (29.6M)** contribute slightly more total revenue than male customers (25.8M).

---

## 📈 KPIs Tracked

| KPI | Value |
|-----|-------|
| Total Revenue | 55.4M |
| Total Interest Earned | 7.9M |
| Total Income | 577M |
| Customer Satisfaction Score (CSS) | 3.19 |
| Transaction Amount | 45M |
| Transaction Count | 657K |

---

## 📁 Repository Structure

```
credit-card-financial-dashboard/
│
├── README.md
├── SQL_Query_-_Financial_Dashboard_Data.sql
├── Credit_Card_Dashboard.pbix
├── Credit_Card_Financial_Dashboard-Customer.pdf
├── Credit_Card_Financial_Dashboard-Transaction.pdf
└── data/
    ├── credit_card.csv      (not included — add your own)
    ├── customer.csv         (not included — add your own)
    ├── cc_add.csv           (not included — add your own)
    └── cust_add.csv         (not included — add your own)
```

---

## 🚀 Future Improvements

- Automate the CSV → PostgreSQL load with a Python/Airflow pipeline instead of manual `COPY` commands.
- Add a **delinquency risk** page using the `Delinquent_Acc` field to flag at-risk customers.
- Build **DAX-based YoY/QoQ growth measures** for revenue and transaction count.
- Add **row-level security (RLS)** in Power BI Service for region-based access control.
- Investigate the **Online channel revenue gap** with a dedicated digital-adoption funnel analysis.

---

## 👤 Author

**Project type:** Personal/portfolio data analytics project
**Tools used:** PostgreSQL, Power BI Desktop
**Contact:** Add your name, LinkedIn, and portfolio link here.

---

