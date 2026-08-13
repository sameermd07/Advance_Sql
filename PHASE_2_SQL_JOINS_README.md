# 🔵 PHASE 2 — SQL JOINs

> **Goal:** Understand how to combine related tables correctly and, more importantly, choose the right JOIN based on which records you need to preserve.

---

## 🎯 What You Should Be Able to Do

By the end of this phase, you should be comfortable with:

```text
INNER JOIN
LEFT JOIN
RIGHT JOIN
FULL OUTER JOIN
SELF JOIN
CROSS JOIN
SEMI JOIN
ANTI JOIN
```

But the real target is not memorizing definitions.

The important question is:

> **"What records do I want to preserve?"**

---

# 1. Why JOINs Exist

In a relational database, data is usually split across multiple tables.

For example:

```text
customers
--------------------------------
customer_id
customer_name
city
customer_type
```

and:

```text
orders
--------------------------------
order_id
customer_id
order_date
status
total_amount
```

Suppose we want:

```text
customer_name
order_id
total_amount
```

The customer name exists in `customers`.

The order information exists in `orders`.

The common key is:

```text
customers.customer_id
        =
orders.customer_id
```

A JOIN combines the related records.

---

# 2. Understand the Table Grain Before Joining

This is one of the most important Data Engineering concepts in this phase.

Think about what **one row represents** in each table.

```text
customers
→ 1 row per customer

orders
→ 1 row per order

order_items
→ 1 row per order item

payments
→ potentially 1 or more rows per order
```

Before writing a JOIN, ask:

> **What does one row represent in each table?**

If you do not understand the grain, you can write a perfectly valid JOIN that produces incorrect business results.

---

# 3. INNER JOIN

## Meaning

> Return rows where a matching record exists in both tables.

Example:

```text
customers:

A
B
C
D

orders:

A
B
D
E
```

An INNER JOIN returns:

```text
A
B
D
```

Customer `C` is excluded because there is no matching order.

Order `E` is excluded because there is no matching customer.

---

## Syntax

```sql
select
    c.customer_id,
    c.customer_name,
    o.order_id,
    o.total_amount
from customers c
inner join orders o
    on c.customer_id = o.customer_id;
```

### Result

You get customers who have matching orders.

### Think:

```text
INNER JOIN
    ↓
matching records only
```

---

# 4. LEFT JOIN 🔥

## Meaning

> Keep everything from the left table and bring matching records from the right table.

Example:

```text
customers:

A
B
C
D

orders:

A
B
D
E
```

A LEFT JOIN keeps:

```text
A
B
C
D
```

Customer `C` remains even though there is no matching order.

The order columns become NULL.

---

## Example

```sql
select
    c.customer_id,
    c.customer_name,
    o.order_id,
    o.total_amount
from customers c
left join orders o
    on c.customer_id = o.customer_id;
```

If customer `113` has no order:

```text
customer_id | customer_name | order_id | total_amount
-------------|---------------|----------|-------------
113          | raj malhotra  | null     | null
```

### Remember

```text
LEFT JOIN
→ Don't lose my left table.
```

---

# 5. LEFT JOIN for Data Quality 🔥

This is where JOINs become especially useful for Data Engineering.

Requirement:

> Find customers who have never placed an order.

Use:

```sql
select
    c.customer_id,
    c.customer_name
from customers c
left join orders o
    on c.customer_id = o.customer_id
where o.order_id is null;
```

### How it works

First:

```text
customers
    ↓
LEFT JOIN
    ↓
orders
```

For customers without a matching order:

```text
order columns = NULL
```

Then:

```sql
where o.order_id is null
```

keeps only those unmatched customers.

This is commonly called an:

> **ANTI JOIN PATTERN**

Useful for:

```text
missing records
unprocessed records
orphan records
source vs target comparison
```

---

# 6. RIGHT JOIN

RIGHT JOIN is essentially the opposite of LEFT JOIN.

```sql
select
    c.customer_name,
    o.order_id
from customers c
right join orders o
    on c.customer_id = o.customer_id;
```

Meaning:

> Keep everything from `orders`.

However, many developers prefer rewriting RIGHT JOIN as LEFT JOIN because LEFT JOIN is often easier to read.

Instead of:

```text
customers
RIGHT JOIN orders
```

you can write:

```text
orders
LEFT JOIN customers
```

Same preservation idea.

### Recommendation

Know RIGHT JOIN, but become especially strong with LEFT JOIN.

---

# 7. FULL OUTER JOIN

## Meaning

> Return everything from both tables, whether or not a match exists.

Suppose:

```text
customers:

101
102
103

orders:

101
102
104
```

A FULL OUTER JOIN conceptually gives:

```text
101
102
103
104
```

The unmatched side contains NULL values.

---

## MySQL Note

MySQL does not directly support `FULL OUTER JOIN`.

One common approach is to combine LEFT and RIGHT JOIN results with `UNION`.

Conceptually:

```sql
select ...
from customers c
left join orders o
    on c.customer_id = o.customer_id

union

select ...
from customers c
right join orders o
    on c.customer_id = o.customer_id;
```

The exact implementation depends on the columns and duplicate-handling requirements.

---

# 8. SELF JOIN

A SELF JOIN means:

> A table is joined with itself.

Consider:

```text
employees

employee_id | employee_name | manager_id
------------|---------------|-----------
1           | rahul         | null
2           | priya         | 1
3           | arjun         | 1
4           | sneha         | 2
```

Requirement:

> Display employee name and manager name.

The same table represents two roles:

```text
employees e
→ employee

employees m
→ manager
```

Query:

```sql
select
    e.employee_name as employee,
    m.employee_name as manager
from employees e
left join employees m
    on e.manager_id = m.employee_id;
```

### Key idea

Same table.

Different aliases.

Different logical roles.

---

# 9. CROSS JOIN

A CROSS JOIN creates every possible combination between two tables.

Suppose:

```text
colors

red
blue
```

and:

```text
sizes

S
M
L
```

CROSS JOIN produces:

```text
red   S
red   M
red   L
blue  S
blue  M
blue  L
```

Query:

```sql
select
    c.customer_name,
    p.product_name
from customers c
cross join products p;
```

If there are:

```text
15 customers
12 products
```

the result can contain:

```text
15 × 12 = 180 rows
```

### ⚠️ Warning

CROSS JOIN can become extremely large.

For example:

```text
100,000 rows × 100,000 rows
```

can create a massive result.

Use it intentionally.

---

# 🔥 10. JOIN Multiplication

This is one of the most important concepts for Data Engineers.

Suppose customer `101` has:

```text
3 orders
```

When you join customers to orders:

```text
customer 101
    ↓
3 matching order rows
```

Customer `101` appears three times.

That is not automatically a problem.

It is the correct result because the relationship is:

```text
1 customer
    ↓
many orders
```

---

# 11. Multi-Level JOIN Multiplication

Suppose:

```text
1 customer
    ↓
many orders
    ↓
many order_items
```

Example:

```text
customer 101
order 1001
item 1

customer 101
order 1001
item 2

customer 101
order 1001
item 3

customer 101
order 1001
item 4
```

The customer/order information repeats because there are multiple item rows.

This is called:

> **Row multiplication**

or:

> **Duplicate amplification**

---

# 🚨 12. The JOIN Duplication Problem

Suppose:

```text
orders

order_id | total_amount
---------|-------------
1001     | 10000
```

And order `1001` has 3 items.

After joining with `order_items`, you may see:

```text
order_id | total_amount
---------|-------------
1001     | 10000
1001     | 10000
1001     | 10000
```

If you then run:

```sql
sum(o.total_amount)
```

you can get:

```text
30000
```

instead of:

```text
10000
```

The JOIN did not necessarily fail.

The problem is that the query changed the number of rows representing the order.

### Important Data Engineering rule

Before aggregating after a JOIN:

> **Understand the grain of the result.**

---

# 13. SEMI JOIN

A SEMI JOIN means:

> Return records from table A where a matching record exists in table B.

A common SQL pattern uses `exists`.

Requirement:

> Find customers who have placed at least one order.

```sql
select
    c.customer_id,
    c.customer_name
from customers c
where exists (
    select 1
    from orders o
    where o.customer_id = c.customer_id
);
```

Notice:

We are not selecting order columns.

We only ask:

> Does a matching order exist?

This is the idea of a SEMI JOIN.

---

# 14. ANTI JOIN

An ANTI JOIN is the opposite idea.

> Return records from A where no matching record exists in B.

Using `not exists`:

```sql
select
    c.customer_id,
    c.customer_name
from customers c
where not exists (
    select 1
    from orders o
    where o.customer_id = c.customer_id
);
```

This finds customers who never placed an order.

---

# 🧠 15. SEMI JOIN vs ANTI JOIN

Remember:

```text
EXISTS
   ↓
matching record exists
   ↓
SEMI JOIN

NOT EXISTS
   ↓
matching record does not exist
   ↓
ANTI JOIN
```

---

# 🔥 16. LEFT JOIN + WHERE Trap

This is a very important interview and Data Engineering concept.

Suppose the requirement is:

> Find all customers and their delivered orders.

A tempting query is:

```sql
select
    c.customer_name,
    o.order_id,
    o.status
from customers c
left join orders o
    on c.customer_id = o.customer_id
where o.status = 'delivered';
```

The problem:

```sql
where o.status = 'delivered'
```

removes rows where `o.status` is NULL.

Therefore, customers with no delivered order disappear.

Your LEFT JOIN can effectively behave like an INNER JOIN for that condition.

---

## Better approach

Put the condition into the JOIN:

```sql
select
    c.customer_name,
    o.order_id,
    o.status
from customers c
left join orders o
    on c.customer_id = o.customer_id
    and o.status = 'delivered';
```

Now:

> Keep every customer, but only attach delivered orders.

### Memorize this distinction

```text
Condition in WHERE
→ filters the final rows

Condition in LEFT JOIN ... ON
→ controls which right-side rows are matched
   while preserving the left table
```

---

# 🧪 17. Phase 2 Practice

> **Do not immediately look for solutions. Try to write the queries yourself first.**

---

## 🟢 Level 1 — Basic JOINs

### Q1

Display:

```text
customer_name
order_id
order_date
total_amount
```

for all customers who have placed orders.

**Concepts:**

```text
INNER JOIN
```

---

### Q2

Display all customers, including customers who have never placed an order.

Return:

```text
customer_id
customer_name
order_id
```

**Concepts:**

```text
LEFT JOIN
```

---

### Q3

Find customers who have never placed an order.

**Concepts:**

```text
LEFT JOIN
IS NULL
```

or:

```text
NOT EXISTS
```

---

### Q4

Display:

```text
customer_name
customer_type
order_id
status
total_amount
```

for all orders.

Choose the JOIN based on the records you need to preserve.

---

# 🟡 18. Level 2 — Multiple Tables

### Q5

Display:

```text
order_id
customer_name
product_name
quantity
unit_price
```

by joining:

```text
customers
orders
order_items
products
```

Think about the relationship:

```text
customers
    ↓
orders
    ↓
order_items
    ↓
products
```

---

### Q6

Find:

> Total revenue generated by each customer.

Return:

```text
customer_id
customer_name
total_revenue
```

⚠️ Think carefully about the grain before aggregating.

---

### Q7

Find:

> Total quantity of products sold by each product.

Return:

```text
product_id
product_name
total_quantity
```

---

# 🟠 19. Level 3 — Data Quality

### Q8

Find orders that:

> Have no payment record.

Hint:

```text
orders
LEFT JOIN
payments
```

Then identify unmatched payments.

---

### Q9

Find payments that:

> Do not have a corresponding order.

Try an anti-join pattern.

---

### Q10

Find orders that:

> Have no shipment record.

Again, think:

```text
orders
LEFT JOIN
shipments
WHERE shipment is NULL
```

---

# 🔴 20. Level 4 — JOIN Multiplication

### Q11

Calculate the total order amount from `order_items`.

Formula:

```text
quantity × unit_price ×
(1 - discount_percent / 100)
```

Return:

```text
order_id
calculated_amount
```

Then compare it against:

```text
orders.total_amount
```

---

### Q12 🔥

Find orders where:

```text
calculated order amount
        !=
orders.total_amount
```

This is a real data-quality validation pattern.

---

# 🔥 21. Level 5 — Real Data Engineering

### Q13

Imagine:

```text
source_orders
target_orders
```

Find records that exist in the source but not in the target.

This is a common incremental-pipeline validation problem.

Think:

```text
source
  ↓
LEFT JOIN target
  ↓
target key IS NULL
```

---

### Q14

Find records that exist in both source and target but have a different:

```text
amount
status
```

This moves from:

> "Does the record exist?"

to:

> "Does the target contain the correct version of the record?"

---

### Q15 🔥

Find customers who:

- have at least one delivered order
- have never cancelled an order
- have total delivered revenue > ₹10,000

This combines:

```text
JOIN
GROUP BY
HAVING
CASE / conditional aggregation
```

---

# 💻 22. LeetCode JOIN Practice

After completing the scenarios, practice:

### 1378 — Replace Employee ID With The Unique Identifier

https://leetcode.com/problems/replace-employee-id-with-the-unique-identifier/

Focus:

```text
INNER / LEFT JOIN
```

### 1068 — Product Sales Analysis I

https://leetcode.com/problems/product-sales-analysis-i/

Focus:

```text
JOIN
filtering
```

### 1581 — Customer Who Visited but Did Not Make Any Transactions

https://leetcode.com/problems/customer-who-visited-but-did-not-make-any-transactions/

Focus:

```text
LEFT JOIN
ANTI JOIN pattern
```

### 1661 — Average Time of Process per Machine

https://leetcode.com/problems/average-time-of-process-per-machine/

Focus:

```text
SELF JOIN / aggregation concepts
```

### 577 — Employee Bonus

https://leetcode.com/problems/employee-bonus/

Focus:

```text
LEFT JOIN
NULL handling
```

### 1280 — Students and Examinations

https://leetcode.com/problems/students-and-examinations/

Focus:

```text
CROSS JOIN
LEFT JOIN
aggregation
```

### 570 — Managers with at Least 5 Direct Reports

https://leetcode.com/problems/managers-with-at-least-5-direct-reports/

Focus:

```text
SELF JOIN
GROUP BY
HAVING
```

---

# 🧠 23. JOIN Decision Cheat Sheet

Don't memorize JOIN definitions only.

Ask:

> **What records do I want to preserve?**

```text
Want matching records only?
        ↓
INNER JOIN

Want EVERYTHING from left?
        ↓
LEFT JOIN

Want EVERYTHING from right?
        ↓
RIGHT JOIN

Want everything from both?
        ↓
FULL OUTER JOIN

Want unmatched records?
        ↓
LEFT JOIN + IS NULL
or
NOT EXISTS

Want to check whether something exists?
        ↓
EXISTS

Want records where something does NOT exist?
        ↓
NOT EXISTS

Want every possible combination?
        ↓
CROSS JOIN

Want a table related to itself?
        ↓
SELF JOIN
```

---

# 🔥 24. The Most Important Data Engineering Rule

Before joining tables:

```text
1. Understand the grain
        ↓
2. Identify the join key
        ↓
3. Understand relationship
        ↓
4. Predict row count
        ↓
5. JOIN
        ↓
6. Validate row count
        ↓
7. Aggregate only after understanding multiplication
```

Example:

```text
customers
1 row/customer

orders
1 row/order

order_items
1 row/item
```

Relationship:

```text
1 customer
    ↓
many orders
    ↓
many order_items
```

Therefore:

```text
customers
    ↓
JOIN orders
    ↓
more rows
    ↓
JOIN order_items
    ↓
even more rows
```

This is normal.

The danger is aggregating a value from a higher-grain table after it has been multiplied.

---

# 🎯 25. Phase 2 Final Checklist

Before considering this phase complete:

### JOIN Fundamentals

- [ ] Explain INNER JOIN
- [ ] Explain LEFT JOIN
- [ ] Explain RIGHT JOIN
- [ ] Explain FULL OUTER JOIN
- [ ] Explain SELF JOIN
- [ ] Explain CROSS JOIN

### Data Engineering JOINs

- [ ] LEFT JOIN + IS NULL
- [ ] EXISTS
- [ ] NOT EXISTS
- [ ] Source vs target comparison
- [ ] Missing-record detection
- [ ] Orphan-record detection

### Advanced Understanding

- [ ] Understand table grain
- [ ] Understand one-to-many relationships
- [ ] Understand row multiplication
- [ ] Identify duplicate amplification
- [ ] Understand the LEFT JOIN + WHERE trap
- [ ] Validate row counts after JOINs

---

# 🏆 Phase 2 Interview Questions

### Q1. What is the difference between INNER JOIN and LEFT JOIN?

> INNER JOIN returns only matching rows. LEFT JOIN keeps every row from the left table and adds matching rows from the right table.

### Q2. How do you find records in one table that don't exist in another?

Use an anti-join pattern:

```sql
select ...
from source s
left join target t
    on s.id = t.id
where t.id is null;
```

or:

```sql
select ...
from source s
where not exists (
    select 1
    from target t
    where t.id = s.id
);
```

### Q3. What is JOIN multiplication?

> When a JOIN changes the number of rows representing an entity because one row on one side matches multiple rows on the other side.

### Q4. Why can a LEFT JOIN behave like an INNER JOIN?

Because filtering a right-table column in the `where` clause removes NULL unmatched rows.

### Q5. How can you avoid incorrect aggregation after a JOIN?

> Understand the grain of each table, determine the expected cardinality, aggregate at the correct grain, and validate row counts.

### Q6. What is a SEMI JOIN?

> Return rows from one table where a matching row exists in another table, commonly expressed using `exists`.

### Q7. What is an ANTI JOIN?

> Return rows from one table where no matching row exists in another table, commonly expressed using `not exists` or `left join ... is null`.

---

# ⭐ Final Phase 2 Takeaway

The goal is not:

> "I know six JOIN types."

The goal is:

> **"I know which records I need, which table's rows I need to preserve, what the relationship is, and how the JOIN will affect row counts."**

A strong Data Engineer thinks:

```text
What is the grain?
       ↓
What is the relationship?
       ↓
Which records should survive?
       ↓
Which JOIN should I use?
       ↓
Will rows multiply?
       ↓
Can my aggregation become incorrect?
       ↓
How do I validate the result?
```

Once you think this way, JOINs become much more than syntax.
