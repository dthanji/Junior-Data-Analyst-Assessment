# Tunda Box Junior Data Analyst Assessment — Responses

**Candidate:** Dennis Thanji

> Consolidated worked responses based on the assessment case material and preparation. Confirm the final submission matches the exact assessment instructions.

## Part 1 — Project experience: Trading Bot V6 research

I worked on a personal data-driven research project called Trading Bot V6, where I wanted to investigate whether a cryptocurrency trading strategy could perform consistently across different market conditions, rather than simply produce impressive historical returns.

I worked with historical one-hour cryptocurrency market data, focusing on assets such as Bitcoin, Ethereum and Solana. My work involved analysing trading performance, calculating metrics such as returns, profit factor, win rate and drawdown, and comparing results across development and out-of-sample testing periods.

One important finding came from testing different exit strategies. The baseline trailing-stop approach recorded a simulated gain of $5,331 during development testing and $1,426 during the holdout period. However, I did not treat these results as proof of sustainable profitability. We evaluated alternative approaches, investigated calculation errors and identified areas requiring further validation.

The analysis influenced my recommendations: retain the baseline approach for further research, reject alternatives that did not demonstrate sufficient improvement, and prioritise data accuracy, realistic transaction costs and out-of-sample validation before considering any live deployment.

This project taught me that reliable analysis is not just about calculating results; it is about questioning assumptions, validating the numbers and making recommendations based on evidence.

### Handling conflicting numbers

I would not immediately assume that either my calculation or my colleague's calculation is wrong. I would first compare how we each defined the metric, including data sources, date ranges, filters, business rules and calculation methods.

I would investigate underlying records for duplicate transactions, missing values, incorrect status filters or differences in how refunds and cancelled orders were handled. Once I identify the cause, I would validate the corrected calculation, discuss the findings with my colleague and agree on a consistent definition. I would document assumptions and the calculation method so the metric can be reproduced. If the discrepancy could affect a business decision, I would communicate it rather than presenting an unverified figure as final.

## Part 2 — SQL and data analysis

### S1. Count orders by status

    SELECT status, COUNT(DISTINCT order_id) AS total_orders
    FROM orders
    GROUP BY status
    ORDER BY status;

| Status | Unique orders |
|---|---:|
| Cancelled | 2 |
| Completed | 16 |
| Refunded | 2 |
| **Total** | **20** |

There are 21 rows because order 1006 appears twice as an exact duplicate. Counting distinct order IDs prevents the duplicate from inflating the order count.

### S2. Revenue by city

    WITH deduplicated_orders AS (
        SELECT DISTINCT order_id, customer_id, status, amount_kes
        FROM orders
    )
    SELECT UPPER(TRIM(c.city)) AS city,
           SUM(o.amount_kes) AS total_revenue_kes
    FROM deduplicated_orders o
    JOIN customers c ON o.customer_id = c.customer_id
    WHERE o.status = 'completed'
    GROUP BY UPPER(TRIM(c.city))
    ORDER BY total_revenue_kes DESC;

| City | Completed revenue (KES) |
|---|---:|
| Kisumu | 187,100 |
| Nairobi | 20,650 |
| Mombasa | 5,150 |
| Nakuru | 3,300 |

Kisumu has the highest recorded revenue, but this is heavily influenced by order 1013 worth KES 185,000. Verify the transaction before relying on the ranking. The inner join excludes order 1020 because customer 999 is absent from the customers table.

### S3. Customers who never ordered

    SELECT c.customer_id, c.city, c.channel
    FROM customers c
    LEFT JOIN orders o ON c.customer_id = o.customer_id
    WHERE o.order_id IS NULL;

The result is customer 109 (Eldoret, Organic) and customer 110 (Kisumu, Paid Social). A LEFT JOIN retains all customers; a NULL order ID identifies customers with no matching order. This differs from having no completed orders: customer 106 has a refunded order but no completed order.

### S4. Repeat purchase rate

Assumption: a repeat buyer has at least two distinct completed orders. The denominator is all 10 registered customers.

    WITH customer_orders AS (
        SELECT customer_id, COUNT(DISTINCT order_id) AS completed_orders
        FROM orders o
        JOIN customers c ON c.customer_id = o.customer_id
        WHERE o.status = 'completed'
        GROUP BY o.customer_id
    )
    SELECT COUNT(*) AS repeat_buyers,
           (SELECT COUNT(*) FROM customers) AS total_customers,
           ROUND(COUNT(*) * 100.0 /
                 (SELECT COUNT(*) FROM customers), 2) AS repeat_purchase_rate
    FROM customer_orders
    WHERE completed_orders >= 2;

Four customers qualify: 101, 102, 104 and 107. The repeat purchase rate is 4 / 10 = **40%**. Using only customers with at least one completed order as the denominator gives 4 / 8 = 50%. The denominator must be stated. Customer 107 has two completed order records, although one has a missing amount; the order-count definition counts the event while the missing amount remains a data-quality issue.

### S5. Month-on-month revenue

    WITH deduplicated_orders AS (
        SELECT DISTINCT order_id, order_date, status, amount_kes
        FROM orders
    ),
    monthly_revenue AS (
        SELECT DATE_TRUNC('month', order_date) AS revenue_month,
               SUM(amount_kes) AS total_revenue
        FROM deduplicated_orders
        WHERE status = 'completed'
        GROUP BY DATE_TRUNC('month', order_date)
    ),
    comparison AS (
        SELECT revenue_month, total_revenue,
               LAG(total_revenue) OVER (ORDER BY revenue_month)
                   AS previous_month_revenue
        FROM monthly_revenue
    )
    SELECT TO_CHAR(revenue_month, 'YYYY-MM') AS month,
           total_revenue, previous_month_revenue,
           ROUND(100.0 * (total_revenue - previous_month_revenue)
                 / NULLIF(previous_month_revenue, 0), 2) AS percentage_change
    FROM comparison
    ORDER BY revenue_month;

| Month | Revenue (KES) | Previous month (KES) | Change |
|---|---:|---:|---:|
| 2026-06 | 9,000 | — | — |
| 2026-07 | 6,800 | 9,000 | -24.44% |
| 2026-08 | 192,000 | 6,800 | +2,723.53% |
| 2026-09 | 10,300 | 192,000 | -94.64% |

August's sharp increase is largely explained by order 1013 (KES 185,000). Without that order, August revenue would be KES 7,000. Order 1009's NULL amount is ignored by SUM and should be investigated. The calculation uses 100.0 to avoid integer division in PostgreSQL.

## Part 3 — Data quality and spreadsheets

### D1. Data-quality audit

1. **Duplicate order:** Order 1006 appears twice with identical details. Verify the source and count one valid order.
2. **Unusual transaction:** Order 1013 is KES 185,000, far above the other orders. Verify the invoice and payment record; do not automatically delete it.
3. **Missing amount:** Order 1009 is marked completed but has a NULL amount. Investigate and recover the value if possible; do not assume zero.
4. **Unmatched customer:** Order 1020 references customer 999, absent from the customer table. Investigate the missing or incorrect customer reference.
5. **Inconsistent city labels:** Nairobi appears with different capitalisation and trailing whitespace. Standardise with TRIM and a canonical case or city mapping.
6. **Date inconsistency:** Customer 103 signed up on 20 June 2026, but order 1003 is dated 11 June 2026. Verify both dates and the customer identity.
7. **Customers without orders:** Customers 109 and 110 have no matching order records. They may be valid non-converting customers and should not automatically be deleted.

I would validate records against source systems, correct only confirmed errors, preserve raw values, document assumptions and flag unresolved issues before publishing the report.

### D2. Spreadsheet formulas

Completed M-Pesa revenue:

    =SUMIFS(E:E,D:D,"completed",F:F,"M-Pesa")

This sums amounts in column E where status in column D is completed and payment method in column F is M-Pesa. Deduplicate confirmed repeated orders before using this as a final report figure.

Retrieve a customer's city from a worksheet named customers:

    =XLOOKUP(B2,customers!A:A,customers!B:B,"Not found")

This looks up the customer ID in B2 against customer IDs in customers column A and returns the city from column B. "Not found" flags unmatched IDs. For older Excel versions:

    =IFERROR(VLOOKUP(B2,customers!A:B,2,FALSE),"Not found")

### D3. Mean versus median

The mean is the sum of values divided by the number of observations. The median is the middle value after sorting observations.

After removing the exact duplicate and excluding the missing amount, the 15 completed-order amounts have a mean of **KES 14,540** and a median of **KES 2,400**.

I recommend the median as the more representative measure of a typical order value because the KES 185,000 transaction pulls the mean upwards. I would still verify that transaction before deciding whether it should be excluded from official reporting. If it is confirmed as an error and excluded, the median becomes KES 2,375.

## Part 4 — Statistics and A/B testing

### A1. Conversion rate analysis

Conversion rate = orders / sessions × 100.

- September: 2,600 / 50,000 × 100 = **5.20%**.
- October: 2,948 / 68,000 × 100 = **4.34%**.
- Change: approximately **-0.86 percentage points**.

| Channel | September | October |
|---|---:|---:|
| Organic | 2,400 / 40,000 = 6.0% | 2,318 / 38,000 = 6.1% |
| Paid Social | 200 / 10,000 = 2.0% | 630 / 30,000 = 2.1% |

Both channel conversion rates improved slightly, but Paid Social's share of sessions increased from 20% to approximately 44.1%. The overall decline is largely a traffic-mix effect. I would assess campaign quality, customer acquisition cost, average order value and profitability before changing budgets.

### A2. A/B testing

- Existing page: 50 / 1,000 = **5%**.
- New page: 60 / 1,000 = **6%**.
- Absolute difference: **1 percentage point**.
- Relative improvement: (6% - 5%) / 5% × 100 = **20%**.

The observed difference is not statistically significant at the conventional 5% level (approximate two-proportion test p-value 0.33). I would continue testing with adequate sample size, random assignment and a predefined stopping rule before concluding that the new page performs better.

### A3. Correlation versus causation

Correlation means two variables are associated; causation means one produces a change in the other. Customers who receive promotional SMS messages may purchase more often, but they may already be more loyal or more likely to buy. To test whether SMS causes additional purchases, randomly assign eligible customers to a message group and a control group, compare outcomes over the same period and assess statistical significance.

## Part 5 — Business case and recommendations

Tunda Box sessions increased from 50,000 to 68,000 and orders increased from 2,600 to 2,948, but overall conversion declined from 5.2% to 4.34%.

Organic conversion improved from 6.0% to 6.1%, while Paid Social improved from 2.0% to 2.1%. Paid Social sessions increased from 10,000 to 30,000, shifting its share of traffic from 20% to approximately 44.1%. The aggregate conversion decline is therefore largely explained by a change in traffic composition, not a deterioration in both channels.

Recommendations:

1. Investigate Paid Social performance by campaign, audience, platform and device.
2. Compare acquisition cost, average order value, revenue, repeat purchases and return on advertising spend before changing budgets.
3. Investigate landing-page engagement, checkout abandonment, payment failures and delivery costs.
4. Run controlled experiments on landing pages and checkout improvements.
5. Monitor channel-level sessions, orders, conversion, acquisition cost and profitability.

At October's 68,000 sessions, maintaining September's 5.2% overall conversion rate would imply 3,536 orders, 588 more than the actual 2,948. This is an illustrative benchmark, not a guaranteed recoverable order count, because the traffic mix changed.

I would not immediately stop Paid Social. I would investigate the profitability and quality of the additional traffic and focus on increasing profitable orders rather than maximising traffic or conversion rate alone.
