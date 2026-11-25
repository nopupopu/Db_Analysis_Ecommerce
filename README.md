# E-Commerce Sales Analysis

Analysis of four years of order data from a fictional global e-commerce store selling consumer electronics (AirPods, MacBooks, gaming monitors, accessories). The goal was to understand what drives revenue, how sales moved over time, and whether the newly launched loyalty program is worth investing in.

**Tools:** Excel (cleaning, pivot tables), SQL (CTEs, window functions, joins, CASE), Tableau (dashboard)

## Contents
- [Data](#data)
- [Data cleaning](#data-cleaning)
- [Part 1: Trends (Excel)](#part-1-trends-excel)
- [Part 2: Targeted questions (SQL)](#part-2-targeted-questions-sql)
- [Part 3: Dashboard (Tableau)](#part-3-dashboard-tableau)
- [Recommendations](#recommendations)
- [Repo structure](#repo-structure)

## Data
Four tables, joined on order ID, customer ID and country code:

| Table | Contents |
|---|---|
| `orders` | One row per order: customer, product, USD and local price, purchase date, platform (website / mobile app) |
| `order_status` | Purchase, ship, delivery and refund timestamps per order |
| `customers` | Marketing channel, account creation method, country, loyalty-program flag, signup date |
| `geo_lookup` | Country code → region (NA, EMEA, APAC, LATAM) |

![ERD](Data/elist_erd.png)

## Data cleaning
The raw extract had **108,127 rows**. Issues found and fixed:
- **~15,200 duplicate rows** (same order ID repeated), leaving **92,930 unique orders**
- Inconsistent product names, e.g. `27in"" 4k gaming monitor` vs `27in 4K gaming monitor`, and lowercase `bose soundsport headphones`
- 16 invalid timestamps (e.g. a minute value of 62, `/N`, blanks)
- 33 orders with no price
- Blank / `unknown` marketing channels kept as their own category instead of being dropped

## Part 1: Trends (Excel)
Pivot tables for yearly, monthly and quarterly performance, product mix, loyalty program, marketing channel and shipping. See the [Excel workbook](Excel/elist_orders_case_study.xlsx).

**Yearly performance**

| Year | Orders | Revenue | AOV |
|---|---|---|---|
| Year 1 | 14,294 | $3.26M | $228 |
| Year 2 | 28,922 | $8.63M | $298 |
| Year 3 | 30,733 | $7.80M | $254 |
| Year 4 | 18,966 | $4.36M | $230 |

- Year 2 had the highest revenue and AOV.
- In Year 4, revenue fell **44%** compared with Year 3, and orders fell 38%.
- **Seasonality:** the first half of the year is stronger. Q2 is the best quarter ($6.7M across all years) and Q4 the weakest ($5.6M).

**Products**

| Product | Share of orders | Share of revenue |
|---|---|---|
| Apple AirPods | 45% | 28% |
| 27in 4K gaming monitor | 22% | 35% |
| Samsung charging cable pack | 20% | 2% |
| MacBook Air | 4% | 22% |
| ThinkPad | 3% | 11% |

- Cheap accessories drive order volume; computer hardware drives revenue.
- Bose headphones sold only 25 units over four years.
- Refund rates are highest for laptops (ThinkPad 12.0%, MacBook Air 11.8%) and lowest for the charging pack (1.3%).

**Loyalty program**
- In Year 1, members spent **$30 less** per order than non-members ($202 vs $231).
- By Year 4 this had flipped: members spent **$34 more** per order ($246 vs $212) and generated **27% more revenue** ($2.44M vs $1.92M).
- The biggest gap is email: in Year 4, members spent $524K through email vs $165K for non-members (about 3.2×).
- Caveat: this is a correlation. Customers who join a loyalty program may already be higher spenders, so the program's causal effect would need an experiment to confirm.

## Part 2: Targeted questions (SQL)
See the [SQL queries](sql/elist_sales_trends_queries).

| Question | Techniques |
|---|---|
| Monthly and quarterly orders, sales and AOV for MacBooks in North America | Joins across 3 tables, `DATE_TRUNC`, CTE |
| Monthly refund rate and refunds for Apple products | `CASE WHEN` inside aggregates |
| Products with the highest refund rate | CTE, product-name cleaning with `CASE` |
| AOV and new customers by account creation method | Filtering on signup date, `COUNT(DISTINCT)` |
| Days between signup and first purchase | CTE with `MIN()`, `DATE_DIFF` |
| Best marketing channel per region by sales, AOV and orders | `ROW_NUMBER() OVER (PARTITION BY region ...)` |
| Month with the most refunds for each brand | CTE + `QUALIFY ROW_NUMBER()` |

Queries are written in BigQuery SQL. In MySQL, the `QUALIFY` filters would become a CTE with `ROW_NUMBER()` and a `WHERE rn = 1` outside it.

## Part 3: Dashboard (Tableau)
See the [Tableau dashboard export (PDF)](Data_visualisation/elist_db%20analysis_tableau.pdf), built on a [joined dataset](Data/elist_tableau_dataset.csv).

- Sales, order and time-to-ship trends by product
- Product, order and sales mix over time
- Filters for region, marketing channel and purchase platform

## Recommendations
1. **Expand the loyalty program**, using email as the main lever, since members respond to it far more than non-members.
2. **Grow computer hardware**, which earns most of the revenue from a small share of orders.
3. **Add cost data** (customer acquisition cost, cost of goods sold) to measure profit and customer lifetime value, not just revenue.
4. **Test the loyalty program's real impact** with an A/B test or a before/after comparison of the same customers.
5. **Investigate laptop refunds**, which run at about 12%, several times the rate for accessories.

## Repo structure
```
Data/                 raw CSVs, ERD, joined dataset used for Tableau
Excel/                cleaning and pivot-table workbook
sql/                  analysis queries and the Tableau dataset query
Data_visualisation/   Tableau dashboard export (PDF)
```

---
*Dataset and question set adapted from [Sean Atkinson's e-commerce case study](https://github.com/sean-atkinson/elist_ecommerce_analysis).*
