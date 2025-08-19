# Case Study #1: Danny’s Diner

## 📚 Table of Contents
All information regarding the case study can be found [here](https://8weeksqlchallenge.com/case-study-1/).

## 📝 Problem Statement
Danny’s Diner needs insights from customer data to understand visiting patterns, spending habits, and favorite menu items in order to improve the customer experience, evaluate the loyalty program, and create easy-to-use datasets for the team.

## 🗂 Entity Relationship Diagram
<img width="669" height="407" alt="image" src="https://github.com/user-attachments/assets/7d3f00f8-8e86-4fe0-ae6e-763da11c2e06" />

## Case Study Questions
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

**Steps:**
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

**Steps:**
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

**Steps:**
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

---

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**
```sql
SELECT
    product_name,
    COUNT(s.product_id) as most_purchased_item
FROM dannys_diner.sales s 
INNER JOIN dannys_diner.menu m
	on s.product_id = m.product_id
GROUP BY product_name
ORDER BY most_purchased_item DESC
LIMIT 1;
```

**Steps:**
- Join `dannys_diner.sales` with `dannys_diner.menu` on `product_id` to get item names.
- Aggregate with `COUNT(product_id)` grouped by `product_name` to total purchases per item across all customers.
- Order by the count in descending order to bring the most purchased item to the top.
- Limit to the top row to return only the single most purchased item.

**Answer:**
| product_name | most_purchased_item |
|--------------|-----------------|
| ramen        | 8               |

**5. Which item was the most popular for each customer?**
```sql
WITH most_popular_item AS (
SELECT 
	customer_id,
    product_name,
  	COUNT(s.product_id) as item_count,
    DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY COUNT(s.product_id)) AS rank_number
FROM dannys_diner.sales s
INNER JOIN dannys_diner.menu m
	on s.product_id = m.product_id
GROUP BY customer_id, product_name
)

SELECT 
  customer_id, 
  product_name, 
  item_count
FROM most_popular_item
WHERE rank_number = 1;
```

**Steps:**
- Create a CTE `most_popular_item` to calculate the total number of times each `product_name` was purchased by each `customer_id`.  
- Join `dannys_diner.sales` with `dannys_diner.menu` on `product_id` to get the product names.  
- Aggregate with `COUNT(product_id)` grouped by `customer_id` and `product_name` to get purchase counts per customer.  
- Use `DENSE_RANK()` partitioned by `customer_id` and ordered by the purchase count in descending order to rank items from most to least popular for each customer.  
- In the outer query, filter for rows where `rank_number = 1` to return only the most popular item(s) for each customer.  

**Answer:**
| customer_id | product_name | item_count |
|-------------|--------------|------------|
| A           | sushi        | 1          |
| B           | ramen        | 2          |
| B           | curry        | 2          |
| B           | sushi        | 2          |
| C           | ramen        | 3          |

---

**6. Which item was purchased first by the customer after they became a member?**
```sql
WITH first_member_purchase AS (
SELECT
	s.customer_id,
    product_id,
	RANK() OVER(PARTITION BY s.customer_id ORDER BY order_date) AS rank_number
FROM dannys_diner.sales s
INNER JOIN dannys_diner.members mem
	on s.customer_id = mem.customer_id
WHERE order_date > join_date
)

SELECT 
	customer_id,
    product_name
FROM first_member_purchase f
INNER JOIN dannys_diner.menu menu
	on f.product_id = menu.product_id
WHERE rank_number = 1
ORDER BY customer_id;
```

**Steps:**
- Create a CTE `first_member_purchase` to identify each customer's purchases **after** they became a member.
- Use `RANK()` partitioned by `customer_id` and ordered by `order_date` to find their first purchase after joining.
- Join `dannys_diner.sales` with `dannys_diner.members` on `customer_id` to filter by join date.
- Join with `dannys_diner.menu` to get the `product_name` for each purchase.
- Filter for `rank_number = 1` to keep only the first purchase per customer.
- Order by `customer_id` for clarity.
  
**Answer:**
| customer_id | product_name |
|-------------|--------------|
| A           | ramen        |
| B           | sushi        |

---

**7. Which item was purchased just before the customer became a member?
