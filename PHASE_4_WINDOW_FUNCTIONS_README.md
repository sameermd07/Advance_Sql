# 🔥 PHASE 4 — SQL Window Functions

> **Goal:** Learn how to calculate rankings, running totals, previous/next values, latest records, and other row-level analytics without collapsing the result set.

---

# 1. What Is a Window Function?

Suppose we have:

```text
customer_id | order_id | amount
------------|----------|-------
101         | 1001     | 5000
101         | 1002     | 3000
101         | 1003     | 7000
102         | 1004     | 4000
102         | 1005     | 2000
```

Requirement:

> Show every order along with the customer's total revenue.

A normal `group by`:

```sql
select
    customer_id,
    sum(total_amount) as customer_total
from orders
group by customer_id;
```

returns one row per customer.

The individual orders disappear.

A window function keeps the original rows:

```sql
select
    order_id,
    customer_id,
    total_amount,
    sum(total_amount) over (
        partition by customer_id
    ) as customer_total
from orders;
```

Result conceptually:

```text
customer_id | order_id | amount | customer_total
------------|----------|--------|---------------
101         | 1001     | 5000   | 15000
101         | 1002     | 3000   | 15000
101         | 1003     | 7000   | 15000
102         | 1004     | 4000   | 6000
102         | 1005     | 2000   | 6000
```

### Core idea

> **GROUP BY collapses rows. Window functions calculate across related rows while keeping the rows.**

---

# 2. Basic Syntax

The general pattern is:

```sql
function() over (
    partition by ...
    order by ...
)
```

For example:

```sql
sum(total_amount) over (
    partition by customer_id
) as customer_total
```

Think:

```text
PARTITION BY
→ divide the data into logical groups

ORDER BY
→ arrange rows inside each group
```

Not every window function requires both clauses.

---

# 3. GROUP BY vs Window Function

### GROUP BY

```sql
select
    customer_id,
    sum(total_amount) as customer_total
from orders
group by customer_id;
```

Result:

```text
1 row per customer
```

### Window function

```sql
select
    order_id,
    customer_id,
    total_amount,
    sum(total_amount) over (
        partition by customer_id
    ) as customer_total
from orders;
```

Result:

```text
1 row per order
+
customer total
```

### Interview answer

> `GROUP BY` reduces the result to one row per group, while a window function performs a calculation across a window of related rows without collapsing them.

---

# 4. ROW_NUMBER()

`row_number()` assigns a unique sequential number to rows.

Example:

```sql
select
    order_id,
    customer_id,
    total_amount,
    row_number() over (
        order by total_amount desc
    ) as rn
from orders;
```

Possible result:

```text
order_id | amount | rn
---------|--------|---
1014     | 99999  | 1
1004     | 99999  | 2
1017     | 74999  | 3
```

Even when values tie, `row_number()` gives different numbers.

---

# 5. ROW_NUMBER() + PARTITION BY 🔥

Requirement:

> Number orders separately for each customer, with the highest amount first.

```sql
select
    order_id,
    customer_id,
    total_amount,
    row_number() over (
        partition by customer_id
        order by total_amount desc
    ) as rn
from orders;
```

Conceptually:

```text
customer 101
    order A → 1
    order B → 2
    order C → 3

customer 102
    order D → 1
    order E → 2
```

`partition by` restarts the numbering for each customer.

---

# 🔥 6. Latest Order Per Customer

This is one of the most important Data Engineering patterns.

Requirement:

> Find the latest order for every customer.

```sql
select *
from (
    select
        o.*,
        row_number() over (
            partition by customer_id
            order by order_date desc
        ) as rn
    from orders o
) x
where rn = 1;
```

### Mental pattern

```text
PARTITION BY entity
ORDER BY timestamp DESC
ROW_NUMBER() = 1
```

This pattern is useful for:

```text
latest record
latest customer information
deduplication
CDC processing
SCD processing
incremental processing
```

---

# 7. Deterministic Ordering

If two records have exactly the same `order_date`, their relative order may not be guaranteed.

You can add a tie-breaker:

```sql
row_number() over (
    partition by customer_id
    order by order_date desc, order_id desc
)
```

This means:

```text
latest date first
and if dates tie
→ highest order_id first
```

This is a good production habit.

---

# 8. RANK vs DENSE_RANK vs ROW_NUMBER

Suppose values are:

```text
100
100
90
80
```

### ROW_NUMBER

```text
100 → 1
100 → 2
90  → 3
80  → 4
```

### RANK

```text
100 → 1
100 → 1
90  → 3
80  → 4
```

There is a gap after the tie.

### DENSE_RANK

```text
100 → 1
100 → 1
90  → 2
80  → 3
```

No gap after the tie.

---

# 9. When Should You Use Each?

## ROW_NUMBER()

Use when you need an exact sequence or exactly one row.

Examples:

```text
latest record per customer
deduplication
top 1 record per group
```

---

## RANK()

Use when ties should share a rank and gaps are acceptable.

Example:

```text
competition ranking
```

---

## DENSE_RANK()

Use when ties should share a rank but ranking levels should not have gaps.

Example:

```text
top 3 salary levels
```

---

# 🔥 10. LAG()

`lag()` looks at a previous row.

Example:

```sql
select
    customer_id,
    order_id,
    order_date,
    lag(order_date) over (
        partition by customer_id
        order by order_date
    ) as previous_order_date
from orders;
```

Possible result:

```text
customer | order | date       | previous_date
---------|-------|------------|--------------
101      | 1001  | 2024-01-05 | null
101      | 1003  | 2024-01-15 | 2024-01-05
101      | 1012  | 2024-04-10 | 2024-01-15
```

The first row has no previous row, so its previous value is NULL.

---

# 11. LEAD()

`lead()` looks at a future row.

```sql
select
    customer_id,
    order_id,
    order_date,
    lead(order_date) over (
        partition by customer_id
        order by order_date
    ) as next_order_date
from orders;
```

Useful for:

```text
next transaction
next status
next event
next login
next purchase
```

### Easy memory trick

```text
LAG
→ look backward

LEAD
→ look forward
```

---

# 🔥 12. Running Total

Suppose:

```text
date       | revenue
-----------|--------
Jan 1      | 100
Jan 2      | 200
Jan 3      | 300
```

Expected:

```text
date       | revenue | running_total
-----------|---------|--------------
Jan 1      | 100     | 100
Jan 2      | 200     | 300
Jan 3      | 300     | 600
```

Query:

```sql
sum(revenue) over (
    order by order_date
    rows between unbounded preceding and current row
) as running_total
```

Meaning:

```text
everything from the beginning
+
current row
```

---

# 13. Running Total Per Customer

```sql
sum(total_amount) over (
    partition by customer_id
    order by order_date
    rows between unbounded preceding and current row
) as running_customer_revenue
```

Now each customer's running total is calculated independently.

Conceptually:

```text
customer 101
order 1 → 500
order 2 → 800
order 3 → 1500

customer 102
order 1 → 400
order 2 → 900
```

---

# 14. Moving Average

Requirement:

> Calculate a 3-order moving average.

```sql
avg(total_amount) over (
    order by order_date
    rows between 2 preceding and current row
) as moving_avg
```

The frame contains:

```text
previous 2 rows
+
current row
```

This is useful for:

```text
trend analysis
rolling metrics
smoothing
monitoring
```

---

# 15. FIRST_VALUE and LAST_VALUE

You can also retrieve values from a window.

```sql
first_value(total_amount) over (...)
```

and:

```sql
last_value(total_amount) over (...)
```

These are useful in some analytics problems.

However, prioritize:

```text
row_number()
rank()
dense_rank()
lag()
lead()
sum() over()
avg() over()
count() over()
```

first.

---

# 16. COUNT() OVER()

Window functions are not limited to ranking.

You can count rows without collapsing them:

```sql
select
    customer_id,
    order_id,
    count(*) over (
        partition by customer_id
    ) as customer_order_count
from orders;
```

Every order row will show that customer's total number of orders.

---

# 17. Combining Aggregation + Window Functions

You can first aggregate, then rank the aggregated result.

Example:

> Rank customers by total revenue.

```sql
select
    customer_id,
    total_revenue,
    rank() over (
        order by total_revenue desc
    ) as revenue_rank
from (
    select
        customer_id,
        sum(total_amount) as total_revenue
    from orders
    group by customer_id
) x;
```

This is a common pattern:

```text
raw rows
   ↓
GROUP BY
   ↓
customer-level metrics
   ↓
WINDOW FUNCTION
   ↓
ranking
```

---

# 🧪 18. Phase 4 Practice

> Try these yourself before checking solutions.

---

## 🟢 Level 1

### Q1

Display every order along with:

```text
order_id
customer_id
total_amount
customer_total_revenue
```

Use:

```text
sum() over()
partition by
```

---

### Q2

Assign a row number to all orders based on:

> Highest `total_amount` first.

Use:

```text
row_number()
order by
```

---

### Q3

Assign a row number within each customer based on latest order.

Use:

```text
partition by customer_id
order by order_date desc
```

---

### Q4 🔥

Find the latest order for every customer.

Use the pattern:

```text
row_number()
partition by customer_id
order by order_date desc
```

Then filter:

```text
rn = 1
```

---

# 🟡 19. Level 2

### Q5

Find the highest-value order for each customer.

---

### Q6

Find the top 2 orders for every customer.

Think:

```text
row_number()
partition by customer_id
order by total_amount desc
```

Then:

```text
rn <= 2
```

---

### Q7

Rank customers based on their total revenue.

This requires:

```text
group by
+
window function
```

---

### Q8

Find the second-highest order for every customer.

Be careful about the difference between:

```text
row_number
rank
dense_rank
```

---

# 🟠 20. Level 3 — LAG / LEAD

### Q9

For every order, show:

```text
customer_id
order_id
order_date
previous_order_date
```

Use `lag()`.

---

### Q10 🔥

Calculate the number of days between a customer's current order and previous order.

This connects directly to Phase 5.

Conceptually:

```text
current date
-
previous date
```

---

### Q11

For every customer, show:

```text
current_order
next_order
```

Use `lead()`.

---

# 🔴 21. Level 4 — Real Data Engineering

## Q12 — Deduplication

Imagine the orders table contains duplicate versions of records.

Keep only the latest record for each `order_id` based on `updated_at`.

Pattern:

```sql
row_number() over (
    partition by order_id
    order by updated_at desc
)
```

Then keep:

```text
rn = 1
```

This is a very common Data Engineering pattern.

---

## Q13 — Latest Customer Record

Suppose customer information arrives multiple times.

Find the latest record for every customer based on:

```text
updated_at
```

Pattern:

```text
partition by customer_id
order by updated_at desc
```

---

## Q14 — Running Revenue

Calculate cumulative delivered revenue over time.

Output:

```text
order_date
daily_revenue
running_revenue
```

Think about whether you need to aggregate revenue by date before applying the running total.

---

## Q15 🔥

For each customer calculate:

```text
customer_id
order_id
order_date
order_amount
previous_order_amount
difference_from_previous_order
```

This combines:

```text
LAG
arithmetic
PARTITION BY
ORDER BY
```

---

# 💻 22. LeetCode Window Function Practice

### 178 — Rank Scores

https://leetcode.com/problems/rank-scores/

Focus:

```text
RANK
```

### 184 — Department Highest Salary

https://leetcode.com/problems/department-highest-salary/

Focus:

```text
ranking
partitioning
```

### 185 — Department Top Three Salaries

https://leetcode.com/problems/department-top-three-salaries/

Focus:

```text
DENSE_RANK
partition by
```

### 180 — Consecutive Numbers

https://leetcode.com/problems/consecutive-numbers/

Focus:

```text
LAG / LEAD
```

### 1174 — Immediate Food Delivery II

https://leetcode.com/problems/immediate-food-delivery-ii/

Focus:

```text
ROW_NUMBER
latest/first record patterns
```

### 1204 — Last Person to Fit in the Bus

https://leetcode.com/problems/last-person-to-fit-in-the-bus/

Focus:

```text
running totals
ordering
```

---

# 🧠 23. The Five Window Functions to Master

Make these your priority:

```text
row_number()
rank()
dense_rank()
lag()
lead()
```

Then master:

```text
sum() over()
avg() over()
count() over()
```

---

# 🔥 24. Window Function Mental Model

When you see:

> "Keep every row but calculate something across related rows."

Think:

```text
WINDOW FUNCTION
```

When you see:

> "Give each record a sequence."

Think:

```text
ROW_NUMBER
```

When you see:

> "Rank records with ties."

Think:

```text
RANK / DENSE_RANK
```

When you see:

> "Find the previous value."

Think:

```text
LAG
```

When you see:

> "Find the next value."

Think:

```text
LEAD
```

When you see:

> "Calculate cumulative values."

Think:

```text
SUM() OVER()
```

---

# 🎯 25. Phase 4 Final Checklist

Before moving on:

- [ ] Explain window functions
- [ ] Explain `over()`
- [ ] Explain `partition by`
- [ ] Explain `order by` inside a window
- [ ] Explain GROUP BY vs window functions
- [ ] Use `row_number()`
- [ ] Use `rank()`
- [ ] Use `dense_rank()`
- [ ] Use `lag()`
- [ ] Use `lead()`
- [ ] Calculate running totals
- [ ] Calculate moving averages
- [ ] Find latest record per entity
- [ ] Deduplicate using `row_number()`
- [ ] Rank aggregated results
- [ ] Understand window frames

---

# 🏆 26. Interview Questions

### Q1. What is a window function?

> A window function performs a calculation across a set of related rows while preserving the individual rows in the result.

### Q2. GROUP BY vs window function?

> GROUP BY collapses rows into groups. Window functions calculate across related rows without collapsing them.

### Q3. ROW_NUMBER vs RANK?

> ROW_NUMBER gives each row a unique number. RANK gives tied rows the same rank and leaves gaps after ties.

### Q4. RANK vs DENSE_RANK?

> Both give tied rows the same rank, but RANK leaves gaps while DENSE_RANK does not.

### Q5. How do you find the latest record per customer?

Use:

```sql
row_number() over (
    partition by customer_id
    order by updated_at desc
)
```

and keep:

```text
rn = 1
```

### Q6. What does LAG do?

> LAG returns a value from a previous row in the ordered window.

### Q7. What does LEAD do?

> LEAD returns a value from a following row in the ordered window.

### Q8. What is a running total?

> A cumulative aggregate where the current row is calculated together with all preceding rows in the window.

---

# ⭐ Final Phase 4 Takeaway

The goal is not:

> "I know what ROW_NUMBER means."

The goal is to see a business requirement and immediately recognize the SQL pattern.

For example:

> **"Give me the latest record per customer."**

You should immediately think:

```text
partition by customer_id
        ↓
order by updated_at desc
        ↓
row_number()
        ↓
rn = 1
```

Or:

> **"Show the previous order amount."**

Think:

```text
partition by customer_id
        ↓
order by order_date
        ↓
lag()
```

Or:

> **"Calculate cumulative revenue."**

Think:

```text
sum()
over(
    order by date
)
```

That pattern-recognition is the real Phase 4 skill.
