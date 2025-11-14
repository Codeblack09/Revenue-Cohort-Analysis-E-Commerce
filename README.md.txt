# Financial & Operational Analytics Dashboard

---

## 1. What's the Goal?
The point of this project was to go beyond a simple "sales" dashboard. **I** wanted to build a tool that could answer deeper questions about the business:
* How stable is **my** revenue, and can **I** spot problems automatically?
* Are **my** new customers sticking around? (i.e., retention)
* How do **my** support tickets and payment problems relate to each other?

---

## 2. How I Built It (The Data Model)

This whole report is built on a star schema. The `Transactions` table is the "center" of everything, connecting to **my** dimensions for `dim_date`, `Product`, and `Customers`.

**I** also pulled in `Support_Tickets` and `Sessions` data, linking them back to the `Customers` table. This is what lets **me** see how customer *behavior* (like filing a ticket) impacts their *transactions*.

A key part of this setup is a dedicated `Measures` table. All **my** DAX formulas live there, which keeps the model clean and easy to manage.

**[Space to insert Data Model screenshot]**
*Replace this line with your Data Model image.*


---

## 3. Feature 1: Revenue Trend & Anomaly Detection
**I** didn't just want to see a line chart of revenue. **I** wanted the report to *tell me* when a month's revenue was unusually high or low.

To do this, **I** built a measure that calculates a Z-Score for every month. It compares the current month's revenue against the long-term average and standard deviation. If a month is more than 1.5 standard deviations away from the average, this measure flags it as an "anomaly" so it stands out.

**DAX Measure:**
```dax
Revenue_Anomaly = 
VAR vAllMonthlyRevenues =
    SUMMARIZE(
        ALL('dim_date'),  // Ignores all date slicers
        'dim_date'[Year],
        'dim_date'[Month_num],
        "__MonthlyRevenue", [Total_Revenue]
    )
VAR vGlobalAvg = AVERAGEX(vAllMonthlyRevenues, [__MonthlyRevenue])
VAR vGlobalStdDev = STDEVX.P(vAllMonthlyRevenues, [__MonthlyRevenue])
VAR vCurrentRevenue = [Total_Revenue]
VAR vZ_Score =
    IF(
        vGlobalStdDev > 0,
        DIVIDE( (vCurrentRevenue - vGlobalAvg), vGlobalStdDev ),
        0
    )
RETURN
IF(
    ABS(vZ_Score) > 1.5,
    vCurrentRevenue,
    BLANK()
)

[Space to insert Revenue Anomaly screenshot] Replace this line with your screenshot for this feature.

---

## 4. Feature 2: Customer Cohort & Retention Analysis
This is the most important part for understanding customer loyalty. **I** wanted to track customers based on the month they joined (their "Cohort") and see what percentage of them were still active in the following months.

This required three key measures:

1.  **Cohort_customers:** First, **I** calculate the total number of unique customers in each cohort, ignoring all other filters except their cohort month.
    **DAX Measure:**
    ```dax
    Cohort_customers = CALCULATE(DISTINCTCOUNT(Customers[customer_id]), ALLEXCEPT('Customers',Customers[Cohort_Month]))
    ```

2.  **Active_Customers:** Next, **I** just get a simple count of unique customers who made a transaction in the current period.
    **DAX Measure:**
    ```dax
    Active_Customers = DISTINCTCOUNT(Transactions[customer_id])
    ```

3.  **Retention %:** Finally, **I** divide the active customers by their original cohort size. This gives **me** the retention rate for that specific cohort over time.
    **DAX Measure:**
    ```dax
    Retention % = DIVIDE([Active_Customers], [Cohort_customers], 0)
    ```

**[Space to insert Cohort Analysis screenshot]**
*Replace this line with your screenshot for this feature.*

---

## 5. Feature 3: Operational Efficiency (Tickets vs. Payments)

**I** had a theory that payment issues were connected to support tickets. To investigate, **I** needed to link these two different parts of the business.

First, **I** needed a simple count of tickets:
**DAX Measure:**
```dax
Ticket_Volume = COUNT(Support_Tickets[customer_id])

---

## 6. Live Interactive Dashboard
[Revenue, Cohort & Operational Analysis E-Commerce](https://your-link-here)
*Replace 'https://your-link-here' with your actual public dashboard link.*
