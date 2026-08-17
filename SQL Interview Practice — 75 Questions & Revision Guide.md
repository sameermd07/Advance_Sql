# SQL Interview Practice — 75 Questions & Revision Guide

A practical SQL revision and interview-preparation guide containing **75 SQL problems**, progressing from basic filtering to advanced business scenarios.

The goal is not just to memorize queries. For every question, understand:

1. What problem is being solved?
2. Which SQL concept is being tested?
3. Why does the query work?
4. What happens with `NULL`, duplicates, or ties?
5. Can you write the query without looking at the answer?

---

# 📚 Table of Contents

1. [Set Structure](#-set-structure)
2. [Database Tables Used](#-database-tables-used)
3. [Section 1 — Basics & Filtering](#section-1--basics--filtering-112)
4. [Section 2 — Joins](#section-2--joins-1324)
5. [Section 3 — Aggregation](#section-3--aggregation-2536)
6. [Section 4 — Window Functions](#section-4--window-functions-3751)
7. [Section 5 — Subqueries & CTEs](#section-5--subqueries--ctes-5261)
8. [Section 6 — Advanced / Business Scenarios](#section-6--advanced--business-scenarios-6275)
9. [Self-Score](#-self-score)
10. [Interview Revision Checklist](#-interview-revision-checklist)
11. [Important SQL Patterns](#-important-sql-patterns)

---

# 🎯 Set Structure

| Section | Questions | Main Focus |
|---|---:|---|
| Basics & Filtering | #1–12 | Core querying |
| Joins | #13–24 | Combining tables |
| Aggregation | #25–36 | Turning rows into business answers |
| Window Functions | #37–51 | Per-row analytical calculations |
| Subqueries & CTEs | #52–61 | Multi-step and readable SQL |
| Advanced / Business Scenarios | #62–75 | Real-world analytical problems |

---

# 🗄️ Database Tables Used

The questions use several example tables.

### `employees`

```sql
employees (
    employee_id,
    name,
    department,
    salary,
    manager_id
)
```

### `customers`

```sql
customers (
    customer_id,
    name,
    email,
    phone,
    age,
    income
)
```

### `orders`

```sql
orders (
    order_id,
    customer_id,
    order_date,
    status,
    amount,
    region
)
```

### `order_items`

```sql
order_items (
    item_id,
    order_id,
    product_id,
    quantity,
    price
)
```

### `products`

```sql
products (
    product_id,
    name,
    product_name,
    category,
    price
)
```

Other example tables include:

- `students`
- `sales`
- `users`
- `daily_traffic`
- `monthly_sales`
- `logins`
- `activity`
- `attendance`
- `friends`
- `experiment_events`
- `customer_revenue`

---

# Section 1 — Basics & Filtering #1–12

## #1 Find all employees earning more than $70,000

### Concept

Basic filtering using `WHERE`.

### Query

```sql
select *
from employees
where salary > 70000;
```

### Remember

`WHERE` filters individual rows before grouping or aggregation.

---

## #2 Find the 5 most recent orders

### Concept

Sorting + limiting results.

```sql
select *
from orders
order by order_date desc
limit 5;
```

### Remember

- `desc` → newest/highest first
- `asc` → oldest/lowest first
- `limit` restricts the number of returned rows

---

## #3 Find customers whose email is a Gmail address

### Concept

Pattern matching using `LIKE`.

```sql
select *
from customers
where email like '%gmail.com';
```

### Remember

`%` means zero or more characters.

Examples:

```sql
'a%'       -- starts with a
'%gmail%'  -- contains gmail
'%gmail.com' -- ends with gmail.com
```

---

## #4 Find orders whose status is shipped or delivered

### Concept

`IN`.

```sql
select *
from orders
where status in ('shipped', 'delivered');
```

Equivalent:

```sql
where status = 'shipped'
   or status = 'delivered'
```

`IN` is cleaner when checking multiple values.

---

## #5 Find customers with a missing phone number

### Concept

Handling `NULL`.

```sql
select *
from customers
where phone is null;
```

### Important

Never use:

```sql
phone = null
```

Use:

```sql
phone is null
```

or:

```sql
phone is not null
```

---

## #6 Find all distinct product categories

### Concept

Removing duplicate values.

```sql
select distinct category
from products;
```

---

## #7 Find orders placed in the last 30 days

### Concept

Date filtering.

```sql
select *
from orders
where order_date >= current_date - interval '30 days';
```

> **PostgreSQL syntax.** Date functions differ between SQL databases.

---

## #8 Find products priced between $50 and $200

### Concept

`BETWEEN`.

```sql
select *
from products
where price between 50 and 200;
```

### Important

`BETWEEN` is inclusive.

Equivalent:

```sql
where price >= 50
  and price <= 200
```

---

## #9 Sort employees by department and salary

```sql
select *
from employees
order by department asc, salary desc;
```

### Meaning

First sort by department.

Within each department, sort employees by salary from highest to lowest.

---

## #10 Rename `name` to `Full Name`

### Concept

Column alias.

```sql
select name as "Full Name"
from employees;
```

---

## #11 Find the 3 highest-priced products

```sql
select *
from products
order by price desc
limit 3;
```

---

## #12 Find employees whose name starts with A

```sql
select *
from employees
where name like 'A%';
```

---

# Section 2 — Joins #13–24

Joins are one of the most important SQL interview topics.

The main idea:

> **Use joins when information required for your answer exists in multiple tables.**

---

## #13 List each order alongside the customer's name

```sql
select o.order_id,
       c.name
from orders o
inner join customers c
    on o.customer_id = c.customer_id;
```

### Concept

`INNER JOIN`

Only matching records from both tables are returned.

---

## #14 List every customer and their order ID if they have one

```sql
select c.name,
       o.order_id
from customers c
left join orders o
    on c.customer_id = o.customer_id;
```

### Why `LEFT JOIN`?

We want **every customer**, even customers who never ordered.

Customers without orders will have:

```text
order_id = NULL
```

---

## #15 Find customers who have never placed an order

### Important interview pattern: Anti Join

```sql
select c.name
from customers c
left join orders o
    on c.customer_id = o.customer_id
where o.order_id is null;
```

### Pattern to remember

```sql
left join
where right_table.key is null
```

This is commonly used to find records that **do not have a match**.

---

## #16 List every employee with their manager's name

### Concept

Self Join.

The same table is joined with itself.

```sql
select e.name as employee,
       m.name as manager
from employees e
left join employees m
    on e.manager_id = m.employee_id;
```

### Important

`e` represents the employee.

`m` represents the manager.

---

## #17 Reconcile two tables and show rows existing in only one

```sql
select *
from table_a a
full outer join table_b b
    on a.id = b.id
where a.id is null
   or b.id is null;
```

### Concept

`FULL OUTER JOIN`

Returns:

- matching records
- records only in table A
- records only in table B

The `where` condition removes matching records and leaves only unmatched rows.

---

## #18 Find pairs of students in the same class

### Concept

Self Join + avoiding duplicate pairs.

```sql
select a.name,
       b.name
from students a
join students b
    on a.class_id = b.class_id
   and a.student_id < b.student_id;
```

### Why `a.student_id < b.student_id`?

Without it:

```text
Alice - Bob
Bob - Alice
```

would both appear.

The condition keeps only one pair.

---

## #19 List products that have never been ordered

```sql
select p.name
from products p
left join order_items oi
    on p.product_id = oi.product_id
where oi.order_id is null;
```

### Pattern

Again:

**LEFT JOIN + NULL check = records without a match.**

---

## #20 Count orders for every customer, including zero orders

```sql
select c.name,
       count(o.order_id) as order_count
from customers c
left join orders o
    on c.customer_id = o.customer_id
group by c.name;
```

### Important interview point

Use:

```sql
count(o.order_id)
```

rather than:

```sql
count(*)
```

Because `COUNT(column)` ignores `NULL`.

---

## #21 Join orders, customers and products

```sql
select o.order_id,
       c.name,
       p.product_name
from orders o
join customers c
    on o.customer_id = c.customer_id
join order_items oi
    on o.order_id = oi.order_id
join products p
    on oi.product_id = p.product_id;
```

### Join path

```text
orders
   ↓
customers

orders
   ↓
order_items
   ↓
products
```

---

## #22 Find employees working in the same department as Priya

```sql
select e2.name
from employees e1
join employees e2
    on e1.department = e2.department
where e1.name = 'Priya'
  and e2.name != 'Priya';
```

### Concept

Self Join.

---

## #23 Find total item count per order

```sql
select o.order_id,
       count(oi.item_id) as item_count
from orders o
join order_items oi
    on o.order_id = oi.order_id
group by o.order_id;
```

---

## #24 List customers and their most recent order date

```sql
select c.name,
       max(o.order_date) as last_order
from customers c
left join orders o
    on c.customer_id = o.customer_id
group by c.name;
```

### Why `MAX()`?

The maximum order date is the customer's latest order.

---

# Section 3 — Aggregation #25–36

Aggregation answers questions such as:

- How much?
- How many?
- What is the average?
- What is the minimum?
- What is the maximum?

Important functions:

```text
count()
sum()
avg()
min()
max()
```

---

## #25 Total revenue per region

```sql
select region,
       sum(revenue) as total_revenue
from sales
group by region;
```

---

## #26 Regions where average revenue exceeds $5,000

```sql
select region,
       avg(revenue) as avg_revenue
from sales
group by region
having avg(revenue) > 5000;
```

### Important

`WHERE` filters rows.

`HAVING` filters groups.

Remember:

```text
WHERE  → before GROUP BY
HAVING → after GROUP BY
```

---

## #27 Count distinct customers per product category

```sql
select category,
       count(distinct customer_id) as customer_count
from orders
group by category;
```

### Key concept

`count(distinct ...)` counts unique values rather than rows.

---

## #28 Completed vs cancelled orders per region

```sql
select region,
       sum(case
               when status = 'completed' then 1
               else 0
           end) as completed,
       sum(case
               when status = 'cancelled' then 1
               else 0
           end) as cancelled
from orders
group by region;
```

### Concept

Conditional aggregation.

This pattern is extremely useful in real-world data engineering and analytics.

---

## #29 Find the second-highest salary

```sql
select max(salary)
from employees
where salary < (
    select max(salary)
    from employees
);
```

### Why does this handle ties?

If several employees have the highest salary, they are all excluded.

The query finds the maximum salary below the highest salary.

---

## #30 Minimum, maximum and average salary per department

```sql
select department,
       min(salary) as min_salary,
       max(salary) as max_salary,
       avg(salary) as avg_salary
from employees
group by department;
```

---

## #31 Top 10 products by total revenue

```sql
select product_id,
       sum(quantity * price) as revenue
from order_items
group by product_id
order by revenue desc
limit 10;
```

### Important

Revenue is calculated as:

```text
quantity × price
```

Then the values are summed for each product.

---

## #32 Number of orders per month

```sql
select date_trunc('month', order_date) as month,
       count(*) as order_count
from orders
group by 1
order by 1;
```

> PostgreSQL-specific `date_trunc()` syntax.

---

## #33 Find duplicate email addresses

```sql
select email,
       count(*) as occurrences
from users
group by email
having count(*) > 1;
```

### Pattern

To find duplicates:

```sql
group by column
having count(*) > 1
```

Memorize this pattern.

---

## #34 Total sales and order count

```sql
select sum(amount) as total_sales,
       count(*) as order_count
from orders;
```

---

## #35 Average order value per customer with 5+ orders

```sql
select customer_id,
       avg(amount) as average_order_value
from orders
group by customer_id
having count(*) >= 5;
```

---

## #36 Percentage contribution of each region to total revenue

```sql
select region,
       sum(revenue) * 100.0 /
       (
           select sum(revenue)
           from sales
       ) as pct_of_total
from sales
group by region;
```

### Concept

Each group's value divided by the overall total.

---

# Section 4 — Window Functions #37–51

Window functions are extremely important for SQL interviews.

Unlike `GROUP BY`, window functions **do not collapse rows**.

Example:

```sql
select name,
       salary,
       rank() over (order by salary desc)
from employees;
```

The employee rows remain.

---

## #37 Rank employees by salary

```sql
select name,
       salary,
       rank() over (
           order by salary desc
       ) as rnk
from employees;
```

---

## #38 Rank employees within each department

```sql
select name,
       department,
       salary,
       rank() over (
           partition by department
           order by salary desc
       ) as dept_rank
from employees;
```

### Important

```sql
partition by department
```

means:

> Restart the ranking for every department.

---

## #39 Compare RANK and DENSE_RANK

```sql
select name,
       salary,
       rank() over (
           order by salary desc
       ) as rnk,
       dense_rank() over (
           order by salary desc
       ) as drnk
from employees;
```

### Example

If salaries are:

```text
100
100
90
80
```

`RANK()`:

```text
1
1
3
4
```

`DENSE_RANK()`:

```text
1
1
2
3
```

### Remember

`RANK()` leaves gaps.

`DENSE_RANK()` does not.

---

## #40 Running total of daily sales

```sql
select order_date,
       amount,
       sum(amount) over (
           order by order_date
       ) as running_total
from orders;
```

---

## #41 Seven-day rolling average

```sql
select visit_date,
       visits,
       avg(visits) over (
           order by visit_date
           rows between 6 preceding and current row
       ) as rolling_avg
from daily_traffic;
```

### Why 6 preceding?

Current row + previous 6 rows = 7 rows.

---

## #42 Month-over-month revenue change

```sql
select month,
       revenue,
       revenue -
       lag(revenue) over (
           order by month
       ) as change
from monthly_sales;
```

### Important

`lag()` looks at the previous row.

---

## #43 Days until customer's next order

```sql
select customer_id,
       order_date,
       lead(order_date) over (
           partition by customer_id
           order by order_date
       ) - order_date as days_to_next
from orders;
```

### Important

`lead()` looks forward.

```text
lag()  → previous row
lead() → next row
```

---

## #44 Assign students to quartiles

```sql
select name,
       score,
       ntile(4) over (
           order by score desc
       ) as quartile
from students;
```

### Concept

`NTILE(4)` divides rows into four groups.

---

## #45 Show department's highest salary beside every employee

```sql
select name,
       department,
       salary,
       first_value(salary) over (
           partition by department
           order by salary desc
       ) as top_salary_in_dept
from employees;
```

---

## #46 Percentile rank of each salary

```sql
select name,
       salary,
       percent_rank() over (
           order by salary
       ) as pct_rank
from employees;
```

---

## #47 Return only each customer's latest order

### Classic interview problem

```sql
with ranked as (
    select *,
           row_number() over (
               partition by customer_id
               order by order_date desc
           ) as rn
    from orders
)
select *
from ranked
where rn = 1;
```

### Pattern to memorize

```text
ROW_NUMBER()
PARTITION BY group
ORDER BY date DESC
WHERE rn = 1
```

This pattern is extremely common in data engineering.

---

## #48 Top 3 highest-paid employees per department

```sql
with ranked as (
    select *,
           rank() over (
               partition by department
               order by salary desc
           ) as rnk
    from employees
)
select *
from ranked
where rnk <= 3;
```

### `RANK()` vs `ROW_NUMBER()`

If ties should all be included:

```sql
rank()
```

If exactly 3 rows are required:

```sql
row_number()
```

---

## #49 Compare product price with category average

```sql
select product_id,
       category,
       price,
       price -
       avg(price) over (
           partition by category
       ) as diff_from_avg
from products;
```

---

## #50 Percentage of running total

```sql
select order_date,
       amount,
       sum(amount) over (
           order by order_date
       ) * 100.0 /
       sum(amount) over () as pct_of_total_so_far
from orders;
```

---

## #51 Users active for 3+ consecutive days

### Concept

Gaps and islands.

```sql
with numbered as (
    select user_id,
           login_date,
           login_date -
           (
               row_number() over (
                   partition by user_id
                   order by login_date
               )
           )::int as grp
    from logins
)
select user_id,
       count(*) as consecutive_days
from numbered
group by user_id, grp
having count(*) >= 3;
```

### Important concept

This is a classic:

> **Gaps and Islands problem**

The technique groups consecutive dates into the same "island".

---

# Section 5 — Subqueries & CTEs #52–61

CTEs make complicated SQL easier to understand.

Basic structure:

```sql
with cte_name as (
    select ...
)
select ...
from cte_name;
```

---

## #52 Employees earning above their department average

```sql
select name,
       salary
from employees e1
where salary > (
    select avg(salary)
    from employees e2
    where e2.department = e1.department
);
```

### Concept

Correlated subquery.

The inner query depends on the current row of the outer query.

---

## #53 Customers with at least one order above $1,000

```sql
select name
from customers c
where exists (
    select 1
    from orders o
    where o.customer_id = c.customer_id
      and o.amount > 1000
);
```

### Concept

`EXISTS`

Checks whether at least one matching row exists.

---

## #54 Month-over-month revenue using a CTE

```sql
with monthly as (
    select date_trunc('month', order_date) as mo,
           sum(amount) as revenue
    from orders
    group by 1
)
select mo,
       revenue,
       lag(revenue) over (
           order by mo
       ) as prev_month
from monthly;
```

### Why use a CTE?

First calculate monthly revenue.

Then perform the window calculation.

This makes the logic easier to read.

---

## #55 Full management hierarchy

### Recursive CTE

```sql
with recursive org as (
    select employee_id,
           name,
           manager_id,
           1 as level
    from employees
    where manager_id is null

    union all

    select e.employee_id,
           e.name,
           e.manager_id,
           o.level + 1
    from employees e
    join org o
        on e.manager_id = o.employee_id
)
select *
from org
order by level;
```

### Concept

Recursive CTEs are useful for hierarchical data such as:

```text
CEO
 ├── Manager
 │    ├── Employee
 │    └── Employee
 └── Manager
      └── Employee
```

---

## #56 Customers whose total spend exceeds average customer spend

```sql
with totals as (
    select customer_id,
           sum(amount) as total
    from orders
    group by customer_id
)
select *
from totals
where total > (
    select avg(total)
    from totals
);
```

### Pattern

1. Calculate total per customer.
2. Calculate average of those totals.
3. Return customers above that average.

---

## #57 Products never ordered using NOT EXISTS

```sql
select name
from products p
where not exists (
    select 1
    from order_items oi
    where oi.product_id = p.product_id
);
```

### Important

`NOT EXISTS` is another powerful anti-join technique.

---

## #58 Top 3 departments by average salary

```sql
with dept_avg as (
    select department,
           avg(salary) as avg_sal
    from employees
    group by department
)
select *
from dept_avg
order by avg_sal desc
limit 3;
```

---

## #59 Filter using a subquery in FROM

```sql
select *
from (
    select *,
           rank() over (
               partition by department
               order by salary desc
           ) as rnk
    from employees
) t
where rnk <= 3;
```

### Concept

A subquery in the `FROM` clause acts like a temporary table.

---

## #60 Customers who ordered in January but not February

```sql
select distinct customer_id
from orders
where order_date between '2026-01-01' and '2026-01-31'
  and customer_id not in (
      select customer_id
      from orders
      where order_date between '2026-02-01' and '2026-02-28'
  );
```

### Concept

Set exclusion using `NOT IN`.

### Important

Be careful with `NOT IN` when the subquery can contain `NULL`.

In many production situations, `NOT EXISTS` is safer.

---

## #61 Chain two CTEs

```sql
with totals as (
    select customer_id,
           sum(amount) as total
    from orders
    group by customer_id
),
ranked as (
    select *,
           rank() over (
               order by total desc
           ) as rnk
    from totals
)
select *
from ranked
where rnk <= 5;
```

### Pattern

```text
Raw data
   ↓
CTE 1: aggregate
   ↓
CTE 2: rank
   ↓
Final filter
```

---

# Section 6 — Advanced / Business Scenarios #62–75

These questions are closer to the type of SQL used in analytics and real-world data engineering.

---

## #62 Bucket customers into age groups

```sql
select customer_id,
       case
           when age < 25 then '18-24'
           when age < 35 then '25-34'
           when age < 50 then '35-49'
           else '50+'
       end as age_bucket
from customers;
```

### Concept

`CASE WHEN`

Extremely important for data transformation.

---

## #63 Random sample of 1,000 customers

```sql
select *
from customers
order by random()
limit 1000;
```

> PostgreSQL syntax. Other databases use different random functions.

---

## #64 Cohort retention

```sql
with first_month as (
    select user_id,
           date_trunc('month', min(activity_date)) as cohort_month
    from activity
    group by user_id
)
select f.cohort_month,
       count(distinct a.user_id) as active_users
from first_month f
join activity a
    on f.user_id = a.user_id
where date_trunc('month', a.activity_date)
      = f.cohort_month + interval '1 month'
group by f.cohort_month;
```

### Concept

Cohort analysis.

Users are grouped based on when they first became active, then their later activity is measured.

---

## #65 Z-score of customer revenue

```sql
select customer_id,
       revenue,
       (
           revenue - avg(revenue) over ()
       ) /
       nullif(
           stddev(revenue) over (),
           0
       ) as revenue_zscore
from customers;
```

### Formula

```text
Z = (value - mean) / standard deviation
```

Useful for identifying unusually high or low values.

---

## #66 One-hot-style purchase categories

```sql
select customer_id,
       max(
           case
               when category = 'Electronics' then 1
               else 0
           end
       ) as bought_electronics,
       max(
           case
               when category = 'Clothing' then 1
               else 0
           end
       ) as bought_clothing
from orders
group by customer_id;
```

### Example output

```text
customer_id | bought_electronics | bought_clothing
------------|--------------------|----------------
101         | 1                  | 0
102         | 1                  | 1
103         | 0                  | 1
```

### Concept

Conditional aggregation can be used to create feature columns.

---

## #67 Products frequently bought together

### Market Basket Analysis

```sql
select a.product_id as product_a,
       b.product_id as product_b,
       count(*) as times_together
from order_items a
join order_items b
    on a.order_id = b.order_id
   and a.product_id < b.product_id
group by a.product_id,
         b.product_id
order by times_together desc;
```

### Why `a.product_id < b.product_id`?

Prevents duplicate pairs:

```text
A - B
B - A
```

Only:

```text
A - B
```

is retained.

---

## #68 Missing-data audit

```sql
select
    sum(
        case
            when age is null then 1
            else 0
        end
    ) as missing_age,

    sum(
        case
            when income is null then 1
            else 0
        end
    ) as missing_income,

    count(*) as total_rows
from customers;
```

### Concept

Data-quality auditing.

This pattern is useful when working with messy datasets.

---

## #69 Flag outliers using IQR

### Step 1 — Calculate Q1 and Q3

```sql
with stats as (
    select *,
           percentile_cont(0.25)
               within group (order by revenue)
               over () as q1,

           percentile_cont(0.75)
               within group (order by revenue)
               over () as q3
    from customer_revenue
)
```

### Step 2 — Identify outliers

```sql
select *,
       (
           revenue < q1 - 1.5 * (q3 - q1)
           or
           revenue > q3 + 1.5 * (q3 - q1)
       ) as is_outlier
from stats;
```

### IQR formula

```text
IQR = Q3 - Q1

Lower bound = Q1 - 1.5 × IQR

Upper bound = Q3 + 1.5 × IQR
```

Values outside these bounds are considered outliers.

---

## #70 Classify customers as New, Returning or Churned

```sql
with stats as (
    select customer_id,
           min(order_date) as first_order,
           max(order_date) as last_order
    from orders
    group by customer_id
)
select customer_id,
       case
           when first_order >= current_date - interval '30 days'
               then 'New'

           when last_order < current_date - interval '90 days'
               then 'Churned'

           else 'Returning'
       end as status
from stats;
```

### Concept

Business-rule classification using `CASE`.

---

## #71 Monthly DAU/MAU stickiness ratio

### DAU

Daily Active Users.

```sql
with dau as (
    select date_trunc('month', activity_date) as mo,
           activity_date,
           count(distinct user_id) as d
    from activity
    group by 1, 2
),
mau as (
    select date_trunc('month', activity_date) as mo,
           count(distinct user_id) as m
    from activity
    group by 1
)
select d.mo,
       avg(d.d) * 1.0 / max(mau.m) as stickiness
from dau d
join mau
    on d.mo = mau.mo
group by d.mo;
```

### Concept

Measures how frequently users engage with a product.

---

## #72 Find dates with zero activity

```sql
with all_dates as (
    select generate_series(
        min(activity_date),
        max(activity_date),
        '1 day'
    ) as dt
    from attendance
)
select dt
from all_dates
where dt not in (
    select distinct activity_date
    from attendance
);
```

### Concept

Generate a complete date range and compare it against actual activity.

Useful for detecting missing dates.

> `generate_series()` is PostgreSQL-specific.

---

## #73 Pivot monthly rows into columns

```sql
select product_id,
       sum(
           case
               when month = 'Jan' then amount
               else 0
           end
       ) as jan,

       sum(
           case
               when month = 'Feb' then amount
               else 0
           end
       ) as feb
from sales
group by product_id;
```

### Concept

Manual pivot using conditional aggregation.

---

## #74 A/B test click-through rate

```sql
select variant,
       sum(clicked) * 1.0 / count(*) as ctr
from experiment_events
group by variant;
```

### Formula

```text
CTR = clicks / total events
```

The `* 1.0` ensures decimal division in databases where integer division would otherwise occur.

---

## #75 Friend-of-friend suggestions

```sql
select f2.friend_id as suggested,
       count(*) as mutual_friends
from friends f1
join friends f2
    on f1.friend_id = f2.user_id
where f1.user_id = 123
  and f2.friend_id != 123
  and f2.friend_id not in (
      select friend_id
      from friends
      where user_id = 123
  )
group by f2.friend_id
order by mutual_friends desc;
```

### Concept

Graph-style SQL.

The query:

1. Finds the user's friends.
2. Finds friends of those friends.
3. Removes the original user.
4. Removes people already connected.
5. Counts mutual connections.
6. Ranks suggestions.

---

# 🧠 Self-Score

After attempting all 75 questions **without looking at the answers**, calculate your score.

| Score | Meaning |
|---:|---|
| 60–75 | Interview-ready across most SQL levels |
| 40–59 | Strong fundamentals; improve windows and advanced SQL |
| 20–39 | Core querying is developing; revise joins and aggregation |
| Under 20 | Start with fundamentals and work upward |

### Recommended rule

Do not count a question as correct simply because you recognized the answer.

Count it as correct only if you can:

- Understand the requirement.
- Choose the correct SQL concept.
- Write the query yourself.
- Explain the query.
- Handle common edge cases.

---

# 🔥 Interview Revision Checklist

## Basics

Before an interview, you should be able to write these without hesitation:

```text
SELECT
WHERE
DISTINCT
ORDER BY
LIMIT
LIKE
IN
BETWEEN
IS NULL
CASE
```

---

## Joins

You should understand:

```text
INNER JOIN
LEFT JOIN
RIGHT JOIN
FULL OUTER JOIN
SELF JOIN
ANTI JOIN
```

Especially memorize:

### Find records with no match

```sql
select a.*
from table_a a
left join table_b b
    on a.id = b.id
where b.id is null;
```

---

# Aggregation

Know:

```text
COUNT
COUNT(DISTINCT)
SUM
AVG
MIN
MAX
GROUP BY
HAVING
```

Most important distinction:

```text
WHERE  → filters rows
HAVING → filters groups
```

---

# Window Functions

Know these extremely well:

```text
ROW_NUMBER()
RANK()
DENSE_RANK()
LAG()
LEAD()
NTILE()
FIRST_VALUE()
LAST_VALUE()
PERCENT_RANK()
```

And understand:

```sql
over (
    partition by ...
    order by ...
)
```

---

# 🏆 Most Important Interview Patterns

## 1. Latest record per group

```sql
with ranked as (
    select *,
           row_number() over (
               partition by customer_id
               order by order_date desc
           ) as rn
    from orders
)
select *
from ranked
where rn = 1;
```

---

## 2. Top N per group

```sql
with ranked as (
    select *,
           rank() over (
               partition by department
               order by salary desc
           ) as rnk
    from employees
)
select *
from ranked
where rnk <= 3;
```

---

## 3. Find duplicates

```sql
select email,
       count(*)
from users
group by email
having count(*) > 1;
```

---

## 4. Find records without a match

```sql
select a.*
from table_a a
left join table_b b
    on a.id = b.id
where b.id is null;
```

---

## 5. Running total

```sql
select order_date,
       amount,
       sum(amount) over (
           order by order_date
       ) as running_total
from orders;
```

---

## 6. Previous row

```sql
lag(value) over (
    order by date
)
```

---

## 7. Next row

```sql
lead(value) over (
    order by date
)
```

---

## 8. Conditional aggregation

```sql
sum(
    case
        when status = 'completed' then 1
        else 0
    end
)
```

---

## 9. Compare against group average

```sql
select *,
       avg(salary) over (
           partition by department
       ) as dept_avg
from employees;
```

---

## 10. CTE pipeline

Think of CTEs as multiple logical steps:

```text
Raw data
   ↓
CTE 1
   ↓
CTE 2
   ↓
CTE 3
   ↓
Final result
```

---

# ⚠️ SQL Interview Traps

## `NULL`

Wrong:

```sql
where phone = null
```

Correct:

```sql
where phone is null
```

---

## `COUNT(*)` vs `COUNT(column)`

```sql
count(*)
```

Counts rows.

```sql
count(column)
```

Ignores `NULL`.

---

## `WHERE` vs `HAVING`

Wrong idea:

```sql
where count(*) > 5
```

Use:

```sql
having count(*) > 5
```

---

## `RANK()` vs `DENSE_RANK()`

```text
RANK       → gaps after ties
DENSE_RANK → no gaps
```

---

## `ROW_NUMBER()` vs `RANK()`

```text
ROW_NUMBER → unique sequential number
RANK       → same rank for ties
```

---

## `UNION` vs `UNION ALL`

```text
UNION      → removes duplicates
UNION ALL  → keeps duplicates
```

When duplicates do not need to be removed, `UNION ALL` is generally preferable.

---

# 📈 Recommended Practice Order

Do not randomly solve the questions.

Use this progression:

```text
#1–12
Basics
   ↓
#13–24
Joins
   ↓
#25–36
Aggregation
   ↓
#37–51
Window Functions
   ↓
#52–61
Subqueries + CTEs
   ↓
#62–75
Business Scenarios
```

---

# 🎯 How to Practice This Repository

For every question:

### Step 1

Read only the problem.

### Step 2

Identify the concept.

Example:

```text
"latest order per customer"
        ↓
ROW_NUMBER()
        ↓
PARTITION BY customer_id
        ↓
ORDER BY order_date DESC
```

### Step 3

Write your own query.

### Step 4

Run it against sample data.

### Step 5

Compare your solution with the answer.

### Step 6

Explain your query in plain English.

If you cannot explain it, you don't fully understand it yet.

---

# 🚀 Final Interview Checklist

Before an SQL interview, make sure you can solve these without searching:

- [ ] Find duplicates
- [ ] Find missing records
- [ ] Find customers with no orders
- [ ] Find second-highest salary
- [ ] Find Nth-highest salary
- [ ] Find top N per department
- [ ] Find latest record per customer
- [ ] Calculate running total
- [ ] Calculate moving average
- [ ] Compare current row with previous row
- [ ] Compare current row with next row
- [ ] Calculate department average
- [ ] Find employees above department average
- [ ] Use `CASE WHEN`
- [ ] Use `GROUP BY` and `HAVING`
- [ ] Perform multi-table joins
- [ ] Write a self join
- [ ] Write an anti join
- [ ] Use CTEs
- [ ] Use recursive CTEs
- [ ] Solve consecutive-date problems
- [ ] Perform conditional aggregation
- [ ] Handle `NULL`
- [ ] Explain `ROW_NUMBER`, `RANK`, and `DENSE_RANK`

---

# 💡 Final Takeaway

The important thing about these 75 questions is **not memorizing 75 separate SQL queries**.

Most of them are combinations of a relatively small number of patterns:

```text
Filtering
   +
Joins
   +
Aggregation
   +
Window Functions
   +
Subqueries
   +
CTEs
   +
CASE
```

If you become comfortable with those building blocks, you can solve many SQL problems that look completely different on the surface.

The highest-priority concepts for interviews are:

**JOINs → GROUP BY/HAVING → Window Functions → CTEs → Subqueries → Real-world business logic**

Use this README as both a **revision sheet** and a **hands-on SQL practice set**.