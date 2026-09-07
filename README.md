# Walmart Sales Data Analysis (SQL)
<img src="Walmart-PNG.png" width="300">
An exploratory data analysis (EDA) project on Walmart sales transaction data using MySQL. The project covers database setup, feature engineering, and a series of business questions across product, sales, and customer dimensions.

## 📁 Files

| File | Description |
|---|---|
| `Walmart_Sales_Data_csv.csv` | Raw transactional sales data (1,000 records) |
| `Walmart_SQL_Queries.sql` | Full SQL script: schema, data load, feature engineering, and analysis queries |

## 📊 Dataset

The dataset contains 1,000 sales transactions from Walmart branches in Myanmar, with the following fields:

| Column | Description |
|---|---|
| `invoice_id` | Unique transaction identifier |
| `branch` | Branch code (A, B, C) |
| `city` | City where the branch is located (Yangon, Naypyitaw, Mandalay) |
| `customer_type` | Member or Normal customer |
| `gender` | Customer gender |
| `product_line` | Product category purchased |
| `unit_price` | Price per unit ($) |
| `quantity` | Units purchased |
| `vat` | Tax amount (5%) |
| `total` | Total bill amount |
| `date` / `time` | Date and time of purchase |
| `payment` | Payment method (Cash, Credit card, Ewallet) |
| `cogs` | Cost of goods sold |
| `gross_margin_pct` | Gross margin percentage |
| `gross_income` | Gross income from the sale |
| `rating` | Customer satisfaction rating (1–10) |

## 🛠 Setup

1. Create the database and table, then load the CSV using `LOAD DATA LOCAL INFILE` (update the file path to match your local system, and enable `local_infile` on your MySQL server/client).
2. Run the **feature engineering** section to add derived columns:
   - `time_of_day` — Morning / Afternoon / Evening, derived from `time`
   - `day_name` — Day of week, derived from `date`
   - `month_name` — Month name, derived from `date`

> **Note:** The script also references a `product_category` column (Good/Bad, based on above/below-average `total`). The `UPDATE ... FROM` syntax used for this in the script is Postgres-style and is not valid in MySQL — see [Known Issues](#-known-issues--fixes) below for the corrected version.

## 🔍 Analysis Covered

**Generic**
- Distinct cities and branch-city mapping

**Product Analysis**
- Distinct product lines, most common payment method, best-selling and highest-revenue product lines
- Monthly revenue and COGS trends
- Highest-revenue city, product line with highest VAT
- Good vs. Bad product categorization (relative to average sales)
- Branches outperforming average quantity sold
- Most common product line by gender, average rating per product line

**Sales Analysis**
- Sales volume by time of day per weekday
- Revenue by customer type
- VAT by city and by customer type

**Customer Analysis**
- Unique customer types and payment methods
- Most common/most valuable customer type
- Gender distribution overall and by branch
- Ratings by time of day and day of week (overall and per branch, using both `GROUP BY` and window functions)

## ⚠️ Known Issues / Fixes

- **`product_category` update** — MySQL doesn't support `UPDATE ... SET ... FROM`. Use a subquery in the `CASE` (as already done for the correlated value) without the trailing `FROM sales`:
  ```sql
  UPDATE sales
  SET product_category = (
      CASE 
          WHEN total >= (SELECT avg_total FROM (SELECT AVG(total) AS avg_total FROM sales) t) THEN "Good"
          ELSE "Bad"
      END
  );
  ```
- **Customer type buying the most (Q4, Customer Analysis)** — `ORDER BY total_sales` should likely be `ORDER BY total_sales DESC` to correctly return the *highest*-spending customer type.
- **Time-of-day boundaries** — Afternoon starts at `12:01:00`, meaning any sale at exactly `12:00:00` falls into Morning; adjust boundaries if more precise cutoffs are needed.
- **File path** — The `LOAD DATA LOCAL INFILE` path is hardcoded to a local machine; update it to your own CSV path before running.

## ▶️ How to Run

1. Import the CSV into MySQL Workbench or your preferred client (enable `local_infile` if using `LOAD DATA`).
2. Execute `Walmart_SQL_Queries.sql` section by section: schema → data load → feature engineering → EDA queries.
3. Review query outputs to answer each business question, or adapt them into a BI dashboard (Power BI/Tableau) for visualization.

## 🧰 Tech Stack

- **Database:** MySQL
- **Data source:** Walmart Sales Dataset (Kaggle-style retail transaction data)
