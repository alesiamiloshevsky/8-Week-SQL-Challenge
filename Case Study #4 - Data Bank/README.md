# Case Study #4 - Data Bank

## 📚 Table of Contents
All information regarding the case study can be found [here](https://8weeksqlchallenge.com/case-study-4/).

## 📝 Problem Statement
Data Bank is a digital-only bank that connects customer account balances to cloud data storage limits. The company wants to expand its customer base while accurately forecasting future storage needs. This case study focuses on analyzing customer distribution, transaction behavior, and growth metrics to help Data Bank make data-driven decisions for expansion and optimization.

## 🗂 Entity Relationship Diagram
<img width="720" height="312" alt="image" src="https://github.com/user-attachments/assets/b28506e8-5024-4331-867d-816098aa03da" />

<br><br>

### Table 1: `Regions`

| region_id | region_name |
|------------|--------------|
| 1 | Africa |
| 2 | America |
| 3 | Asia |
| 4 | Europe |
| 5 | Oceania |

<br>

### Table 2: `Customer Nodes`

| customer_id | region_id | node_id | start_date | end_date   |
|--------------|------------|----------|-------------|------------|
| 1 | 3 | 4 | 2020-01-02 | 2020-01-03 |
| 2 | 3 | 5 | 2020-01-03 | 2020-01-17 |
| 3 | 5 | 4 | 2020-01-27 | 2020-02-18 |
| 4 | 5 | 4 | 2020-01-07 | 2020-01-19 |
| 5 | 3 | 3 | 2020-01-15 | 2020-01-23 |
| 6 | 1 | 1 | 2020-01-11 | 2020-02-06 |
| 7 | 2 | 5 | 2020-01-20 | 2020-02-04 |
| 8 | 1 | 2 | 2020-01-15 | 2020-01-28 |
| 9 | 4 | 5 | 2020-01-21 | 2020-01-25 |
| 10 | 3 | 4 | 2020-01-13 | 2020-01-14 |

<br>

### Table 3: `Customer Transactions`

| customer_id | txn_date   | txn_type | txn_amount |
|--------------|------------|-----------|-------------|
| 429 | 2020-01-21 | deposit | 82 |
| 155 | 2020-01-10 | deposit | 712 |
| 398 | 2020-01-01 | deposit | 196 |
| 255 | 2020-01-14 | deposit | 563 |
| 185 | 2020-01-29 | deposit | 626 |
| 309 | 2020-01-13 | deposit | 995 |
| 312 | 2020-01-20 | deposit | 485 |
| 376 | 2020-01-03 | deposit | 706 |
| 188 | 2020-01-13 | deposit | 601 |
| 138 | 2020-01-11 | deposit | 520 |

## Case Study Questions

### A. Customer Nodes Exploration

**1. How many unique nodes are there on the Data Bank system?**

```sql
```

**Steps:**

**Answer:**

---
