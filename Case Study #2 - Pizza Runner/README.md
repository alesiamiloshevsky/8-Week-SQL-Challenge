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

- Clean the `exclusions` and `extras` columns to ensure consistent handling of missing values.  
- Replace empty strings `''` or the literal string `'null'` with SQL `NULL`.  
- Store the cleaned results in a new temporary table `customer_orders_clean`. 
<br>

```sql
CREATE TEMP TABLE customer_orders_clean AS
SELECT 
  order_id, 
  customer_id, 
  pizza_id, 
  CASE
	  WHEN exclusions = '' OR exclusions = 'null' THEN NULL
	  ELSE exclusions
	  END AS exclusions,
  CASE
	  WHEN extras = '' OR extras = 'null' THEN NULL
	  ELSE extras
	  END AS extras,
	order_time
FROM pizza_runner.customer_orders;

SELECT * FROM customer_orders_clean;
```

<br>
The query above produces the following cleaned version of the `customer_orders` table:

&nbsp;

| order_id | customer_id | pizza_id | exclusions | extras |    order_time     |
|----------|-------------|----------|------------|--------|-------------------|
|    1     |     101     |    1     |    NULL    |  NULL  | 2020-01-01 18:05:02 |
|    2     |     101     |    1     |    NULL    |  NULL  | 2020-01-01 19:00:52 |
|    3     |     102     |    1     |    NULL    |  NULL  | 2020-01-02 23:51:23 |
|    3     |     102     |    2     |    NULL    |  NULL  | 2020-01-02 23:51:23 |
|    4     |     103     |    1     |     4      |  NULL  | 2020-01-04 13:23:46 |
|    4     |     103     |    1     |     4      |  NULL  | 2020-01-04 13:23:46 |
|    4     |     103     |    2     |     4      |  NULL  | 2020-01-04 13:23:46 |
|    5     |     104     |    1     |    NULL    |   1    | 2020-01-08 21:00:29 |
|    6     |     101     |    2     |    NULL    |  NULL  | 2020-01-08 21:03:13 |
|    7     |     105     |    2     |    NULL    |   1    | 2020-01-08 21:20:29 |
|    8     |     102     |    1     |    NULL    |  NULL  | 2020-01-09 23:54:33 |
|    9     |     103     |    1     |     4      |  1, 5  | 2020-01-10 11:22:59 |
|   10     |     104     |    1     |    NULL    |  NULL  | 2020-01-11 18:34:49 |
|   10     |     104     |    1     |   2, 6     |  1, 4  | 2020-01-11 18:34:49 |

---

### Table: runner_orders

When reviewing the `runner_orders` table, we can see inconsistent values across multiple columns:

- Some columns contain the literal string `'null'` or blanks (`''`)
- Numeric fields like `distance` and `duration` also include text units (`'km'`, `'minutes'`, `'mins'`, etc.).  
- The `cancellation` column mixes blanks, `'null'`, and valid text values.
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

- Standardize the `pickup_time`, `distance`, `duration`, and `cancellation` columns to remove inconsistent values.  
- Convert the literal string `'null'` or empty strings `''` into SQL `NULL`.  
- Remove text units from numeric fields:  
  - Remove `'km'` from `distance`.  
  - Remove `'mins'`, `'minute'`, or `'minutes'` from `duration`.  
- Store the cleaned results in a new temporary table `runner_orders_clean`.  
<br>

```sql
CREATE TEMP TABLE runner_orders_clean AS
SELECT 
  order_id, 
  runner_id,  
  CASE
	  WHEN pickup_time LIKE 'null' THEN NULL
	  ELSE pickup_time
	  END AS pickup_time,
  CASE
	  WHEN distance LIKE 'null' THEN NULL
	  WHEN distance LIKE '%km' THEN TRIM('km' from distance)
	  ELSE distance 
    END AS distance,
  CASE
	  WHEN duration LIKE 'null' THEN NULL
	  WHEN duration LIKE '%mins' THEN TRIM('mins' from duration)
	  WHEN duration LIKE '%minute' THEN TRIM('minute' from duration)
	  WHEN duration LIKE '%minutes' THEN TRIM('minutes' from duration)
	  ELSE duration
	  END AS duration,
  CASE
	  WHEN cancellation = '' OR cancellation = 'null' THEN NULL
	  ELSE cancellation
	  END AS cancellation
FROM pizza_runner.runner_orders;

SELECT * FROM runner_orders_clean;
```
<br>

Alter the `pickup_time`, `distance`, and `duration` columns to the correct data type.

<br>

```sql
ALTER TABLE runner_orders_clean
	ALTER COLUMN pickup_time TYPE TIMESTAMP USING pickup_time::timestamp,
	ALTER COLUMN distance TYPE FLOAT USING distance::float,
    ALTER COLUMN duration TYPE INT USING duration::int;
```

<br>

The queries above produce the following cleaned version of the `runner_orders` table:

&nbsp;

| order_id | runner_id |     pickup_time     | distance | duration |      cancellation      |
|----------|-----------|---------------------|----------|----------|------------------------|
|    1     |     1     | 2020-01-01 18:15:34 |   20     |    32    | NULL                   |
|    2     |     1     | 2020-01-01 19:10:54 |   20     |    27    | NULL                   |
|    3     |     1     | 2020-01-03 00:12:37 |  13.4    |    20    | NULL                   |
|    4     |     2     | 2020-01-04 13:53:03 |  23.4    |    40    | NULL                   |
|    5     |     3     | 2020-01-08 21:10:57 |   10     |    15    | NULL                   |
|    6     |     3     | NULL                |  NULL    |   NULL   | Restaurant Cancellation|
|    7     |     2     | 2020-01-08 21:30:45 |   25     |    25    | NULL                   |
|    8     |     2     | 2020-01-10 00:15:02 |  23.4    |    15    | NULL                   |
|    9     |     2     | NULL                |  NULL    |   NULL   | Customer Cancellation  |
|   10     |     1     | 2020-01-11 18:50:20 |   10     |    10    | NULL                   |

---

## Case Study Questions

### A. Pizza Metrics
**1. How many pizzas were ordered?**

```sql
SELECT COUNT(order_id) AS pizza_order_count
FROM customer_orders_clean;
```

**Steps:**
- Query the cleaned orders table `clean_customer_orders`.
- Use `COUNT(order_id)` to count the number of order records.

**Answer:**
| pizza_order_count |
|-------------------|
| 14                |

---

**2. How many unique customer orders were made?**

```sql
SELECT COUNT (DISTINCT order_id) AS unique_customer_orders
FROM customer_orders_clean;
```

**Steps:**
- Query the cleaned orders table `customer_orders_clean`.  
- Use `COUNT(DISTINCT order_id)` to count the number of unique customer orders.

**Answer:**
| unique_customer_orders |
|------------------------|
| 10                     | 

---

**3. How many successful orders were delivered by each runner?**

```sql
SELECT
	runner_id,
	COUNT(order_id) AS successful_delivery
FROM runner_orders_clean
WHERE cancellation IS NULL
GROUP BY runner_id
ORDER BY runner_id;
```

**Steps:**
- Query the cleaned runner orders table `runner_orders_clean`.  
- Filter rows where `cancellation IS NULL` to count only successful deliveries.  
- Group the results by `runner_id`.  
- Use `COUNT(order_id)` to calculate the number of successful deliveries for each runner.  
- Order the results by `runner_id`.

**Answer:**
| runner_id | successful_delivery |
|-----------|----------------------|
| 1         | 4                    |
| 2         | 3                    |
| 3         | 1                    |

--- 

**4. How many of each type of pizza was delivered?**

```sql
SELECT 
	pizza_name,
    COUNT(customer.order_id) AS delivered_pizza_count
FROM customer_orders_clean customer   
INNER JOIN runner_orders_clean runner
	on customer.order_id = runner.order_id
INNER JOIN pizza_names pizza
	on customer.pizza_id = pizza.pizza_id
WHERE cancellation IS NULL
GROUP BY pizza_name
ORDER BY pizza_name;
```

**Steps:**
- Query the cleaned orders tables `customer_orders_clean` and `runner_orders_clean`, and join them on `order_id`.  
- Join with the `pizza_names` table to get the name of each pizza.  
- Filter rows where `cancellation IS NULL` to include only successful deliveries.  
- Group the results by `pizza_name`.  
- Use `COUNT(customer.order_id)` to count how many of each pizza type were delivered.  
- Order the results alphabetically by `pizza_name`.

**Answer:**
| pizza_name  | delivered_pizza_count |
|-------------|-----------------------|
| Meatlovers  | 9                     |
| Vegetarian  | 3                     |

--- 

**5. How many Vegetarian and Meatlovers were ordered by each customer?**

```sql
SELECT
	customer_id,
	pizza_name,
	COUNT(pizza_name) as order_count
FROM customer_orders customer
INNER JOIN pizza_names pizza
	on customer.pizza_id = pizza.pizza_id
GROUP BY customer_id, pizza_name
ORDER BY customer_id;
```

**Steps:**
- Query the cleaned orders table `customer_orders_clean`.  
- Join with the `pizza_names` table on `pizza_id` to get the pizza names.  
- Group the results by `customer_id` and `pizza_name`.  
- Use `COUNT(pizza_name)` to calculate how many of each pizza type was ordered per customer.  
- Order the results by `customer_id`.  

**Answer:**
| customer_id | pizza_name  | order_count |
|-------------|-------------|-------------|
| 101         | Meatlovers  | 2           |
| 101         | Vegetarian  | 1           |
| 102         | Meatlovers  | 2           |
| 102         | Vegetarian  | 1           |
| 103         | Meatlovers  | 3           |
| 103         | Vegetarian  | 1           |
| 104         | Meatlovers  | 3           |
| 105         | Vegetarian  | 1           |

---

**6. What was the maximum number of pizzas delivered in a single order?**

```sql
WITH maximum_pizza AS
(
SELECT
	customer.order_id,
	COUNT(pizza_id) AS pizza_count
FROM customer_orders_clean customer
INNER JOIN runner_orders_clean runner
	on customer.order_id = runner.order_id
WHERE cancellation IS NULL
GROUP BY customer.order_id
)

SELECT MAX(pizza_count) As max_pizza_count
FROM maximum_pizza;
```

**Steps:**
- Create a CTE `maximum_pizza` to calculate how many pizzas were included in each `order_id`.  
- Join `customer_orders_clean` with `runner_orders_clean` on `order_id`.  
- Filter out cancelled orders by checking `cancellation IS NULL`.  
- Use `COUNT(pizza_id)` to get the pizza count per order.  
- In the outer query, apply `MAX(pizza_count)` to find the largest number of pizzas in a single order.  

**Answer:**
| max_pizza_count |
|-----------------|
| 3               |

---

**7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?**

```sql
WITH pizza_count AS (
SELECT
	customer_id,
	CASE
    	WHEN exclusions IS NOT NULL OR extras IS NOT NULL THEN 1
    	ELSE 0
	END AS count_pizza_changes,
	CASE
   		WHEN exclusions IS NULL AND extras IS NULL THEN 1
    	ELSE 0
	END AS count_no_pizza_changes  
FROM customer_orders_clean customer
INNER JOIN runner_orders_clean runner
	on customer.order_id = runner.order_id
WHERE cancellation IS NULL
)

SELECT
	customer_id,
	SUM(count_pizza_changes) AS pizza_changes,
	SUM(count_no_pizza_changes) AS no_pizza_changes
FROM pizza_count 
GROUP BY customer_id
ORDER BY customer_id;
```

**Steps:**
- Create a CTE `pizza_count` to flag each delivered pizza as either having changes  
  (`exclusions` or `extras` present) or no changes (both columns `NULL`).  
- Join `customer_orders_clean` with `runner_orders_clean` on `order_id`.  
- Filter rows where `cancellation IS NULL` to include only successful deliveries.  
- In the outer query, group by `customer_id` and sum the flags to get totals for  
  `pizza_changes` and `no_pizza_changes`.
  
**Answer:**
| customer_id | pizza_changes | no_pizza_changes |
|-------------|---------------|------------------|
| 101         | 0             | 2                |
| 102         | 0             | 3                |
| 103         | 3             | 0                |
| 104         | 2             | 1                |
| 105         | 1             | 0                |

---

**8. How many pizzas were delivered that had both exclusions and extras?**

```sql
SELECT 
	COUNT(customer.order_id) AS pizza_with_exclusions_and_extras
FROM customer_orders_clean customer
INNER JOIN runner_orders_clean runner
	on customer.order_id = runner.order_id
WHERE cancellation IS NULL 
	AND exclusions is NOT NULL 
	AND extras IS NOT NULL;
```

**Steps:**
- Query the cleaned orders tables `customer_orders_clean` and `runner_orders_clean`.  
- Join them on `order_id` to match each customer order with its delivery record.  
- Filter to include only successful deliveries (`cancellation IS NULL`).  
- Further filter rows where both `exclusions` **and** `extras` are not `NULL`.  
- Count the number of such pizzas using `COUNT(customer.order_id)`.  

**Answer:**
| pizza_with_exclusions_and_extras |
|----------------------------------|
| 1                                |

---

**9. What was the total volume of pizzas ordered for each hour of the day?**
```sql
SELECT 
	EXTRACT(HOUR FROM order_time) AS hour_of_the_day,
    COUNT(order_id) AS pizza_orders
FROM customer_orders_clean
GROUP BY hour_of_the_day
ORDER BY hour_of_the_day;
```

**Steps:**
- Query the cleaned customer orders table `customer_orders_clean`.  
- Use `EXTRACT(HOUR FROM order_time)` to obtain the hour of the day for each order.  
- Group the results by the extracted hour.  
- Count the number of orders in each hour with `COUNT(order_id)`.  
- Order the results by `hour_of_the_day`.

**Answer:**
| hour_of_the_day | pizza_orders |
|-----------------|--------------|
| 11              | 1            |
| 13              | 3            |
| 18              | 3            |
| 19              | 1            |
| 21              | 3            |
| 23              | 3            |

---

**10. What was the volume of orders for each day of the week?**
```sql
```

**Steps:**

**Answer:**

---
