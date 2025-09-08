# Case Study #2 - Pizza Runner

## 📚 Table of Contents
All information regarding the case study can be found [here](https://8weeksqlchallenge.com/case-study-2/).

## 📝 Problem Statement
Danny has launched Pizza Runner, a startup that combines pizza delivery with an Uber style runner network. Customers place orders through a mobile app, and freelance runners deliver pizzas from Pizza Runner Headquarters.

The app records order, delivery, and recipe data, but the information is messy with null values, inconsistent formats, and unstructured ingredient lists. To help Danny grow his Pizza Empire, the data needs to be cleaned, transformed, and analyzed with SQL to uncover insights about customer demand, runner performance, and business profitability.

## 🗂 Entity Relationship Diagram
<img width="788" height="477" alt="image" src="https://github.com/user-attachments/assets/155e0171-5bd6-4b67-bdf7-fd868248afec"/>

## 🧹 Data Cleaning

### Table: customer_orders

When reviewing the `customer_orders` table, we can see that some columns contain inconsistent values:  
- The `exclusions` and `extras` columns are inconsistent, containing a mix of `NULL` and blank (`''`) values.
<br>

| order_id | customer_id | pizza_id | exclusions | extras | order_time          |
| -------- | ----------- | -------- | ---------- | ------ | ------------------- |
| 1        | 101         | 1        |            |        | 2020-01-01 18:05:02 |
| 2        | 101         | 1        |            |        | 2020-01-01 19:00:52 |
| 3        | 102         | 1        |            |        | 2020-01-02 23:51:23 |
| 3        | 102         | 2        |            |        | 2020-01-02 23:51:23 |
| 4        | 103         | 1        | 4          |        | 2020-01-04 13:23:46 |
| 4        | 103         | 1        | 4          |        | 2020-01-04 13:23:46 |
| 4        | 103         | 2        | 4          |        | 2020-01-04 13:23:46 |
| 5        | 104         | 1        | null       | 1      | 2020-01-08 21:00:29 |
| 6        | 101         | 2        | null       | null   | 2020-01-08 21:03:13 |
| 7        | 105         | 2        | null       | 1      | 2020-01-08 21:20:29 |
| 8        | 102         | 1        | null       | null   | 2020-01-09 23:54:33 |
| 9        | 103         | 1        | 4          | 1, 5   | 2020-01-10 11:22:59 |
| 10       | 104         | 1        | null       | null   | 2020-01-11 18:34:49 |
| 10       | 104         | 1        | 2, 6       | 1, 4   | 2020-01-11 18:34:49 |
<br>

**🛠️ Cleanup Strategy**

- Standardize the `exclusions` and `extras` columns so they no longer contain inconsistent values.  
- Replace `NULL` or the literal string `'null'` with a consistent placeholder.  
- Store the cleaned results in a new temporary table `clean_customer_orders`.
<br>

```sql
CREATE TEMP TABLE clean_customer_orders AS
SELECT 
  order_id, 
  customer_id, 
  pizza_id, 
  CASE
	  WHEN exclusions IS null OR exclusions LIKE 'null' THEN ' '
	  ELSE exclusions
	  END AS exclusions,
  CASE
	  WHEN extras IS NULL or extras LIKE 'null' THEN ' '
	  ELSE extras
	  END AS extras,
	order_time
FROM pizza_runner.customer_orders;
```

<br>
The query above produces the following cleaned version of the `customer_orders` table:

&nbsp;

| order_id | customer_id | pizza_id | exclusions | extras | order_time          |
| -------- | ----------- | -------- | ---------- | ------ | ------------------- |
| 1        | 101         | 1        |            |        | 2020-01-01 18:05:02 |
| 2        | 101         | 1        |            |        | 2020-01-01 19:00:52 |
| 3        | 102         | 1        |            |        | 2020-01-02 23:51:23 |
| 3        | 102         | 2        |            |        | 2020-01-02 23:51:23 |
| 4        | 103         | 1        | 4          |        | 2020-01-04 13:23:46 |
| 4        | 103         | 1        | 4          |        | 2020-01-04 13:23:46 |
| 4        | 103         | 2        | 4          |        | 2020-01-04 13:23:46 |
| 5        | 104         | 1        |            | 1      | 2020-01-08 21:00:29 |
| 6        | 101         | 2        |            |        | 2020-01-08 21:03:13 |
| 7        | 105         | 2        |            | 1      | 2020-01-08 21:20:29 |
| 8        | 102         | 1        |            |        | 2020-01-09 23:54:33 |
| 9        | 103         | 1        | 4          | 1, 5   | 2020-01-10 11:22:59 |
| 10       | 104         | 1        |            |        | 2020-01-11 18:34:49 |
| 10       | 104         | 1        | 2, 6       | 1, 4   | 2020-01-11 18:34:49 |

---

### Table: runner_orders

- The `pickup_time` column contains valid timestamps, but also includes `'null'` strings alongside actual missing values.  
- The `distance` column is inconsistent, with values stored as numbers with `'km'` (e.g., `20km`), numbers with extra spacing (e.g., `23.4 km`), plain numbers without units (e.g., `23.4`, `10`), and `'null'` strings.  
- The `duration` column is recorded in multiple formats such as `'minutes'`, `'minute'`, `'mins'`, `'min'`, and even entries with extra characters (e.g., `25mins`, `20 s`), as well as `'null'`.  
- The `cancellation` column contains meaningful values like `Restaurant Cancellation` and `Customer Cancellation`, but missing data is represented inconsistently with empty strings (`''`), `'null'` text, and `NULL`.  
<br>

| order_id | runner_id | pickup_time         | distance | duration   | cancellation            |
| -------- | --------- | ------------------- | -------- | ---------- | ----------------------- |
| 1        | 1         | 2020-01-01 18:15:34 | 20km     | 32 minutes |                         |
| 2        | 1         | 2020-01-01 19:10:54 | 20km     | 27 minutes |                         |
| 3        | 1         | 2020-01-03 00:12:37 | 13.4km   | 20 mins    |                         |
| 4        | 2         | 2020-01-04 13:53:03 | 23.4     | 40         |                         |
| 5        | 3         | 2020-01-08 21:10:57 | 10       | 15         |                         |
| 6        | 3         | null                | null     | null       | Restaurant Cancellation |
| 7        | 2         | 2020-01-08 21:30:45 | 25km     | 25mins     | null                    |
| 8        | 2         | 2020-01-10 00:15:02 | 23.4 km  | 15 minute  | null                    |
| 9        | 2         | null                | null     | null       | Customer Cancellation   |
| 10       | 1         | 2020-01-11 18:50:20 | 10km     | 10minutes  | null                    |
<br>

**🛠️ Cleanup Strategy**

- Standardize the `pickup_time` column by replacing `'null'` strings with a consistent placeholder for missing values.  
- Clean the `distance` column by removing the `'km'` suffix and converting values into a consistent numeric format, while treating `'null'` as missing.  
- Normalize the `duration` column by removing `'minutes'`, `'minute'`, `'mins'`, and `'min'` so only numeric values remain, with `'null'` treated as missing.  
- Ensure the `cancellation` column represents missing values consistently instead of mixing `''`, `'null'`, and `NULL`, while keeping meaningful cancellation reasons intact.  
<br>

```sql
CREATE TEMP TABLE clean_runner_orders AS
SELECT 
  order_id, 
  runner_id,  
  CASE
	  WHEN pickup_time LIKE 'null' THEN ' '
	  ELSE pickup_time
	  END AS pickup_time,
  CASE
	  WHEN distance LIKE 'null' THEN ' '
	  WHEN distance LIKE '%km' THEN TRIM('km' from distance)
	  ELSE distance 
    END AS distance,
  CASE
	  WHEN duration LIKE 'null' THEN ' '
	  WHEN duration LIKE '%mins' THEN TRIM('mins' from duration)
	  WHEN duration LIKE '%minute' THEN TRIM('minute' from duration)
	  WHEN duration LIKE '%minutes' THEN TRIM('minutes' from duration)
	  ELSE duration
	  END AS duration,
  CASE
	  WHEN cancellation IS NULL or cancellation LIKE 'null' THEN ' '
	  ELSE cancellation
	  END AS cancellation
FROM pizza_runner.runner_orders;
```
<br>

Alter the `pickup_time`, `distance`, and `duration` columns to the correct data type.

<br>

```sql
ALTER TABLE runner_orders_temp
ALTER COLUMN pickup_time DATETIME,
ALTER COLUMN distance FLOAT,
ALTER COLUMN duration INT;
```

<br>

The queries above produce the following cleaned version of the `customer_orders` table:

&nbsp;

| order_id | runner_id | pickup_time         | distance | duration | cancellation            |
| -------- | --------- | ------------------- | -------- | -------- | ----------------------- |
| 1        | 1         | 2020-01-01 18:15:34 | 20       | 32       |                         |
| 2        | 1         | 2020-01-01 19:10:54 | 20       | 27       |                         |
| 3        | 1         | 2020-01-03 00:12:37 | 13.4     | 20       |                         |
| 4        | 2         | 2020-01-04 13:53:03 | 23.4     | 40       |                         |
| 5        | 3         | 2020-01-08 21:10:57 | 10       | 15       |                         |
| 6        | 3         |                     |          |          | Restaurant Cancellation |
| 7        | 2         | 2020-01-08 21:30:45 | 25       | 25       |                         |
| 8        | 2         | 2020-01-10 00:15:02 | 23.4     | 15       |                         |
| 9        | 2         |                     |          |          | Customer Cancellation   |
| 10       | 1         | 2020-01-11 18:50:20 | 10       | 10       |                         |

---

## Case Study Questions

### A. Pizza Metrics
**1. How many pizzas were ordered?**

```sql
SELECT COUNT(order_id)
FROM clean_customer_orders;
```

**Steps**
- Query the cleaned orders table `clean_customer_orders`.
- Use `COUNT(order_id)` to count the number of order records.
- Return the single aggregated result.

**Answer**
| count |
|-------|
| 14    |

---
**2. How many unique customer orders were made?**
