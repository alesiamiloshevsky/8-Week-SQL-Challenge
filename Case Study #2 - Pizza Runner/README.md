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
- Store the cleaned results in a new temporary table `customer_orders_temp`.  
- Query the new table instead of the original `customer_orders` when performing analysis.  

```sql
CREATE TEMP TABLE customer_orders_temp AS
SELECT 
	order_id,
    customer_id,
    pizza_id,
    CASE
    	WHEN exclusions is null OR exclusions LIKE 'null'  THEN ' '
        ELSE exclusions
        END AS exclusions,
    CASE
    	WHEN extras is null OR extras LIKE 'null' THEN ' '
        ELSE extras
        END AS extras,    
    order_time
FROM customer_orders;

SELECT *
FROM customer_orders;

SELECT *
FROM customer_orders_temp;
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

## Case Study Questions
