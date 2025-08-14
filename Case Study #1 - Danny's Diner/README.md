# Case Study #1: Danny’s Diner

## 📚 Table of Contents
All information regarding the case study can be found [here](https://8weeksqlchallenge.com/case-study-1/).

## 📝 Problem Statement
Danny’s Diner needs insights from customer data to understand visiting patterns, spending habits, and favorite menu items in order to improve the customer experience, evaluate the loyalty program, and create easy-to-use datasets for the team.

## 🗂 Entity Relationship Diagram
<img width="669" height="407" alt="image" src="https://github.com/user-attachments/assets/7d3f00f8-8e86-4fe0-ae6e-763da11c2e06" />

## ❓ Case Study Questions
**1. What is the total amount each customer spent at the restaurant?**
```sql
SELECT 
  sales.customer_id, 
  SUM(menu.price) AS total_sales
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id;
```

**Steps**
- Use `JOIN` to merge `dannys_diner.sales` and `dannys_diner.menu` tables so both `sales.customer_id` and `menu.price` are available.  
- Use `SUM` to calculate the total sales for each customer.  
- Group the results by `sales.customer_id`.  
- Order the output by `sales.customer_id` in ascending order.

**Answer:**
| customer_id | total_sales |
|-------------|-------------|
| A           | 76          |
| B           | 74          |
| C           | 36          |

---

**2. How many days has each customer visited the restaurant?**
```sql
SELECT 
	customer_id, 
	COUNT(DISTINCT order_date) AS count_visits
FROM dannys_diner.sales
GROUP BY customer_id
ORDER BY customer_id;
```

**Steps**
- Select `customer_id` from the `sales` table.  
- Use `COUNT(DISTINCT order_date)` to count unique visit days for each customer.  
- Group the results by `customer_id` so each customer’s total visits are calculated.  

**Answer:**
| customer_id | count_visits|
|-------------|-------------|
| A           | 4           |
| B           | 6           |
| C           | 2           |

---

**3. How many days has each customer visited the restaurant?**
```sql
WITH first_visit AS 
(
SELECT 
	customer_id, 
    order_date,
  	product_name,
    DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY order_date) AS rank_number
FROM dannys_diner.sales s
INNER JOIN dannys_diner.menu m
	on s.product_id = m.product_id
)

SELECT 
	DISTINCT customer_id, 
    product_name
FROM first_visit
WHERE rank_number = 1
ORDER BY customer_id, product_name;
```

**Steps**
- Create a CTE `first_visit` joining `dannys_diner.sales` with `dannys_diner.menu` to bring in `product_name`.
- Use `DENSE_RANK()` with `PARTITION BY customer_id ORDER BY order_date` to identify each customer’s first purchase day.
- Filter to `rank_number = 1` to keep only rows from the first day per customer.
- Select `DISTINCT customer_id, product_name` so duplicate items from the same day aren’t repeated.

**Answer:**
| customer_id | product_name |
|-------------|--------------|
| A           | curry        |
| A           | sushi        |
| B           | curry        |
| C           | ramen        |
















