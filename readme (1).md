# Analysing the Sizwe Coffee Business: Year-End Review (FY2024 – FY2025)

**Course:** SQL Fundamentals (Witle Plus / Witle Technologies)  
**Author:** Junior Data Analyst  
**Environment & Dialect:** Databricks SQL   
**Database:** Sizwe Coffee Dataset (`Sizwe_Coffee_Dataset_1`)  

## 1. Project Overview

### Business Context

**Sizwe Coffee (Pty) Ltd** is a South African coffee roastery and wholesale supplier headquartered in Isando, Johannesburg, with a secondary distribution centre in Epping, Cape Town. The company roasts green beans and sells to 1,400 business customers across South Africa—grocery chains, restaurant groups, independent cafés, hotel groups, and corporate offices. Sizwe Coffee does not own retail storefronts.The purpose of this project is to analyse the company's sales performance, profitability, customer accounts, invoice collections and sales representative performance to support business decision-making.

Across the two financial years covered in the dataset (**1 July 2023 to 30 June 2025**), the company achieved strong top-line growth:

* **Total Revenue:** R441.00 million in FY2024, rising to R480.00 million in FY2025 (+8.84% year-on-year growth).
* **Channels:** Wholesale accounts for ~70% of total revenue (R643.59m across both years); Subscription accounts for ~30% (R277.41m across both years).
* **Workforce:** 320 staff, including 18 regional sales representatives.

Ahead of the FY2025 year-end review meeting, **Nomsa Dlamini** (Financial Director) and **Bongani Twala** (National Sales Manager) commissioned eight SQL investigations:

1. **Task 1 — Channel Performance:** Revenue split by channel for FY2024 and FY2025 side by side, with the growth rate between them.
2. **Task 2 — Where the Profit Actually Comes From:** Revenue, cost of goods (`cogs`), gross profit, and gross margin percentage by product line, sorted by gross profit.
3. **Task 3 — The Collections Problem:** Total invoices, settled invoices, open invoices, average days taken to settle, and total value outstanding per channel.
4. **Task 4 — The Ten Accounts That Matter Most:** Top 10 customers by FY2025 revenue with trading name, segment, province, and assigned sales representative.
5. **Task 5 — A Commercial Gap:** Top 15 Wholesale customers by total revenue who have never received a trade promotion.
6. **Task 6 — Are the Sales Representatives Hitting Target?:** All 18 sales representatives with their territory, monthly target, actual FY2025 monthly average revenue, and percentage of target achieved.
7. **Task 7 — The Shape of the Year:** Monthly revenue across all 24 months with month-on-month percentage change and a rolling three-month average.
8. **Task 8 — Why Has the Margin Moved?:** Revenue, gross margin percentage, and share of total annual revenue per product line across FY2024 and FY2025 to explain why company-wide gross margin fell from 34.7% to 33.0%.

## 2. Dataset Architecture & Entity Relationships

| Table Name | CSV Source File | Rows | Key Columns Used | Primary / Foreign Keys |
| :--- | :--- | :--- | :--- | :--- |
| `customers` | `sizwe_customers.csv` | 1,400 | `customer_id`, `trading_name`, `channel`, `segment`, `province`, `credit_limit`, `payment_terms`, `rep_id` | `customer_id` (PK), `rep_id` (FK) |
| `products` | `sizwe_products.csv` | 23 | `sku`, `product_name`, `product_line`, `pack_size`, `list_price`, `unit_cost` | `sku` (PK) |
| `sales_orders` | `sizwe_sales_orders.csv` | 48,144 | `order_id`, `customer_id`, `order_date`, `delivery_date`, `rep_id`, `channel`, `status` | `order_id` (PK), `customer_id` (FK), `rep_id` (FK) |
| `sales_lines` | `sizwe_sales_lines.csv` | 168,172 | `order_id`, `line_no`, `sku`, `units`, `unit_price`, `discount_pct`, `line_revenue`, `cogs` | `order_id` (FK), `sku` (FK) |
| `ar_ledger` | `sizwe_ar_ledger.csv` | 48,144 | `customer_id`, `invoice_no`, `invoice_date`, `due_date`, `invoice_amount`, `amount_settled`, `settlement_date`, `payment_terms` | `invoice_no` (PK), `customer_id` (FK) |
| `reps` | `sizwe_reps.csv` | 18 | `rep_id`, `rep_name`, `territory`, `hire_date`, `target_monthly` | `rep_id` (PK) |
| `rep_visits` | `sizwe_rep_visits.csv` | 9,400 | `visit_id`, `rep_id`, `customer_id`, `visit_date`, `outcome` | `visit_id` (PK), `customer_id` (FK), `rep_id` (FK) |
| `trade_promotions` | `sizwe_trade_promotions.csv` | 640 | `promo_id`, `customer_id`, `sku`, `start_date`, `end_date`, `mechanic`, `cost` | `promo_id` (PK), `customer_id` (FK), `sku` (FK) |
| `vendors` | `sizwe_vendors.csv` | 74 | `vendor_id`, `vendor_name`, `vendor_type`, `payment_terms`, `is_foreign` | `vendor_id` (PK) |

### How the Tables Connect

* `sales_orders.customer_id` → `customers.customer_id`
* `sales_orders.order_id` → `sales_lines.order_id`
* `sales_lines.sku` → `products.sku`
* `sales_orders.rep_id` and `customers.rep_id` → `reps.rep_id`
* `ar_ledger.customer_id` → `customers.customer_id`
* `rep_visits.customer_id` and `trade_promotions.customer_id` → `customers.customer_id`
* **Revenue & Channel Rules:** Revenue is stored on `sales_lines.line_revenue` (and cost of goods on `sales_lines.cogs`), while `sales_orders` holds the order header metadata. Furthermore, `channel` on `customers` and `sales_orders` has zero mismatches across all 48,144 orders because each customer account belongs to a single fixed channel (`Wholesale` or `Subscription`).

## 3. SQL Skills Demonstrated

| SQL Technique | Implementation Details | Tasks Applied |
| :--- | :--- | :--- |
| **Inner Joins (`JOIN`)** | Linking transactional order lines (`sales_lines`) to order headers (`sales_orders`), products (`products`), and customer accounts (`customers`). | Tasks 1, 2, 3, 4, 5, 7, 8 |
| **Left Outer Joins (`LEFT JOIN`)** | Ensuring customers without assigned reps and sales reps with zero orders are never silently dropped, and performing anti-joins on `trade_promotions`. | Tasks 4, 5, 6 |
| **Conditional Aggregation (`CASE WHEN`)** | Pivoting FY2024 (`order_date <= '2024-06-30'`) and FY2025 (`order_date > '2024-06-30'`) revenue side by side, and counting settled vs. open invoices. | Tasks 1, 3, 8 |
| **Date Functions (`DATEDIFF`, `DATE_TRUNC`)** | Calculating days to settle (`DATEDIFF(settlement_date, invoice_date)`) and grouping order dates by calendar month. | Tasks 3, 7 |
| **Common Table Expressions (`WITH` / CTEs)** | Structuring multi-stage aggregations cleanly for monthly revenue trends and annual product line margin analysis. | Tasks 7, 8 |
| **Window Functions (`LAG`, `AVG OVER`, `SUM OVER`)** | Calculating month-over-month percentage growth, 3-month rolling averages (`ROWS BETWEEN 2 PRECEDING AND CURRENT ROW`), and annual revenue share (`PARTITION BY financial_year`). | Tasks 7, 8 |
| **Defensive Null Handling (`COALESCE`, `NULLIF`)** | Guarding against division-by-zero errors and displaying `'Unassigned'` when foreign keys are null. | Tasks 1, 2, 4, 6, 7, 8 |

## 4. How to Run & Reproduce

###  Running in Databricks

1.Obtain access to the Sizwe Coffee dataset provided for the assignment.

2.Open the database in DuckDB or another compatible SQL environment.

3.Confirm that all required tables and columns are available.

4.Open queries.sql.

5.Execute the eight queries individually or in sequence.

6.Review the result sets and verify the calculations.

7.Compare the written answers using the actual query results.

8.Review the Key Findings section below with the evidence from the analysis.


## 5. Key Insights & Verified Findings for Nomsa and Bongani

### 1. Channel Performance (Task 1)
* **Wholesale** generated **R308,166,012.32** in FY2024 and **R335,426,325.02** in FY2025 (**+8.85%** growth, representing **69.88%** of total revenue).
* **Subscription** generated **R132,833,985.15** in FY2024 and **R144,573,698.33** in FY2025 (**+8.84%** growth, representing **30.12%** of total revenue).
* Both channels grew in complete lockstep, lifting total company revenue from **R441.00m** to **R480.00m**.

### 2. Product Line Profitability vs. Margin (Task 2)
* **House Blend** earns the highest absolute gross profit at **R107,148,746.60** (from **R279,530,948.90** in two-year revenue at a **38.33%** gross margin).
* **Single Origin** earns the highest gross margin percentage at **44.27%** (**R75,312,800.87** gross profit on **R170,112,619.61** revenue), followed by **Capsules** (**38.91%** margin, **R42.36m** profit) and **Decaf** (**35.83%** margin, **R32.47m** profit).
* **Bulk Commercial** is the second-largest line by revenue (**R271,838,831.01**) but earns only a **19.90%** gross margin (**R54,102,879.48** gross profit)—less than half the margin rate of Single Origin.

### 3. The Collections Problem (Task 3)
* **Wholesale** is the overwhelming source of the collections slowdown: out of **28,944** invoices, **3,454** remain open with **R87,848,227.20** still outstanding (**99.15%** of Sizwe's total unpaid receivables), and settled invoices take an average of **68.7 days** from the invoice date to settle (averaging **17.2 days past due**).
* **Subscription** has only **44** open invoices out of **19,200** (**R751,178.89** outstanding) and settles in just **3.8 days** on average. Nomsa should immediately focus credit enforcement on Wholesale accounts.

### 4. Top 10 Customers & Unpromoted Wholesale Gap (Tasks 4 & 5)
* **Halvane Retail Group** (Grocery Chain, Western Cape, managed by Kagiso Mkhize) is by far Sizwe's largest customer at **R38,399,999.89** in FY2025 revenue (8.0% of total company revenue), followed by **Highveld Deli 85** (**R3,380,200.33**) and **Table Bay Foods 280** (**R3,333,678.00**).
* Three of Sizwe's top six overall customers have **never received a trade promotion**:
  1. **Umhlanga Fresh 108** (Restaurant Group, Gauteng) — **R6,224,772.37** two-year revenue (**R3,246,754.35** in FY2025, #4 overall customer).
  2. **Ridgeway Foods 308** (Independent Cafe, KwaZulu-Natal) — **R4,448,744.72** two-year revenue (**R2,320,555.37** in FY2025, #6 overall customer).
  3. **Kalahari Kitchen 42** (Independent Cafe, Gauteng) — **R4,447,225.01** two-year revenue (**R2,320,923.25** in FY2025, #5 overall customer).
* Bongani should immediately structure volume rebates or promotional support for these three accounts to prevent competitor poaching.

### 5. Sales Representative Target Attainment (Task 6)
* **16 of the 18 sales representatives** exceeded 100% of their monthly target in FY2025, led by **Karabo van Wyk** (`R-016`, Gauteng South, **250.32%** of R900k target at **R2,252,880.96/month**), **Kagiso Mkhize** (`R-001`, Gauteng North, **249.82%** of R2.0m target at **R4,996,325.67/month**), and **Johan Maluleke** (`R-018`, KwaZulu-Natal, **242.77%** of R900k target at **R2,184,885.21/month**).
* Only two reps missed target: **Sipho Steyn** (`R-011`, KwaZulu-Natal, **97.99%** at **R1,959,822.17/month** vs. R2.0m target) and **Johan Naidoo** (`R-008`, Gauteng North, **77.63%** at **R1,164,392.30/month** vs. R1.5m target).

### 6. Seasonality & Quietest Month (Task 7)
* Revenue follows a pronounced winter peak and summer trough: monthly sales climb steadily from February through **June and July** (peaking at **R41.48m** in June 2024, **R44.58m** in July 2024, and **R45.13m** in June 2025).
* **December** is consistently Sizwe's quietest month, plunging **-13.09% MoM** to **R29,872,699.78** in December 2023 and **-12.89% MoM** to **R32,609,167.66** in December 2024 (followed closely by January at **R31.26m** and **R34.18m**).

### 7. Why Gross Margin Fell from 34.7% to 33.0% (Task 8)
* **Pure Product Mix Shift (Simpson's Paradox):** Individual product line margins did **not** decline—every single line held a virtually flat margin across FY2024 and FY2025 (`Bulk Commercial`: **19.89%** vs. **19.91%**; `House Blend`: **38.32%** vs. **38.34%**; `Single Origin`: **44.24%** vs. **44.31%**; `Capsules`: **39.01%** vs. **38.79%**; `Decaf`: **35.85%** vs. **35.81%**).
* Instead, 100% of Sizwe's top-line expansion came from **Bulk Commercial** (the lowest-margin line at **19.91%**), which surged by **46.37%** from **R110,338,581.35** (**25.02%** of total revenue) in FY2024 to **R161,500,249.66** (**33.65%** of total revenue) in FY2025.
* At the same time, all four higher-margin product lines (carrying **35.8%–44.3%** margins) experienced slight revenue contractions and lost share of total revenue (`House Blend` share fell from **32.14%** to **28.71%**; `Single Origin` fell from **19.45%** to **17.57%**; `Capsules` fell from **12.88%** to **10.84%**; `Decaf` fell from **10.50%** to **9.24%**). Replacing 8.63 percentage points of high-margin revenue share with a 19.91%-margin bulk product pulled the company-wide weighted gross margin down by 1.7 percentage points.
