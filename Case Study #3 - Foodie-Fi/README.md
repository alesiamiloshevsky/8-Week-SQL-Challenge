# Case Study #3 - Foodie-Fi

## 📚 Table of Contents
All information regarding the case study can be found [here](https://8weeksqlchallenge.com/case-study-3/).

## 📝 Problem Statement
Foodie-Fi is a streaming service with monthly and annual plans that gives customers unlimited access to exclusive cooking videos from around the world. Danny and his team want to use this data to see how customers move from trial to paid plans, which plans they choose, when they switch plans, and when they cancel.

## 🗂 Entity Relationship Diagram
<img width="685" height="282" alt="image" src="https://github.com/user-attachments/assets/780b8225-832c-4bee-8618-1695c07ffedd" />

<br><br>

### Table 1: `plans`

| plan_id | plan_name       | price |
|---------|------------------|-------|
| 0       | trial            | 0     |
| 1       | basic monthly    | 9.90  |
| 2       | pro monthly      | 19.90 |
| 3       | pro annual       | 199   |
| 4       | churn            | null  |

<br>

### Table 2: `subscriptions`

| customer_id | plan_id | start_date |
|-------------|---------|------------|
| 1           | 0       | 2020-08-01 |
| 1           | 1       | 2020-08-08 |
| 2           | 0       | 2020-09-20 |
| 2           | 3       | 2020-09-27 |
| 11          | 0       | 2020-11-19 |
| 11          | 4       | 2020-11-26 |
| 13          | 0       | 2020-12-15 |
| 13          | 1       | 2020-12-22 |
| 13          | 2       | 2021-03-29 |
| 15          | 0       | 2020-03-17 |
| 15          | 2       | 2020-03-24 |
| 15          | 4       | 2020-04-29 |
| 16          | 0       | 2020-05-31 |
| 16          | 1       | 2020-06-07 |
| 16          | 3       | 2020-10-21 |
| 18          | 0       | 2020-07-06 |
| 18          | 2       | 2020-07-13 |
| 19          | 0       | 2020-06-22 |
| 19          | 2       | 2020-06-29 |
| 19          | 3       | 2020-08-29 |

## Case Study Questions

### A. Customer Journey

<br>

Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.

<br>

```sql
SELECT 
	subscriptions.customer_id,
    plans.plan_id,
    plans.plan_name,
    plans.price, 
    subscriptions.start_date
FROM foodie_fi.plans plans
INNER JOIN foodie_fi.subscriptions subscriptions
	on plans.plan_id = subscriptions.plan_id
WHERE subscriptions.customer_id IN (1, 2, 11, 13, 15, 16, 18, 19)
ORDER BY customer_id, plan_id;
```
<br>

I joined the `subscriptions` and `plans` tables to include plan names and prices for each subscription, making the results clearer and easier to summarize.

<br>

| customer_id | plan_id | plan_name     | price  | start_date |
|-------------|---------|---------------|--------|------------|
| 1           | 0       | trial         | 0.00   | 2020-08-01 |
| 1           | 1       | basic monthly | 9.90   | 2020-08-08 |
| 2           | 0       | trial         | 0.00   | 2020-09-20 |
| 2           | 3       | pro annual    | 199.00 | 2020-09-27 |
| 11          | 0       | trial         | 0.00   | 2020-11-19 |
| 11          | 4       | churn         | null   | 2020-11-26 |
| 13          | 0       | trial         | 0.00   | 2020-12-15 |
| 13          | 1       | basic monthly | 9.90   | 2020-12-22 |
| 13          | 2       | pro monthly   | 19.90  | 2021-03-29 |
| 15          | 0       | trial         | 0.00   | 2020-03-17 |
| 15          | 2       | pro monthly   | 19.90  | 2020-03-24 |
| 15          | 4       | churn         | null   | 2020-04-29 |
| 16          | 0       | trial         | 0.00   | 2020-05-31 |
| 16          | 1       | basic monthly | 9.90   | 2020-06-07 |
| 16          | 3       | pro annual    | 199.00 | 2020-10-21 |
| 18          | 0       | trial         | 0.00   | 2020-07-06 |
| 18          | 2       | pro monthly   | 19.90  | 2020-07-13 |
| 19          | 0       | trial         | 0.00   | 2020-06-22 |
| 19          | 2       | pro monthly   | 19.90  | 2020-06-29 |
| 19          | 3       | pro annual    | 199.00 | 2020-08-29 |

<br>

Based on the table above, here are the eight customers’ onboarding journeys.

<br>

**Customer 1** - Started a free trial on August 1st, 2020 and moved to basic monthly on August 8th, 2020.

<br>

| customer_id | plan_id | plan_name     | price  | start_date |
|-------------|---------|---------------|--------|------------|
| 1           | 0       | trial         | 0.00   | 2020-08-01 |
| 1           | 1       | basic monthly | 9.90   | 2020-08-08 |

<br>

**Customer 2** - Began a free trial on September 20th, 2020 and upgraded to pro annual on September 27th, 2020.

<br>

| customer_id | plan_id | plan_name     | price  | start_date |
|-------------|---------|---------------|--------|------------|
| 2           | 0       | trial         | 0.00   | 2020-09-20 |
| 2           | 3       | pro annual    | 199.00 | 2020-09-27 |

<br>

**Customer 11** - Started a free trial on November 19th, 2020 and churned on November 26th, 2020 (access continues until the billing period ends).

<br>

| customer_id | plan_id | plan_name     | price  | start_date |
|-------------|---------|---------------|--------|------------|
| 11          | 0       | trial         | 0.00   | 2020-11-19 |
| 11          | 4       | churn         | null   | 2020-11-26 |

<br>

**Customer 13** - Began a free trial on December 15th, 2020, switched to basic monthly on December 22nd, 2020, and upgraded to pro monthly on March 29th, 2021.

<br>

| customer_id | plan_id | plan_name     | price  | start_date |
|-------------|---------|---------------|--------|------------|
| 13          | 0       | trial         | 0.00   | 2020-12-15 |
| 13          | 1       | basic monthly | 9.90   | 2020-12-22 |
| 13          | 2       | pro monthly   | 19.90  | 2021-03-29 |

<br>

**Customer 15** - Started a free trial on March 17th, 2020, upgraded to pro monthly on March 24th, 2020, and churned on April 29th, 2020.

<br>

| customer_id | plan_id | plan_name     | price  | start_date |
|-------------|---------|---------------|--------|------------|
| 15          | 0       | trial         | 0.00   | 2020-03-17 |
| 15          | 2       | pro monthly   | 19.90  | 2020-03-24 |
| 15          | 4       | churn         | null   | 2020-04-29 |

<br>

**Customer 16** - Began a free trial on May 31st, 2020, moved to basic monthly on June 7th, 2020, and upgraded to pro annual on October 21st, 2020.

<br>

| customer_id | plan_id | plan_name     | price  | start_date |
|-------------|---------|---------------|--------|------------|
| 16          | 0       | trial         | 0.00   | 2020-05-31 |
| 16          | 1       | basic monthly | 9.90   | 2020-06-07 |
| 16          | 3       | pro annual    | 199.00 | 2020-10-21 |

<br>

**Customer 18** - Started a free trial on July 6th, 2020 and upgraded to pro monthly on July 13th, 2020.

<br>

| customer_id | plan_id | plan_name     | price  | start_date |
|-------------|---------|---------------|--------|------------|
| 18          | 0       | trial         | 0.00   | 2020-07-06 |
| 18          | 2       | pro monthly   | 19.90  | 2020-07-13 |

<br>

**Customer 19** - Began a free trial on June 22nd, 2020, switched to pro monthly on June 29th, 2020, and upgraded to pro annual on August 29th, 2020.

<br>

| customer_id | plan_id | plan_name     | price  | start_date |
|-------------|---------|---------------|--------|------------|
| 19          | 0       | trial         | 0.00   | 2020-06-22 |
| 19          | 2       | pro monthly   | 19.90  | 2020-06-29 |
| 19          | 3       | pro annual    | 199.00 | 2020-08-29 |

---

### B. Data Analysis Questions

**1. How many customers has Foodie-Fi ever had?**

```sql
SELECT COUNT(DISTINCT customer_id) AS customers_count
FROM foodie_fi.subscriptions;
```

**Steps:**
- Use `COUNT(DISTINCT customer_id)` to count the number of unique customers.

**Answer:**
| customers_count |
|-----------------|
| 1000            |

---

**2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value**

```sql
SELECT 
  DATE_PART('month', start_date) AS month_date,
  COUNT(customer_id)
FROM foodie_fi.subscriptions
WHERE plan_id = 0
GROUP BY month_date
ORDER BY month_date;
```

**Steps:**
- Filter rows where `plan_id = 0` to count only trial plan subscriptions.  
- Use `DATE_PART('month', start_date)` to extract the month number from each `start_date`.  
- Group the results by the extracted month and count the number of `customer_id` records in each month.  
- Order the output by `month_date` to show months in chronological order.  

**Answer:**
| month_date | count |
|------------|------|
| 1          | 88   |
| 2          | 68   |
| 3          | 94   |
| 4          | 81   |
| 5          | 88   |
| 6          | 79   |
| 7          | 89   |
| 8          | 88   |
| 9          | 87   |
| 10         | 79   |
| 11         | 75   |
| 12         | 84   |

---

**3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name**

```sql
SELECT
	subscriptions.plan_id,
	plan_name,
	COUNT(subscriptions.customer_id) AS count_of_events
FROM foodie_fi.subscriptions subscriptions
INNER JOIN foodie_fi.plans plans
	on subscriptions.plan_id = plans.plan_id
WHERE DATE_PART('year', start_date) > 2020
GROUP BY subscriptions.plan_id, plan_name
ORDER BY subscriptions.plan_id;
```

**Steps:**
- Query the `foodie_fi.subscriptions` table and join it with the `foodie_fi.plans` table on `plan_id` to get the plan names.  
- Filter for subscriptions where the `start_date` year is greater than 2020.  
- Group the results by `subscriptions.plan_id` and `plan_name`.  
- Count the number of `customer_id` events per plan using `COUNT(subscriptions.customer_id)`.  
- Order the output by `subscriptions.plan_id`.

**Answer:**
| plan_id | plan_name      | count_of_events |
|-------- |----------------|-----------------|
| 1       | basic monthly  | 8               |
| 2       | pro monthly    | 60              |
| 3       | pro annual     | 63              |
| 4       | churn          | 71              |

---

**4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?**

```sql
SELECT 
	COUNT(DISTINCT customer_id) AS churn_count,
    ROUND(COUNT(customer_id) / (SELECT COUNT(DISTINCT customer_id)::numeric 	FROM foodie_fi.subscriptions) * 100, 1) AS churn_percentage
FROM foodie_fi.subscriptions
WHERE plan_id = 4;
```

**Steps:**
- Filter rows where `plan_id = 4` to count only churned customers.  
- Use `COUNT(DISTINCT customer_id)` to get the total number of churned customers (`churn_count`).  
- Divide this by the total number of distinct customers in the entire table and multiply by 100 to get a percentage.
- Cast the total‐customer count to `numeric` so the division is performed as a decimal rather than integer.  
- Use `ROUND(..., 1)` to round the percentage to one decimal place.
  
**Answer:**
| churn_count | churn_percentage |
|-------------|------------------|
| 307         | 30.7             |

---

**3. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?**

```sql
```

**Steps:**

**Answer:**

---
