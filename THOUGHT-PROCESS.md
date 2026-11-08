# Thought Process and Analytical Approach

## 1. Start with the business question

I first translate each request into a clear analytical question:
- What is being counted or measured?
- Which population and time period are in scope?
- Which statuses or categories qualify?
- What should the denominator be?
- What decision will the result support?

This prevents technically correct calculations from answering the wrong business question.

## 2. Inspect before calculating

For the Tunda Box case, I inspect identifiers, missing values, duplicate order IDs as well as exact duplicate rows, date relationships, categorical consistency and unusual numeric values before trusting summary statistics.

I distinguish between:
- A confirmed issue, such as the exact duplicate row for order 1006.
- A suspicious value requiring verification, such as order 1013 at KES 185,000.
- A missing value, such as order 1009's amount.
- A potentially valid business outcome, such as customers 109 and 110 having no orders.

I do not silently delete unusual records or replace missing amounts with zero.

## 3. Define metric logic explicitly

### Order counts
I use distinct order IDs to avoid counting the repeated order 1006 twice. In production, I would inspect all records sharing an order ID. SELECT DISTINCT only removes rows identical across the selected fields; conflicting versions require a documented canonical-record rule.

### Revenue
I filter to completed orders, deduplicate confirmed exact duplicates, and sum known amounts. SQL SUM ignores NULL values, so I flag the missing amount and explain that the total may be incomplete.

For revenue by city, an inner join excludes unmatched customer 999. That is appropriate for city attribution only when clearly disclosed. A company-wide revenue total may need to retain a valid transaction even when customer attribution is unresolved.

### City standardisation
TRIM removes leading and trailing whitespace; UPPER standardises case. In a production workflow I would also use a canonical mapping table to validate city names rather than assuming all text variants are equivalent.

### Repeat purchase rate
I define a repeat buyer as a customer with at least two distinct completed orders. I use all 10 registered customers as the denominator, giving 4/10 = 40%. If the denominator were only customers with at least one completed order, the result would be 4/8 = 50%. Both can be useful, but they answer different questions; the denominator must be stated.

### Monthly change
I aggregate completed revenue by month and use LAG to compare each month with the preceding month. I use 100.0 in the percentage calculation to avoid integer division in PostgreSQL.

## 4. Interpret unusual results carefully

August revenue is KES 192,000, driven by the KES 185,000 transaction. Excluding that order for sensitivity analysis gives KES 7,000. This is not a decision to remove the transaction; it is a way to show how much the conclusion depends on verification of one record.

The mean completed order value is KES 14,540, while the median is KES 2,400. The median better describes the centre of this skewed sample. If the KES 185,000 transaction is later proven invalid and excluded, the median is KES 2,375.

## 5. Compare like with like

For conversion rates, I calculate orders divided by sessions. I then break the overall rate down by channel. Both Organic and Paid Social conversion rates improved slightly, but Paid Social's share of sessions rose from 20% to about 44.1%. This change in traffic mix explains much of the aggregate decline.

I avoid recommending that a channel be stopped based on conversion rate alone. I would also examine customer acquisition cost, average order value, revenue, repeat purchases and profitability.

## 6. Treat experiments as evidence, not proof

The A/B example has 5% conversion in the control group and 6% in the new-page group. The observed relative uplift is 20%, but the approximate two-proportion test p-value is 0.33. At a 5% significance threshold, this sample does not provide sufficient evidence of a real difference.

A stronger experiment requires random assignment, an adequate sample size, a predefined primary metric and stopping rule, and checks for instrumentation or sample-ratio problems.

Likewise, an association between SMS receipt and purchases does not prove that SMS caused the purchases. A randomised control group helps address selection bias and confounding.

## 7. Communicate decisions and limitations

My reporting approach is:
1. State the result and metric definition.
2. Explain the most important driver.
3. Separate confirmed facts from assumptions and unresolved questions.
4. Quantify uncertainty or sensitivity where useful.
5. Recommend the next investigation or action.
6. Avoid overstating what the data proves.

## 8. Project experience and honesty

The Trading Bot V6 example is personal historical research. The reported $5,331 development and $1,426 holdout results are simulated backtest figures, not live trading income or guarantees. I describe my contribution as performing calculations and developing recommendations. I do not claim production deployment, live profitability or work I did not perform.

## 9. Reproducibility and limitations

The SQL examples use PostgreSQL syntax. The spreadsheet formulas assume the stated worksheet names and column positions. The notebook reproduces key calculations using Python.

The assessment dataset is small and illustrative. It is not sufficient to establish long-term business trends or the profitability of marketing channels. Source-system verification is still required for the duplicate, outlier, missing amount, orphan customer and date inconsistency.

For production use, I would preserve raw source data, maintain an auditable cleaning log, validate primary and foreign keys, use deterministic deduplication rules, and automate data-quality checks.
