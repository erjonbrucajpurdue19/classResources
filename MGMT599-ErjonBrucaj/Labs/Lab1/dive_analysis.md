# DIVE Analysis – Discounting and Profitability (Superstore Dataset)

Author: Erjon Brucaj  
Course: MGMT 599  
Lab 1 – Building a Data Lake and AI-Assisted Analytics 

---

### Initial Question:
> What is the relationship between discounts and profitability?

## D - Discover (Basic Finding)

### Query:
```python
df['profit_margin'] = df['profit'] / df['sales']
df.groupby('discount').agg({
    'profit': 'mean',
    'profit_margin': 'mean',
    'order_id': 'nunique'
}).reset_index().sort_values('discount')
```

### Basic Answer:
| discount | profit       | profit_margin | num_orders |
|----------|--------------|----------------|-------------|
| 0.00     | 66.90        | 0.34           | 4798        |
| 0.10     | 96.06        | 0.16           | 94          |
| 0.15     | 27.29        | 0.03           | 52          |
| 0.20     | 24.70        | 0.18           | 3657        |
| 0.30     | -45.68       | -0.12          | 227         |
| 0.40     | -111.93      | -0.22          | 206         |
| 0.50     | -310.70      | -0.55          | 66          |
| 0.80     | -101.80      | -1.83          | 300         |

### Prompt Used
"""
I found that discount levels above 30% result in negative average profits and margins. 
For example, 50% discounts have an average profit of –$310 and margin of –54.9%, 
while 80% discounts have a –182.5% margin.

Why might these extreme losses be happening at high discount levels?
What are some business explanations or hypotheses?
Could certain products, regions, or segments be driving these deep losses?
Suggest follow-up queries or factors I should explore.
"""


### First Impression:
As discounts increase past 30%, profit margins consistently become negative. The deeper the discount, the worse the profitability, revealing a clear danger zone in discounting strategy.

---

## I - Investigate (Dig Deeper)

### Guiding Questions:
1. Why do higher discounts result in losses?
2. Are certain product categories or customer segments responsible?

### Queries:
```python
df.groupby(['category', 'discount']).agg({
    'profit': 'sum',
    'order_id': 'nunique'
}).reset_index()

df['DiscountTier'] = pd.cut(df['discount'],
                            bins=[-0.01, 0.1, 0.2, 0.3, 0.5, 1.0],
                            labels=["0-10%", "11-20%", "21-30%", "31-50%", "51%+"])

tier_summary = df.groupby('DiscountTier').agg(
    OrderCount=('order_id', 'nunique'),
    TotalProfit=('profit', 'sum'),
    AvgProfitMargin=('profit_margin', 'mean'),
    AvgDiscount=('discount', 'mean')
)
```
### Prompt Used
"""
Following up on the discount-profitability analysis:

I discovered that:
- Discounts up to 20% tend to result in positive average profit.
- Discounts at 30% and above lead to increasingly negative profits and profit margins.
- The most extreme drop is at 80% discount, with an average profit margin of -1.83.

Now I want to dig deeper and understand **why** this pattern exists.

Can you help me investigate:
1. What product categories or sub-categories are most commonly associated with high discounts (30%+)?
2. Are specific customer segments or regions driving the use of high discounts?
3. Do high-discount orders have higher quantities or sales volumes?
4. Could returns, refunds, or heavy promotions explain this loss?
5. What hypotheses or dimensions should I explore to understand these losses better?

Please suggest additional queries or breakdowns I can run in Python or SQL to continue the investigation.
"""


### Findings:
- **Discount Tier Summary**:
    | Discount Tier | Order Count | Total Profit | Avg Profit Margin |
    |---------------|-------------|--------------|--------------------|
    | 0–10%         | 2679        | $330K        | 0.34               |
    | 11–20%        | 2436        | $91K         | 0.17               |
    | 21–30%        | 211         | -$10K        | -0.12              |
    | 31–50%        | 280         | -$48K        | -0.30              |
    | 51%+          | 685         | -$76K        | -1.14              |

- **Pattern**: Discounting above 20% drives negative margins. The largest losses come from the 51%+ group.

---

## V - Validate (Challenge Assumptions)

### Validation Questions:
1. Are these patterns consistent across time?
2. Could they be skewed by seasonality or outliers?

### Queries:
```python
high_discount_orders = df[df['discount'] >= 0.3]
print("Unique orders with discounts ≥ 30%:", high_discount_orders['order_id'].nunique())

print("Orders with zero or negative sales:
", df[df['sales'] <= 0])

df['order_date'] = pd.to_datetime(df['order_date'])
df['Order Month'] = df['order_date'].dt.to_period('M')
monthly_profit = high_discount_orders.groupby('Order Month')['profit'].sum()
```

### Prompt Used
"""
I’m analyzing discounting and profitability patterns in a retail dataset (Superstore). 

I discovered that orders with discounts ≥ 30% consistently result in financial losses.
There are over 1,000 unique orders in this category, and monthly profit breakdowns show losses every month — with especially large losses in November and December.

Here are my validation questions:

1. I found that monthly losses from high-discount orders spike in November and December. 
   Could this be due to seasonal promotions like Black Friday or holiday campaigns? 
   What other explanations should I consider for these patterns, and how can I test whether these are intentional (planned loss leaders) or unintentional (pricing errors or poor strategy)?

2. I used 30% as the cutoff to define “high-discount” orders. 
   How can I validate whether this threshold is statistically meaningful? 
   Are there better ways (e.g., discount-profit regression, segmentation, binning) to detect the point where discounts begin to hurt profitability?

Please help me interpret these findings and recommend next steps to strengthen or challenge my conclusions.
"""

### Findings:
- Over **1,000 orders** had discounts ≥ 30%.
- No sales had zero or negative values—no data errors.
- **Consistent losses** every month for high-discount orders.
- **November and December** have the **deepest monthly losses**, likely due to holiday promotions.

---

## E - Extend (Strategic Application)

### Strategic Questions:
1. What should the business do with unprofitable loyal customers?
2. How can we measure the impact of smarter discounting?

### Queries:
```python
loyalty_check = high_discount_orders.groupby('customer').agg({
    'order_id': 'count',
    'profit': 'sum',
    'discount': 'mean'
}).sort_values('profit').head(10)
```

### Prompt Used
"""
I’m analyzing discounting and profitability patterns in a retail dataset (Superstore).

I discovered that some repeat customers consistently purchase with high discounts (20–35%) and still generate negative total profit over time. For example, customers like Cindy Stewart and Grant Thornton placed between 3 to 12 orders but account for thousands of dollars in cumulative losses.

Here are my strategic application questions:

1. Should the business continue offering large discounts to these repeat buyers if they remain unprofitable? 
   How can we rethink loyalty in terms of contribution to profit rather than frequency of purchase?

2. Could we create a smarter loyalty or discounting program that rewards high-value behaviors — such as higher basket size or cross-category purchases — rather than just repeat transactions?

3. Is it possible to predict which customers are likely to become chronically unprofitable under the current discount strategy? 
   What customer-level attributes or behaviors should we analyze to forecast this?

Please help me interpret these findings and recommend how the business can act on them, measure impact, and mitigate risk.
"""

### Findings:
Many repeat customers consistently use high discounts and generate losses. Example:

| Customer         | Orders | Total Profit | Avg Discount |
|------------------|--------|--------------|---------------|
| Cindy Stewart    | 6      | -$6,626.39   | 0.20          |
| Grant Thornton   | 3      | -$4,108.66   | 0.25          |
| Luke Foster      | 7      | -$3,583.98   | 0.32          |

---

## Final Recommendations

### 1. **Redesign Loyalty Strategy**
Create a **Customer Value Matrix** that evaluates both profitability and frequency. Stop rewarding customers who are frequent but unprofitable.

|                 | Low Profitability                | High Profitability              |
|-----------------|----------------------------------|----------------------------------|
| High Frequency  | "Profit Drains" – Reprice/Migrate | "Champions" – Reward & Nurture |
| Low Frequency   | "Occasional" – Deprioritize       | "Potentials" – Engage & Grow    |

---

### 2. **Smarter Loyalty Program**
Reward customers based on **profitable behaviors**:
- Bonus points for buying from **high-margin categories**
- **Non-monetary perks** (e.g. free shipping) instead of deep discounts
- Introduce **tiers based on profitability**, not just spend

---

### 3. **Predict and Prevent Future Losses**
Build a simple logistic regression model to flag **new unprofitable customers** early:
- Inputs: `first_order_discount`, `early_avg_discount`, `product mix`
- Target: `cumulative_profit < 0`
- Action: Send flagged customers **smarter offers**, not blanket discounts

---

### Metrics to Track Impact:
- Increase in **Average Customer Lifetime Profit**
- Higher **average margin per order**
- Decrease in orders with discounts ≥ 30%

---
