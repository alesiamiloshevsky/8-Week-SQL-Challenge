# Case Study #1: Danny’s Diner

## 📚 Table of Contents
- [Business Task](#Problem-Statement)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Case Study Questions](#Case-Study-Questions)

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

---

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
- Create a CTE `first_member_purchase` to identify each customer's purchases after they became a member.
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

**7. Which item was purchased just before the customer became a member?**
```sql
WITH last_nonmember_purchase AS (
SELECT
	s.customer_id,
  	product_id,
	DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY order_date DESC) AS rank_number
FROM dannys_diner.sales s
INNER JOIN dannys_diner.members mem
	on s.customer_id = mem.customer_id
WHERE 	order_date < join_date
)

SELECT 
	customer_id,
	product_name
FROM last_nonmember_purchase l
INNER JOIN dannys_diner.menu menu
	on l.product_id = menu.product_id
WHERE rank_number = 1
ORDER BY customer_id;
```

**Steps:**
- Create a CTE `last_nonmember_purchase` to capture each customer’s purchases before they became a member.
- Use `DENSE_RANK()` partitioned by `customer_id` and ordered by `order_date DESC` to identify the last pre-membership purchase date (keeps all items if multiple were bought that day).
- Join `dannys_diner.sales` with `dannys_diner.members` on `customer_id` and filter with `order_date < join_date`.
- Join the CTE with `dannys_diner.menu` to retrieve the `product_name` for each purchase.
- Filter for `rank_number = 1` to keep only purchases from the last pre-membership day.
- Order by `customer_id` for clarity.

**Answer**
| customer_id | product_name |
|-------------|--------------|
| A           | curry        |
| A           | sushi        |
| B           | sushi        |

---

**8. What is the total items and amount spent for each member before they became a member?**
```sql
SELECT
	s.customer_id,
	COUNT(s.product_id) AS total_items,
    SUM(price) AS total_spent
FROM dannys_diner.sales s 
INNER JOIN dannys_diner.menu menu
	on s.product_id = menu.product_id
INNER JOIN dannys_diner.members mem
	on s.customer_id = mem.customer_id
WHERE order_date < join_date
GROUP BY s.customer_id
ORDER BY s.customer_id;
```

**Steps:**
- Join `dannys_diner.sales` with `dannys_diner.menu` on `product_id` to access prices.
- Join with `dannys_diner.members` on `customer_id` to filter only customers who became members.
- Apply the condition `order_date < join_date` to include only purchases made before membership.
- Use `COUNT(product_id)` to calculate the total number of items purchased.
- Use `SUM(price)` to calculate the total amount spent.
- Group the results by `customer_id` to get totals per customer.
- Order by `customer_id` for clarity.


**Answer**
| customer_id | total_items | total_spent |
|-------------|-------------|-------------|
| A           | 2           | 25          |
| B           | 3           | 40          |

---

**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**
```sql
WITH customer_points AS (
SELECT 
	customer_id,
    CASE
        WHEN m.product_id = 1 THEN price * 20
        ELSE price * 10
    END AS points
FROM dannys_diner.sales s 
INNER JOIN dannys_diner.menu m
	on s.product_id = m.product_id
)

SELECT 
	customer_id,
    SUM(points) AS total_points
FROM customer_points
GROUP BY customer_id
ORDER BY customer_id;
```

**Steps**
- Build a CTE `customer_points` that maps each `product_id` to base points:
  - `product_id = 1` (sushi) → `price * 20`
  - all other items → `price * 10`
- Join `customer_points` with `dannys_diner.sales` on `product_id` to attach points to every purchase.
- Sum `points` per `customer_id` to get each customer’s total.
- Order by `customer_id` for clarity.

**Answer**
| customer_id | total_points |
|-------------|--------------|
| A           | 860          |
| B           | 940          |
| C           | 360          |

---

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**
```sql
WITH customer_points AS (
SELECT 
	sales.customer_id,
    CASE
        WHEN menu.product_id = 1 AND order_date < join_date THEN price * 20
  		WHEN order_date BETWEEN join_date AND join_date + INTERVAL '6 days' THEN price * 20
  		WHEN menu.product_id = 1 AND order_date > join_date + INTERVAL '6 days' THEN price * 20
        ELSE price * 10
    END AS points
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu
	on sales.product_id = menu.product_id
INNER JOIN dannys_diner.members
	on sales.customer_id = members.customer_id
WHERE order_date < '2021-02-01'
)

SELECT 
	customer_id,
    SUM(points) AS total_points
FROM customer_points
GROUP BY customer_id
ORDER BY customer_id;
```

**Assumptions**
- The standard earning rate is 10 points for every $1 spent, while sushi always earns double at 20 points per $1.  
- Before a customer’s join date, only sushi purchases qualify for the double rate; all other items earn the standard rate.  
- From the join date through the following six days (a full seven-day period), all purchases earn double points regardless of the item.  
- After this initial week, sushi continues to earn double points, while all other items return to the base rate of 10 points per $1.  
- Only transactions made before February 1, 2021 are included in the calculation.  
- Each sales record represents a separate transaction and is counted individually when assigning points.  

**Steps**
- Create a CTE `customer_points` to calculate points earned for each purchase up to Jan 31.
- Join `dannys_diner.sales` with `dannys_diner.menu` (to get `price`) and `dannys_diner.members` (to get `join_date`).
- Use a `CASE` expression to assign points:
  - `product_id = 1` (sushi) before join date → `price * 20`
  - All items during join week (join_date → join_date + 6 days) → `price * 20`
  - Sushi after join week → `price * 20`
  - All other items → `price * 10`
- In the outer query, sum `points` per `customer_id` as `total_points`.
- Order by `customer_id` for readability.

**Answer**
| customer_id | total_points |
|-------------|--------------|
| A           | 1370         |
| B           | 820          |
