# 🟢 PHASE 1 — Advanced Filtering & Aggregation

> **Goal:** Move beyond basic SQL and learn how a Data Engineer uses filtering, aggregation, conditional logic, and NULL handling for analytics, data quality, and pipeline validation.

---

## 📚 What You'll Learn

| Topic | Why it matters |
|---|---|
| `case when` | Conditional logic and data classification |
| Conditional aggregation | Calculate multiple business/data-quality metrics in one query |
| Conditional `count()` | Count records matching different conditions |
| `having` | Filter groups after aggregation |
| `coalesce()` | Replace NULL values safely |
| `nullif()` | Prevent divide-by-zero and turn unwanted values into NULL |
| Data-quality aggregation | Measure valid, invalid, missing, processed records |
| Scenario-based SQL | Apply everything to real Data Engineering problems |

---

# 1. CASE WHEN

## What is CASE WHEN?

`case` is SQL's way of applying conditional logic.

Think of it like:

```text
if condition
    → result

else if condition
    → result

else
    → result
```

### Basic Example

Suppose we have:

```text
employees

employee_name | salary
--------------|--------
rahul         | 120000
sameer        | 75000
arjun         | 35000
```

We want to classify employees based on salary.

```sql
select
    employee_name,
    salary,
    case
        when salary >= 100000 then 'high'
        when salary >= 50000 then 'medium'
        else 'low'
    end as salary_category
from employees;
```

Result:

```text
employee_name | salary | salary_category
--------------|--------|----------------
rahul         | 120000 | high
sameer        | 75000  | medium
arjun         | 35000  | low
```

---

# 2. CASE WHEN Inside Aggregations

This is where `case` becomes extremely useful for Data Engineering.

Suppose we have:

```text
orders

order_id | customer_id | amount | status
---------|-------------|--------|----------
1        | C101        | 500    | delivered
2        | C102        | 1000   | cancelled
3        | C101        | 700    | delivered
4        | C103        | 300    | returned
```

Requirement:

> Calculate delivered revenue for each customer.

```sql
select
    customer_id,
    sum(
        case
            when status = 'delivered' then amount
            else 0
        end
    ) as delivered_revenue
from orders
group by customer_id;
```

### How it works

For every row:

```text
status = delivered?
        |
       yes → amount
        |
       no  → 0
```

Then `sum()` adds those values.

For C101:

```text
500 + 700 = 1200
```

For C102:

```text
0
```

For C103:

```text
0
```

This technique is called:

> **Conditional Aggregation**

---

# 3. Why Data Engineers Use Conditional Aggregation

A Data Engineer often needs several metrics from the same dataset.

For example:

```text
total records
valid records
invalid records
successful records
failed records
processed records
unprocessed records
```

Instead of running many separate queries, conditional aggregation allows you to calculate multiple metrics together.

This is especially useful for:

- Data-quality dashboards
- ETL validation
- Pipeline monitoring
- Business reporting
- Batch-processing statistics
- Source vs target validation

---

# 4. Conditional COUNT

Suppose we want an order-status dashboard.

Instead of filtering the table multiple times, we can calculate everything in one query.

```sql
select
    count(*) as total_orders,

    count(
        case
            when status = 'delivered' then 1
        end
    ) as delivered_orders,

    count(
        case
            when status = 'cancelled' then 1
        end
    ) as cancelled_orders,

    count(
        case
            when status = 'returned' then 1
        end
    ) as returned_orders
from orders;
```

Possible result:

```text
total_orders | delivered_orders | cancelled_orders | returned_orders
-------------|------------------|------------------|----------------
1000         | 700              | 150              | 100
```

The remaining 50 records could have other statuses.

---

# 5. Why Does COUNT(CASE WHEN...) Work?

Consider:

```sql
count(
    case
        when status = 'delivered' then 1
    end
)
```

For every row:

```text
delivered → 1
anything else → NULL
```

`count(expression)` counts non-NULL values.

So:

```text
1
1
NULL
1
NULL
```

becomes:

```text
3
```

Therefore:

```sql
count(case when condition then 1 end)
```

means:

> Count rows where the condition is true.

---

# 6. SUM(CASE WHEN...) vs COUNT(CASE WHEN...)

Both are useful.

### COUNT version

```sql
count(
    case
        when status = 'delivered' then 1
    end
)
```

Means:

> Count matching rows.

### SUM version

```sql
sum(
    case
        when status = 'delivered' then 1
        else 0
    end
)
```

Also means:

> Count matching rows.

Example:

```sql
select
    sum(
        case
            when status = 'delivered' then 1
            else 0
        end
    ) as delivered_orders
from orders;
```

Both patterns are worth knowing.

---

# 7. HAVING

## WHERE vs HAVING

This is an important interview concept.

### WHERE

`where` filters individual rows **before aggregation**.

Example:

```sql
select *
from orders
where status = 'delivered';
```

Think:

```text
rows
 ↓
filter rows
 ↓
aggregation
```

---

### HAVING

`having` filters groups **after aggregation**.

Example:

> Find customers who placed more than 10 orders.

```sql
select
    customer_id,
    count(*) as order_count
from orders
group by customer_id
having count(*) > 10;
```

Think:

```text
rows
 ↓
group by customer
 ↓
count orders
 ↓
filter groups
```

---

# 8. WHERE vs HAVING — Easy Rule

Remember:

```text
WHERE
→ filters rows

GROUP BY
→ creates groups

HAVING
→ filters groups
```

### Example

> Find customers with more than 10 delivered orders.

```sql
select
    customer_id,
    count(*) as delivered_orders
from orders
where status = 'delivered'
group by customer_id
having count(*) > 10;
```

Execution idea:

```text
orders
  ↓
WHERE status = delivered
  ↓
GROUP BY customer_id
  ↓
COUNT(*)
  ↓
HAVING count(*) > 10
```

---

# 9. COALESCE

## What is COALESCE?

`coalesce()` returns the **first non-NULL value**.

Think:

```text
coalesce(a, b, c)
```

means:

```text
If a is not NULL → use a
Otherwise use b
If b is NULL → use c
```

---

## Example

Suppose:

```text
customer_id | revenue
------------|--------
C101        | 5000
C102        | NULL
```

Query:

```sql
select
    customer_id,
    coalesce(revenue, 0) as revenue
from customer_sales;
```

Result:

```text
customer_id | revenue
------------|--------
C101        | 5000
C102        | 0
```

Instead of returning NULL, C102 gets `0`.

---

# 10. Multiple COALESCE Values

You can provide multiple alternatives.

```sql
select
    coalesce(
        phone,
        email,
        'unknown'
    ) as contact
from customers;
```

Logic:

```text
phone available?
    ↓ yes → phone

otherwise email available?
    ↓ yes → email

otherwise
    ↓
'unknown'
```

Another example:

```sql
coalesce(column1, column2, column3, 'unknown')
```

---

# 11. Why COALESCE Matters in Data Engineering

NULL values are common in real pipelines.

Examples:

```text
missing customer name
missing country
missing revenue
missing phone
missing source value
```

You may need to convert:

```text
NULL
```

into:

```text
0
unknown
not_available
```

depending on the business requirement.

This is common during:

- Silver-layer transformations
- Data cleaning
- Reporting preparation
- Data-quality processing
- Aggregation

---

# 12. NULLIF

## What is NULLIF?

`nullif(a, b)` returns:

```text
NULL if a = b
otherwise a
```

Example:

```sql
select
    nullif(quantity, 0)
from sales;
```

If:

```text
quantity = 0
```

the result becomes:

```text
NULL
```

---

# 13. NULLIF to Prevent Divide-by-Zero

Suppose we calculate:

```text
revenue / quantity
```

If quantity is zero:

```text
1000 / 0
```

This can cause a divide-by-zero error.

Instead:

```sql
select
    revenue / nullif(quantity, 0) as revenue_per_unit
from sales;
```

If:

```text
quantity = 5
```

then:

```text
1000 / 5 = 200
```

If:

```text
quantity = 0
```

then:

```text
nullif(0, 0)
```

becomes:

```text
NULL
```

so the calculation becomes:

```text
1000 / NULL
```

which results in NULL instead of dividing by zero.

---

# 14. COALESCE + NULLIF Together

These functions can be combined.

Suppose we want revenue per unit and want zero instead of NULL:

```sql
select
    coalesce(
        revenue / nullif(quantity, 0),
        0
    ) as revenue_per_unit
from sales;
```

Logic:

```text
quantity = 0
     ↓
NULLIF → NULL
     ↓
division → NULL
     ↓
COALESCE → 0
```

This is a useful NULL-handling pattern.

---

# 🔥 15. Conditional Aggregation for Data Quality

This is one of the most important concepts in this phase.

Imagine you're building a data-quality dashboard.

Your table:

```text
customers

customer_id
email
age
country
```

You need:

```text
total records
valid emails
missing emails
invalid ages
missing countries
```

You can calculate all of them in one query:

```sql
select
    count(*) as total_records,

    count(
        case
            when email is not null then 1
        end
    ) as valid_email_records,

    count(
        case
            when email is null then 1
        end
    ) as missing_email_records,

    count(
        case
            when age < 0
              or age > 120
            then 1
        end
    ) as invalid_age_records,

    count(
        case
            when country is null then 1
        end
    ) as missing_country_records
from customers;
```

---

# 16. Understanding the Data-Quality Query

### Total records

```sql
count(*)
```

Counts every row.

---

### Valid email records

```sql
count(
    case
        when email is not null then 1
    end
)
```

Counts rows where email exists.

---

### Missing email records

```sql
count(
    case
        when email is null then 1
    end
)
```

Counts rows where email is missing.

---

### Invalid age records

```sql
count(
    case
        when age < 0
          or age > 120
        then 1
    end
)
```

Counts ages outside the expected range.

---

### Missing country records

```sql
count(
    case
        when country is null then 1
    end
)
```

Counts missing countries.

---

# 17. Real Data Engineering Mental Model

Suppose a pipeline loads:

```text
10,000 records
```

You might want to know:

```text
total records       = 10,000
valid records       = 9,500
invalid records     = 300
missing values      = 200
processed records   = 9,800
failed records      = 200
```

SQL conditional aggregation lets you create these metrics in a single query.

This can feed:

```text
SQL
 ↓
Data Quality Metrics
 ↓
Pipeline Monitoring
 ↓
Dashboard / Alert
```

---

# 🧪 18. PHASE 1 Scenario Practice

> **Important:** Try these yourself before looking at solutions.

---

## 🟢 Scenario 1 — Customer Revenue

Table:

```text
orders

order_id
customer_id
order_date
amount
status
```

Requirement:

> Find the total number of orders and total revenue for each customer, considering only `delivered` orders.

### Concepts to use

```text
where
group by
count()
sum()
```

Expected thinking:

```text
orders
 ↓
keep delivered
 ↓
group by customer
 ↓
count orders
 ↓
sum amount
```

---

## 🟡 Scenario 2 — Order Status Dashboard

Return these in **one row**:

```text
total_orders
delivered_orders
cancelled_orders
returned_orders
```

### Concepts to use

```text
count()
case when
conditional aggregation
```

The goal is to produce something like:

```text
total_orders | delivered_orders | cancelled_orders | returned_orders
-------------|------------------|------------------|----------------
1000         | 700              | 150              | 100
```

---

## 🟡 Scenario 3 — High-Value Customers

Requirement:

> Find customers whose delivered-order revenue is greater than ₹50,000.

### Concepts to use

```text
where
group by
sum()
having
```

Think:

```text
filter delivered
      ↓
group customer
      ↓
calculate revenue
      ↓
having revenue > 50000
```

---

## 🟡 Scenario 4 — Data Quality

Table:

```text
customers

customer_id
email
age
country
```

Return:

```text
total_customers
missing_email_count
invalid_age_count
missing_country_count
```

Invalid age means:

```text
age < 0
OR
age > 120
```

### Concepts to use

```text
count()
case when
is null
or
conditional aggregation
```

---

# 🔴 19. Scenario 5 — Pipeline Validation

Suppose your source contains:

```text
10,000 records
```

Your target contains:

```text
9,850 records
```

You need to identify:

```text
successfully processed
failed
missing
```

Assume:

```text
source.customer_id
target.customer_id
```

### Hint

This scenario introduces the next major topic:

```text
JOIN
```

You will eventually use patterns such as:

```text
source
   ↓
LEFT JOIN
   ↓
target
   ↓
target.customer_id IS NULL
   ↓
missing records
```

This is a very common pipeline-validation pattern.

---

# 🔴 20. Scenario 6 — Business Scenario

Given:

```text
orders

order_id
customer_id
order_date
amount
status
```

Find customers who:

- placed at least 5 orders
- have at least ₹10,000 delivered revenue
- have never had a cancelled order

This combines:

```text
GROUP BY
HAVING
CASE WHEN
Conditional Aggregation
```

### Break the problem down

You need to calculate per customer:

```text
1. total order count
2. delivered revenue
3. cancelled order count
```

Then filter:

```text
total orders >= 5
AND
delivered revenue >= 10000
AND
cancelled orders = 0
```

This is exactly the type of multi-condition aggregation problem you should become comfortable solving.

---

# 🧠 21. Interview Cheat Sheet

## CASE WHEN

```sql
case
    when condition then value
    else value
end
```

Think:

> SQL IF/ELSE.

---

## Conditional Aggregation

```sql
sum(
    case
        when condition then amount
        else 0
    end
)
```

Think:

> Aggregate only when a condition is true.

---

## Conditional COUNT

```sql
count(
    case
        when condition then 1
    end
)
```

Think:

> Count rows matching a condition.

---

## WHERE

```sql
where condition
```

Think:

> Filter rows before aggregation.

---

## HAVING

```sql
having aggregate_condition
```

Think:

> Filter groups after aggregation.

---

## COALESCE

```sql
coalesce(value, replacement)
```

Think:

> First non-NULL value.

---

## NULLIF

```sql
nullif(value, unwanted_value)
```

Think:

> Convert a specific value into NULL.

---

# 🔥 22. Most Important Patterns to Memorize

### Pattern 1 — Conditional Revenue

```sql
sum(
    case
        when status = 'delivered'
        then amount
        else 0
    end
)
```

---

### Pattern 2 — Conditional Count

```sql
count(
    case
        when status = 'delivered'
        then 1
    end
)
```

---

### Pattern 3 — Group Filtering

```sql
group by customer_id
having count(*) > 10;
```

---

### Pattern 4 — NULL Replacement

```sql
coalesce(revenue, 0)
```

---

### Pattern 5 — Safe Division

```sql
revenue / nullif(quantity, 0)
```

---

### Pattern 6 — Safe Division + Default

```sql
coalesce(
    revenue / nullif(quantity, 0),
    0
)
```

---

# 💻 23. LeetCode Practice

For this phase, don't blindly solve 20 problems.

Start with the **LeetCode SQL 50** study plan.

Official study plan:

**LeetCode SQL 50**

https://leetcode.com/studyplan/top-sql-50/

Recommended warm-up problems for this phase:

| Problem | Title | Main Skill |
|---|---|---|
| 1757 | Recyclable and Low Fat Products | Filtering |
| 584 | Find Customer Referee | Filtering / NULL |
| 595 | Big Countries | Filtering |
| 1148 | Article Views I | Filtering |
| 1683 | Invalid Tweets | Data validation |
| 1633 | Percentage of Users Attended a Contest | Aggregation |
| 1211 | Queries Quality and Percentage | Conditional aggregation |
| 1193 | Monthly Transactions I | Conditional aggregation + dates |
| 1174 | Immediate Food Delivery II | Aggregation / conditional logic |

Don't worry if some problems overlap with things you already know.

The goal is:

```text
Warm-up
   ↓
Recognize SQL pattern
   ↓
Solve independently
   ↓
Understand why the query works
   ↓
Move to harder problems
```

---

# 🧩 24. Phase 1 Learning Flow

The concepts in this phase connect together:

```text
CASE WHEN
    ↓
Conditional Logic
    ↓
Conditional Aggregation
    ↓
COUNT / SUM
    ↓
GROUP BY
    ↓
HAVING
    ↓
NULL Handling
    ↓
COALESCE / NULLIF
    ↓
Data Quality
    ↓
Pipeline Validation
```

---

# 🚀 25. How This Connects to Your Data Engineering Work

These SQL concepts are not isolated interview topics.

They map directly to real pipelines.

### Data Quality

```text
Source
  ↓
SQL validation
  ↓
CASE WHEN
  ↓
Conditional COUNT
  ↓
Quality metrics
```

### Pipeline Monitoring

```text
Total records
      ↓
Processed records
      ↓
Failed records
      ↓
Missing records
```

### Incremental/Data Processing

You may classify records:

```text
new
updated
processed
failed
```

using `case when`.

### Bronze → Silver Transformation

During transformation:

```text
Bronze data
    ↓
NULL handling
    ↓
CASE WHEN
    ↓
Validation
    ↓
Aggregation
    ↓
Silver data
```

---

# 🎯 26. Phase 1 Final Checklist

Before moving to Phase 2, make sure you can explain and use:

### CASE

- [ ] What is `case when`?
- [ ] How does `case` work like IF/ELSE?
- [ ] How do you use `case` inside `sum()`?
- [ ] How do you use `case` inside `count()`?

### Aggregation

- [ ] `count()`
- [ ] `sum()`
- [ ] `avg()`
- [ ] `min()`
- [ ] `max()`
- [ ] `group by`
- [ ] `having`

### Filtering

- [ ] Difference between `where` and `having`
- [ ] Filter rows before aggregation
- [ ] Filter groups after aggregation

### NULL Handling

- [ ] What is NULL?
- [ ] `is null`
- [ ] `is not null`
- [ ] `coalesce()`
- [ ] `nullif()`

### Data Engineering

- [ ] Conditional aggregation
- [ ] Data-quality metrics
- [ ] Missing-record detection
- [ ] Pipeline validation
- [ ] Source vs target thinking

---

# 🏆 Phase 1 Interview Questions

Before moving on, you should be able to answer:

### Q1. What is conditional aggregation?

> Using conditional logic such as `case when` inside aggregate functions like `sum()` or `count()` to calculate metrics based on conditions.

### Q2. WHERE vs HAVING?

> `where` filters individual rows before aggregation, while `having` filters grouped results after aggregation.

### Q3. Why use COUNT(CASE WHEN...)?

> To count only rows that satisfy a specific condition while calculating multiple conditional metrics in a single query.

### Q4. What does COALESCE do?

> It returns the first non-NULL value from the provided expressions.

### Q5. Why use NULLIF?

> It converts a specified value into NULL, commonly to prevent divide-by-zero errors.

### Q6. What is conditional aggregation useful for in Data Engineering?

> It is useful for data-quality checks, pipeline monitoring, validation metrics, success/failure counts, and business metrics.

---

# ⭐ Final Phase 1 Takeaway

The most important mindset is:

> **Don't just write SQL that returns data. Write SQL that measures, validates, transforms, and explains data.**

A Data Engineer should be able to look at a table and ask:

```text
How many records do I have?
        ↓
How many are valid?
        ↓
How many are invalid?
        ↓
How many are missing?
        ↓
How many were successfully processed?
        ↓
Which customers/products/orders meet a business condition?
```

And then translate those questions into:

```text
CASE WHEN
     +
Conditional Aggregation
     +
GROUP BY
     +
HAVING
     +
COALESCE / NULLIF
```

That is the core of **Phase 1 — Advanced Filtering & Aggregation**.
