# Navi Marketing & Credit Analytics Warehouse

End-to-end **Data Engineering project** inspired by Navi Technologies, built to showcase a modern analytics stack using **Databricks, PySpark, Delta Lake, and dimensional marts** for marketing & credit-risk teams.

The goal of this project is to simulate how a fintech company would:
- Ingest raw customer, event, and transaction data
- Build reliable **bronze / silver / gold** layers
- Expose **business-ready marts** for marketing and credit-risk analytics

---

## ✅ Tech Stack

- **Compute / ETL:** Databricks Community Edition (PySpark)
- **Storage Format:** Delta Lake tables (bronze, silver, gold)
- **Warehouse Layers:**
  - `navi_bronze` – raw but structured
  - `navi_silver` – cleaned, typed, conformed
  - `navi_gold` – dimensional marts for business teams
- **Version Control:** GitHub (this repo)

> Designed to be fully **online-only**: no local Python or installs required.

---

## 📊 Data Model (Source Layer)

Three core source datasets (CSV → Databricks tables):

### 1. `customers`
Basic customer profile and acquisition info.

| Column              | Type    | Description                               |
|---------------------|---------|-------------------------------------------|
| customer_id         | string  | Unique customer key                       |
| first_name          | string  | First name                                |
| last_name           | string  | Last name                                 |
| email               | string  | Contact email                             |
| signup_date         | date    | Date the user signed up                   |
| country             | string  | Country                                   |
| risk_segment        | string  | Risk band (low / medium / high)          |
| acquisition_channel | string  | Channel: paid_search, facebook_ads, etc. |

### 2. `events`
Product/app behavioral events.

| Column          | Type      | Description                        |
|-----------------|-----------|------------------------------------|
| event_id        | string    | Unique event key                   |
| customer_id     | string    | Who performed the event            |
| event_timestamp | timestamp | When it happened                   |
| event_type      | string    | page_view, signup_started, etc.    |
| platform        | string    | android / ios / web                |
| campaign_id     | string    | Optional campaign id               |
| session_id      | string    | Session identifier                 |

### 3. `transactions`
Financial transactions used for credit risk.

| Column               | Type      | Description                        |
|----------------------|-----------|------------------------------------|
| transaction_id       | string    | Unique transaction key             |
| customer_id          | string    | Who made the transaction           |
| transaction_timestamp| timestamp | When it occurred                   |
| amount               | double    | Transaction amount                 |
| currency             | string    | Currency (e.g., USD)               |
| merchant_category    | string    | groceries, travel, etc.            |
| is_chargeback        | boolean   | Whether the txn was charged back   |
| is_late_payment      | boolean   | Whether payment was late           |
| loan_id              | string    | Optional loan identifier           |

---

## 🏗 Lakehouse Architecture

### 🔹 Bronze Layer – `navi_bronze`
- Direct copies of uploaded raw tables (structured but not cleaned).
- Created via notebook: `01_bronze_ingest`
- Tables:
  - `bronze_customers`
  - `bronze_events`
  - `bronze_transactions`

### 🔸 Silver Layer – `navi_silver`
Cleaned, typed, and lightly modeled:

- `silver_customers`
  - Standardized column names (`firstName`, `lastName`, `acquisitionChannel`)
  - `signup_date` cast to proper `date` type
- `silver_transactions`
  - Typed timestamps (`transaction_timestamp`)
  - Cast `amount` to `double`
  - Convert `is_chargeback` and `is_late_payment` to `boolean`
- `silver_customer_activity`
  - Aggregated user behavior from events:
    - `total_events`
    - `unique_event_types`
    - `total_sessions`

Created via notebook: `02_silver_transform`.

### 🟡 Gold Layer – `navi_gold`
Business-facing marts for analytics:

#### `fct_marketing_performance`
Grain: **acquisition_channel**

Aggregates:
- `num_customers`
- `total_spend`
- `avg_spend`
- `total_events`
- `total_sessions`
- `total_chargebacks`
- `total_late_payments`

Used by **marketing** to compare quality of different acquisition channels (e.g., “Are paid_search customers riskier than referrals?”).

#### `fct_credit_risk`
Grain: **customer_id**

Features:
- `total_spend`
- `num_transactions`
- `late_payments`
- `chargebacks`
- `total_events`
- `total_sessions`
- Basic `risk_score` based on:
  - chargebacks
  - late payments
  - low total spend

Used by **credit-risk** teams to segment customers into different risk bands.

Both marts are created via notebook: `03_gold_marts`.

---

## ▶️ How to Run This Project (Databricks Community Edition)

1. **Upload CSVs**
   - In Databricks, go to **Data → Add → Upload File**
   - Upload `customers.csv`, `events.csv`, `transactions.csv`
   - Store them as tables in a database such as `navi_raw`.

2. **Run Bronze Notebook**
   - Open `01_bronze_ingest`
   - Creates `navi_bronze` and bronze tables.

3. **Run Silver Notebook**
   - Open `02_silver_transform`
   - Creates `navi_silver`:
     - `silver_customers`
     - `silver_transactions`
     - `silver_customer_activity`

4. **Run Gold Notebook**
   - Open `03_gold_marts`
   - Creates `navi_gold`:
     - `fct_marketing_performance`
     - `fct_credit_risk`

5. **Query Examples**
   ```sql
   USE navi_gold;

   -- Top channels by total spend
   SELECT acquisitionChannel, total_spend, total_chargebacks
   FROM fct_marketing_performance
   ORDER BY total_spend DESC;

   -- Highest-risk customers
   SELECT customer_id, risk_score, late_payments, chargebacks, total_spend
   FROM fct_credit_risk
   ORDER BY risk_score DESC;
