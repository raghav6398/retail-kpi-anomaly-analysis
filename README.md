# Business KPI Monitoring & Anomaly Detection  
Retail Analytics Project (Superstore Dataset)

## Project Overview
This project focuses on monitoring retail business performance using key KPIs and identifying abnormal changes in revenue over time. The objective is to detect unusual spikes or drops in performance and explain why they occurred by drilling down into regions and product categories.

## Business Context
A retail company wants to monitor its weekly business performance using core KPIs such as Revenue, Orders, and Average Order Value (AOV). When abnormal changes occur, the business needs to identify which regions or product categories caused the deviation so that corrective actions can be taken.

---

## Dataset
**Superstore Sales Dataset**  
**Grain:** One row represents one product sold in one order (transaction-level data)

**Key Columns:**
- Order Date
- Order ID
- Region
- Category
- Sub-Category
- Sales

---

## Tools & Technologies
- **Excel** – Data preparation, KPI calculation, statistical analysis  
- **SQL** – Weekly aggregation, window functions, anomaly detection logic  
- **Power BI** – Executive dashboard and interactive analysis  
- **DAX** – KPI measures and week-over-week calculations  

---

## KPI Definition & Data Preparation
- Converted transaction dates to weekly granularity using Week Start Date
- Defined and calculated core KPIs:
  - Revenue
  - Orders
  - Average Order Value (AOV)
- Aggregated transaction-level data to a weekly level to reduce noise and align with business review cycles

---

## Anomaly Detection Methodology
- Calculated week-over-week (WoW) revenue change and WoW percentage
- Established baseline behavior using historical average and standard deviation
- Flagged abnormal weeks using statistical thresholds to identify significant revenue deviations

---

## Root Cause Analysis
- Investigated identified anomaly weeks
- Performed region-wise revenue contribution analysis
- Analyzed category and sub-category level performance
- Identified Technology sub-categories as key drivers of major revenue spikes

---

## SQL Implementation

### Weekly KPI Aggregation
```sql
SELECT
    DATE_TRUNC('week', order_date) AS week_start,
    SUM(sales) AS revenue,
    COUNT(DISTINCT order_id) AS orders,
    SUM(sales) / COUNT(DISTINCT order_id) AS aov
FROM superstore
GROUP BY week_start
ORDER BY week_start;
```

### Week-over-Week Revenue Change
```sql
WITH weekly_kpi AS (
    SELECT
        DATE_TRUNC('week', order_date) AS week_start,
        SUM(sales) AS revenue
    FROM superstore
    GROUP BY week_start
)
SELECT
    week_start,
    revenue,
    (revenue - LAG(revenue) OVER (ORDER BY week_start)) AS wow_change,
    (revenue - LAG(revenue) OVER (ORDER BY week_start))
        / LAG(revenue) OVER (ORDER BY week_start) AS wow_pct
FROM weekly_kpi
ORDER BY week_start;
```

### Anomaly Flagging Logic
```sql
WITH weekly_kpi AS (
    SELECT
        DATE_TRUNC('week', order_date) AS week_start,
        SUM(sales) AS revenue
    FROM superstore
    GROUP BY week_start
),
wow_calc AS (
    SELECT
        week_start,
        revenue,
        (revenue - LAG(revenue) OVER (ORDER BY week_start))
            / LAG(revenue) OVER (ORDER BY week_start) AS wow_pct
    FROM weekly_kpi
)
SELECT
    week_start,
    revenue,
    wow_pct,
    CASE
        WHEN ABS(wow_pct) > 2 * STDDEV(wow_pct) OVER ()
        THEN 'Anomaly'
        ELSE 'Normal'
    END AS anomaly_flag
FROM wow_calc
ORDER BY week_start;
```

---

## Power BI Dashboard

### Dashboard Objective
Provide an executive-level view of weekly business performance, highlight abnormal revenue behavior, and enable quick identification of performance drivers.

### Dashboard Features
- KPI cards for Revenue, Orders, and AOV
- Weekly revenue trend with visible spikes and drops
- Week-over-week revenue change table with conditional formatting
- Category and sub-category contribution analysis
- Minimal slicers for time-based and regional filtering

### DAX Measures Used
```DAX
Revenue = SUM('RAW DATA'[Sales])

Orders = DISTINCTCOUNT('RAW DATA'[Order ID])

AOV = DIVIDE([Revenue], [Orders])

Previous Week Revenue =
CALCULATE (
    [Revenue],
    FILTER (
        ALL ( 'Date' ),
        'Date'[Week Start] = MAX ( 'Date'[Week Start] ) - 7
    )
)

WoW Revenue % =
DIVIDE (
    [Revenue] - [Previous Week Revenue],
    [Previous Week Revenue]
)
```

---

## Key Insights
- Weekly revenue exhibits high volatility with several statistically significant anomalies
- Major revenue spikes are primarily driven by Technology sub-categories
- Week-over-week monitoring enables early detection of abnormal business performance

---

## Skills Demonstrated
- Business KPI design and monitoring
- Time-series analysis and anomaly detection
- SQL window functions and aggregations
- Power BI dashboard development
- DAX-based analytical calculations
- Data storytelling for business stakeholders

---

## Final Outcome
Delivered an end-to-end analytics solution that monitors business KPIs, detects abnormal performance patterns, identifies root causes, and presents insights through an executive-ready Power BI dashboard.
