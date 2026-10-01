# Pizza Runner
<img width="1080" height="1080" alt="image" src="https://github.com/user-attachments/assets/c571cbd9-b8d0-423a-91cb-f03bb4c2a13d" />

## Table of Contents
- [Overview](#overview)
- [Business Context](#business-context)
- [Data Schema](#data-schema)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Data Cleaning & Preparation](#data-cleaning--preparation)
- [Business Questions & Solutions](#business-questions--solutions)
- [Key Insights & Recommendations](#key-insights--recommendations)

## Overview
This project is part of the 8-Week SQL Challenge by Danny Ma. Case Study #2 focuses on Pizza Runner, a mock retro-themed pizza delivery startup. The objective is to clean raw operational data and write end-to-end SQL queries to optimize delivery logistics, analyze runner performance, and evaluate customer ordering behavior.

## Business Context
To scale operations beyond a single location, Pizza Runner collects transactional data on orders, deliveries, and runner logistics. However, the raw database contains formatting inconsistencies, missing values, and unstructured text fields that prevent immediate analysis.

This project addresses two core priorities:
1. Data Cleaning & Transformation: Standardizing data types, stripping text units from numerical metrics, and handling NULL values.
2. Operational Analytics: Querying cleaned datasets to deliver actionable insights on delivery efficiency, customer preferences, and product customization.

## Data Schema
All tables sit within the pizza_runner schema across six relational tables:
- customer_orders: Order-level records, including customer IDs, pizza selections, exclusions, extras, and order timestamps.
- runner_orders: Delivery metrics, including assigned runners, pickup times, travel distances, duration, and cancellation records.
- runners: Registration and onboarding dates for delivery runners.
- pizza_names: Maps pizza IDs to their respective menu names.
- pizza_recipes: Defines the default topping combinations for each pizza type.
- pizza_toppings: Serves as the lookup table for topping names and IDs.

## Entity Relationship Diagram
<img width="520" height="230" alt="image" src="https://github.com/user-attachments/assets/d0096130-fd05-4bbf-a98a-d64493fe1010" />

## Data Cleaning & Preparation

#### Customer Orders Table Clean Up
The raw exclusions and extras columns in customer_orders table had inconsistent missing entries (true NULLs, blank spaces, and the text 'null'). I created a new temporary table called customer_orders_temp using CASE statements to detect all these variations and standardize them into uniform database NULL values.


```sql
DROP TABLE IF EXISTS customer_orders_temp;

CREATE TEMP TABLE customer_orders_temp AS
	SELECT order_id,
    	   customer_id,
           pizza_id,
           CASE
           	WHEN exclusions IS NULL THEN NULL
  			WHEN TRIM(exclusions) IN ('', 'null') THEN NULL
            ELSE exclusions
           END AS exclusions,
           CASE 
           	WHEN extras is NULL THEN NULL
            WHEN TRIM(extras) IN ('', 'null') THEN NULL
            ELSE extras
           END AS extras,
           order_time
	FROM customer_orders;
    
    SELECT * FROM customer_orders_temp
```


✅ Result
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


#### Runner Orders Table Clean Up
The raw runner_orders table had inconsistent missing entries (true NULLs, blanks, and text 'null') and messy text suffixes (km, mins, minutes). I created a temporary table, runner_orders_temp, using CASE statements combined with REGEXP_REPLACE to strip the text units and standardize all missing data into uniform database NULL values. I then used explicit CAST functions to immediately convert the columns into their proper TIMESTAMP, FLOAT, and INTEGER data types.

```sql
DROP TABLE IF EXISTS runner_orders_temp;

CREATE TEMP TABLE runner_orders_temp AS
	SELECT order_id, runner_id,
      CASE 
         WHEN pickup_time is NULL or pickup_time in ('null', ' ') THEN NULL
         ELSE CAST(pickup_time AS timestamp)
      END AS pickup_time, 
      CASE 
         WHEN distance IS NULL OR distance IN ('null', '') THEN NULL
         ELSE CAST(REGEXP_REPLACE(distance, '[a-zA-Z ]', '', 'g') AS FLOAT)
      END AS distance,
      CASE
         WHEN duration IS NULL OR duration IN ('null', '') THEN NULL
         ELSE CAST(REGEXP_REPLACE(duration, '[a-zA-Z]', '', 'g') AS INT)
      END AS duration,
      CASE
      	WHEN cancellation IS NULL OR cancellation IN ('null', '') THEN NULL
        ELSE cancellation
      END AS cancellation
    FROM runner_orders;
    
SELECT *
FROM runner_orders_temp;
```

✅ Result

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


## Business Questions & Solutions

### Question 1 - How many pizzas were ordered?

#### Approach
Since we are looking for the total number of pizza orders, I counted the records in the customer_orders_temp table.

```sql
SELECT COUNT(*) AS total_pizza_order
FROM customer_orders_temp;
```
✅ Result

| total_pizza_order |
| ----------------- |
| 14                |

#### Key Takeaway
Dataset contains a total of 14 pizza orders.


## Key Insights & Recommendations
---


