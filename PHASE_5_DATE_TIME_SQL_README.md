# 🔥 PHASE 5 — SQL Date & Time

> **Goal:** Become comfortable working with dates and timestamps for analytics, data validation, incremental loading, retention analysis, and real Data Engineering pipelines.

---

# 🎯 Why Date & Time SQL Matters

Date/time logic appears everywhere in Data Engineering.

Typical requirements include:

```text
records from the last 24 hours
records older than 7 days
records from today
records from the current month
previous month's revenue
days between orders
first order
latest order
late-arriving records
incremental records
```

This phase connects SQL directly to pipeline scenarios such as:

```text
ADF
ADLS
incremental loading
daily processing
data retention
pipeline validation
```

---

# 1. DATE vs DATETIME / TIMESTAMP

A date represents a calendar date:

```text
2026-08-13
```

A timestamp contains date + time:

```text
2026-08-13 15:30:45
```

Conceptually:

```text
DATE
→ year + month + day

DATETIME / TIMESTAMP
→ date + time
```

Choose the type based on what the business requirement needs.

---

# 2. CURRENT_DATE()

Returns the current date.

```sql
select current_date();
```

Example:

```text
2026-08-13
```

Useful for:

```text
today's records
date comparisons
daily processing
```

---

# 3. CURRENT_TIMESTAMP()

Returns the current date and time.

```sql
select current_timestamp();
```

Example:

```text
2026-08-13 15:30:45
```

Useful when the time component matters.

For example:

```text
last 24 hours
recently updated records
event processing
incremental loading
```

---

# 4. DATE()

If a timestamp contains time and you only need the date:

```sql
select
    date(order_date) as order_day
from orders;
```

Example:

```text
2026-08-13 14:25:10
```

becomes:

```text
2026-08-13
```

---

# 5. YEAR(), MONTH(), DAY()

You can extract parts of a date.

```sql
select
    year(order_date) as order_year,
    month(order_date) as order_month,
    day(order_date) as order_day
from orders;
```

For:

```text
2026-08-13
```

you get:

```text
year  = 2026
month = 8
day   = 13
```

Useful for:

```text
year-wise reports
month-wise reports
day-level analysis
```

---

# 6. EXTRACT()

Another common way to extract date components is:

```sql
select
    extract(year from order_date) as order_year,
    extract(month from order_date) as order_month
from orders;
```

The exact date syntax can vary across SQL engines, so always check the functions supported by your target database.

---

# 🔥 7. DATE Filtering

Suppose we want orders placed on a specific date.

For a DATE column:

```sql
select *
from orders
where order_date = '2026-08-13';
```

If `order_date` contains a timestamp, a range is often safer:

```sql
select *
from orders
where order_date >= '2026-08-13 00:00:00'
  and order_date < '2026-08-14 00:00:00';
```

This captures the entire day.

---

# 8. Why Range Filtering Matters

Suppose:

```text
order_date
2026-08-13 00:10:00
2026-08-13 12:30:00
2026-08-13 23:59:59
```

A condition based on exact equality against a date may not work as expected if the column contains time.

A half-open range:

```text
>= start
<
 next day
```

is a robust timestamp-filtering pattern.

Think:

```text
start_of_day
    ≤ timestamp
    <
start_of_next_day
```

---

# 🔥 9. Last 24 Hours

This is especially important for Data Engineering.

Requirement:

> Find records created in the last 24 hours.

Use the current timestamp and subtract one day:

```sql
select *
from orders
where created_at >= current_timestamp() - interval 1 day;
```

Conceptually:

```text
NOW
 ↓
subtract 24 hours
 ↓
start of window
```

Then return records inside that window.

---

# 10. Older Than 7 Days

Requirement:

> Identify records older than 7 days.

```sql
select *
from orders
where created_at < current_timestamp() - interval 7 day;
```

Conceptually:

```text
now
 ↓
minus 7 days
 ↓
anything before this point = older than 7 days
```

This is useful for:

```text
retention
cleanup
archival
stale-data detection
```

---

# 11. DATE_ADD()

Add a time interval to a date.

Example:

```sql
select
    date_add(order_date, interval 7 day) as date_after_7_days
from orders;
```

If:

```text
2026-08-13
```

then:

```text
2026-08-20
```

---

# 12. DATE_SUB()

Subtract an interval.

```sql
select
    date_sub(order_date, interval 7 day) as date_before_7_days
from orders;
```

Useful for:

```text
lookback windows
retention
historical comparisons
incremental boundaries
```

---

# 13. DATEDIFF()

`datediff()` calculates the difference between dates.

Example:

```sql
select
    datediff(
        '2026-08-13',
        '2026-08-10'
    ) as days_difference;
```

Result:

```text
3
```

---

# 14. Days Between Customer Orders

This connects directly with Phase 4.

Suppose we already calculated:

```text
previous_order_date
```

using `lag()`.

Then:

```sql
select
    customer_id,
    order_id,
    order_date,
    previous_order_date,
    datediff(
        order_date,
        previous_order_date
    ) as days_between_orders
from customer_orders;
```

The overall pattern becomes:

```text
LAG
 ↓
previous date
 ↓
DATEDIFF
 ↓
days between events
```

---

# 15. TIMESTAMPDIFF()

When you need a specific unit such as:

```text
second
minute
hour
day
month
year
```

you can use `timestampdiff()` where supported.

Example:

```sql
select
    timestampdiff(
        hour,
        created_at,
        processed_at
    ) as processing_hours
from pipeline_logs;
```

This can be useful for:

```text
pipeline duration
processing time
SLA measurement
event latency
```

---

# 🔥 16. Month-Wise Aggregation

Requirement:

> Calculate monthly revenue.

One approach:

```sql
select
    year(order_date) as order_year,
    month(order_date) as order_month,
    sum(total_amount) as monthly_revenue
from orders
group by
    year(order_date),
    month(order_date)
order by
    order_year,
    order_month;
```

Output conceptually:

```text
year | month | revenue
-----|-------|--------
2026 | 1     | 150000
2026 | 2     | 175000
2026 | 3     | 190000
```

---

# 17. Grouping by Month

For more advanced reporting, you may create a month key such as:

```text
2026-01
2026-02
2026-03
```

The exact implementation depends on your SQL engine.

The important idea is:

```text
timestamp
    ↓
month key
    ↓
GROUP BY
    ↓
monthly metric
```

---

# 🔥 18. Current Month

A simple approach is to compare year and month:

```sql
select *
from orders
where year(order_date) = year(current_date())
  and month(order_date) = month(current_date());
```

For large tables, be aware that applying functions to a column in a filter can affect index usage. A range-based filter can be preferable when performance matters.

---

# 19. First and Latest Order

Date/time functions become even more useful with aggregation and window functions.

### First order date

```sql
select
    customer_id,
    min(order_date) as first_order_date
from orders
group by customer_id;
```

### Latest order date

```sql
select
    customer_id,
    max(order_date) as latest_order_date
from orders
group by customer_id;
```

---

# 🔥 20. Customers Who Haven't Ordered in 90 Days

Requirement:

> Find customers whose latest order is older than 90 days.

First:

```sql
select
    customer_id,
    max(order_date) as latest_order_date
from orders
group by customer_id;
```

Then compare the latest order with the current date.

Conceptually:

```text
latest_order_date
        ↓
current_date - 90 days
        ↓
older?
        ↓
customer is inactive
```

This is a common retention/CRM analysis pattern.

---

# 🔥 21. Incremental Loading — Last 24 Hours

Suppose a pipeline runs every day.

You want to process records created after the previous processing boundary.

A simple time-window example:

```sql
select *
from orders
where created_at >= current_timestamp() - interval 1 day;
```

Conceptually:

```text
current time
    ↓
minus 24 hours
    ↓
incremental window
    ↓
extract records
```

In production pipelines, you often use a stored watermark such as:

```text
last_processed_timestamp
```

rather than always using the current time minus 24 hours.

---

# 22. Watermark Concept

A watermark records the last successfully processed point.

Example:

```text
last_processed_timestamp =
2026-08-12 18:00:00
```

Next pipeline run:

```sql
select *
from orders
where updated_at > '2026-08-12 18:00:00';
```

Conceptually:

```text
previous successful boundary
          ↓
        watermark
          ↓
new/updated records
```

This is the foundation of many incremental-loading patterns.

---

# ⚠️ 23. Why Watermarks Are Better Than Blind "Last 24 Hours"

Suppose your pipeline runs at:

```text
10:00 AM
```

and the previous run was:

```text
10:00 AM yesterday
```

A last-24-hours query assumes the processing window is exactly one day.

But real pipelines can have:

```text
delays
retries
failures
late-arriving data
long-running jobs
```

A watermark lets the pipeline remember the actual successful processing boundary.

---

# 🔥 24. Late-Arriving Records

Suppose an order belongs to:

```text
August 10
```

but arrives in the system on:

```text
August 13
```

This is a late-arriving record.

You can have:

```text
business date
    ≠
arrival/ingestion date
```

This distinction is very important in Data Engineering.

For example:

```text
order_date
created_at
ingested_at
updated_at
```

may all represent different concepts.

---

# 25. Business Time vs Processing Time

Think carefully about what each timestamp means.

```text
order_date
→ when the business event happened

created_at
→ when the record was created

updated_at
→ when the record was last changed

ingested_at
→ when the pipeline received the record

processed_at
→ when the pipeline processed it
```

Using the wrong timestamp can produce incorrect incremental loads or reports.

---

# 🔥 26. Missing Dates

Suppose expected daily data should contain:

```text
Aug 1
Aug 2
Aug 3
Aug 4
Aug 5
```

but your table contains:

```text
Aug 1
Aug 2
Aug 4
Aug 5
```

August 3 is missing.

This becomes a data-quality problem.

Conceptually:

```text
calendar dates
     ↓
LEFT JOIN actual dates
     ↓
actual date IS NULL
     ↓
missing date
```

The implementation can use a calendar/date dimension or generated date series depending on the SQL engine.

---

# 🔥 27. Date-Based Data Quality

You can ask:

```text
How many records arrived today?
How many arrived yesterday?
How many are older than 7 days?
How many are future-dated?
How many arrived late?
How many dates are missing?
```

Example:

```sql
select
    count(*) as total_records,
    count(
        case
            when date(created_at) = current_date()
            then 1
        end
    ) as today_records
from orders;
```

This combines:

```text
DATE
CURRENT_DATE
CASE
COUNT
```

---

# 🧪 28. Phase 5 Practice

> Try the questions yourself before checking solutions.

---

## 🟢 Level 1 — Basic Date Functions

### Q1

Display:

```text
order_id
order_date
order_year
order_month
order_day
```

Use:

```text
year()
month()
day()
```

---

### Q2

Find orders created today.

---

### Q3

Find orders created in the last 24 hours.

---

### Q4

Find records older than 7 days.

---

# 🟡 29. Level 2 — Date Differences

### Q5

Find the number of days between:

```text
order_date
delivery_date
```

Return:

```text
order_id
days_to_deliver
```

---

### Q6

Find customers whose latest order was more than 90 days ago.

---

### Q7

Find the average number of days between orders for each customer.

This combines:

```text
lag()
datediff()
avg()
group by
```

---

# 🟠 30. Level 3 — Monthly Analysis

### Q8

Calculate monthly revenue.

Return:

```text
year
month
revenue
```

---

### Q9

Find the highest-revenue month.

---

### Q10

Compare monthly revenue with the previous month.

This combines:

```text
monthly aggregation
+
lag()
```

---

# 🔴 31. Level 4 — Data Engineering

### Q11 — Last 24 Hours

Find all records created within the last 24 hours.

---

### Q12 — Older Than 7 Days

Find records older than 7 days.

---

### Q13 — Incremental Loading

Assume:

```text
last_processed_timestamp =
'2026-08-12 18:00:00'
```

Find records where:

```text
updated_at > last_processed_timestamp
```

---

### Q14 — Late Arriving Data

Assume:

```text
order_date
ingested_at
```

Find records where the record was ingested more than 2 days after the business order date.

---

### Q15 🔥 — Data Freshness

Build a query that returns:

```text
total_records
records_today
records_last_24_hours
records_older_than_7_days
```

This is a small pipeline data-quality dashboard.

---

# 🧠 32. Date/Time Cheat Sheet

## Current date

```sql
current_date()
```

## Current timestamp

```sql
current_timestamp()
```

## Extract date

```sql
date(timestamp_column)
```

## Year

```sql
year(date_column)
```

## Month

```sql
month(date_column)
```

## Day

```sql
day(date_column)
```

## Add time

```sql
date_add(date_column, interval 7 day)
```

## Subtract time

```sql
date_sub(date_column, interval 7 day)
```

## Difference in days

```sql
datediff(end_date, start_date)
```

## Difference in specific units

```sql
timestampdiff(hour, start_time, end_time)
```

---

# 🔥 33. Data Engineering Patterns to Memorize

### Last 24 hours

```sql
where created_at >= current_timestamp() - interval 1 day
```

### Older than 7 days

```sql
where created_at < current_timestamp() - interval 7 day
```

### Date difference

```sql
datediff(end_date, start_date)
```

### Previous event

```sql
lag(event_time) over (
    partition by entity_id
    order by event_time
)
```

### First event

```sql
min(event_time)
```

### Latest event

```sql
max(event_time)
```

### Watermark

```sql
where updated_at > last_processed_timestamp
```

---

# ⚠️ 34. Important Timestamp Lessons

### 1. Don't confuse date and timestamp

```text
2026-08-13
```

is not the same type of information as:

```text
2026-08-13 15:30:45
```

---

### 2. Know which timestamp you are using

Ask:

```text
Business event?
Creation?
Update?
Ingestion?
Processing?
```

---

### 3. Be careful with time zones

A timestamp such as:

```text
2026-08-13 10:00:00
```

without timezone context can be ambiguous in distributed systems.

Production pipelines often standardize timestamps, commonly using UTC internally, while presenting local time when required.

---

### 4. Think about boundaries

For a daily window, prefer clear boundaries such as:

```text
>= start
< next_start
```

instead of relying on a maximum timestamp like:

```text
23:59:59
```

This avoids precision issues.

---

# 🎯 35. Phase 5 Final Checklist

Before moving on:

### Date Fundamentals

- [ ] DATE vs timestamp
- [ ] `current_date()`
- [ ] `current_timestamp()`
- [ ] `date()`
- [ ] `year()`
- [ ] `month()`
- [ ] `day()`
- [ ] `extract()`

### Date Arithmetic

- [ ] `date_add()`
- [ ] `date_sub()`
- [ ] `datediff()`
- [ ] `timestampdiff()`

### Analytics

- [ ] First order
- [ ] Latest order
- [ ] Days between orders
- [ ] Monthly aggregation
- [ ] Month-over-month comparison

### Data Engineering

- [ ] Last 24 hours
- [ ] Older than 7 days
- [ ] Incremental loading
- [ ] Watermarks
- [ ] Late-arriving records
- [ ] Data freshness
- [ ] Missing dates
- [ ] Time-zone awareness

---

# 🏆 36. Interview Questions

### Q1. DATE vs TIMESTAMP?

> DATE represents a calendar date, while a timestamp includes date and time information.

### Q2. How do you find records from the last 24 hours?

A common pattern is:

```sql
where created_at >= current_timestamp() - interval 1 day
```

### Q3. How do you find records older than 7 days?

```sql
where created_at < current_timestamp() - interval 7 day
```

### Q4. What is a watermark?

> A watermark is a stored processing boundary, commonly a timestamp or increasing key, used to identify records that need to be processed in the next incremental run.

### Q5. Why is a watermark useful?

> It allows incremental processing to use the actual previous successful processing boundary rather than assuming a fixed time window.

### Q6. What is a late-arriving record?

> A record whose business event time is earlier than the time at which the data arrives or is ingested.

### Q7. Why can time zones matter?

> Distributed systems may run in different time zones, so inconsistent timestamp interpretation can cause incorrect filtering, duplicate processing, or missing records.

---

# ⭐ Final Phase 5 Takeaway

The goal is not:

> "I know DATE functions."

The goal is to recognize real pipeline requirements.

When you hear:

> **"Give me records from the last 24 hours."**

Think:

```text
current timestamp
        ↓
subtract 24 hours
        ↓
filter timestamp
```

When you hear:

> **"Find customers who haven't ordered in 90 days."**

Think:

```text
customer
    ↓
latest order
    ↓
compare with current date
    ↓
90-day threshold
```

When you hear:

> **"Process only new records."**

Think:

```text
watermark
    ↓
updated_at > last_processed_timestamp
    ↓
incremental records
```

When you hear:

> **"The data arrived late."**

Think:

```text
business event time
        ≠
ingestion time
```

That is the real Phase 5 skill:

> **Use time as a business and pipeline dimension, not just as a SQL data type.**
