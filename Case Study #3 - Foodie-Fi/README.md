# Case Study #3 - Foodie-Fi

## 📚 Table of Contents
All information regarding the case study can be found [here](https://8weeksqlchallenge.com/case-study-3/).

## 📝 Problem Statement
Foodie-Fi is a streaming service with monthly and annual plans that gives customers unlimited access to exclusive cooking videos from around the world. Danny and his team want to use this data to see how customers move from trial to paid, which plans they choose, when they switch plans, and when they cancel.

## 🗂 Entity Relationship Diagram
<img width="685" height="282" alt="image" src="https://github.com/user-attachments/assets/780b8225-832c-4bee-8618-1695c07ffedd" />

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
```

**Steps:**

**Answer:**

---
