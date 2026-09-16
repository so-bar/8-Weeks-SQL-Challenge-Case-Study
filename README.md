# dannys-diner-customer-analytics
<img width="1080" height="1080" alt="image" src="https://github.com/user-attachments/assets/6c61bb4b-0bea-45cb-b62d-6cd8623a7a41" />


## Entity Relationship Diagram
<img width="630" height="287" alt="image" src="https://github.com/user-attachments/assets/871d488d-5439-4a4f-b4f3-daa008f4df6d" />
  
**1. What is the total amount each customer spent at the restaurant?**
    
    SELECT
      	customer_id, sum(price) AS total_spent
    FROM dannys_diner.sales s
    JOIN dannys_diner.menu m
    ON s.product_id = m.product_id
    GROUP BY customer_id
    ORDER BY customer_transactions DESC

**Approach:** I started with the sales table because it contains the customer purchases. Since the price isn't stored in sales, I joined it with the menu table using product_id. Then I summed the price for each customer and grouped the results by customer_id. Finally, I ordered the results by total spending in descending order.

| customer_id | total_spent |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

---

**2. How many days has each customer visited the restaurant?**

    SELECT customer_id, COUNT(DISTINCT order_date) as total_visit
    FROM sales
    GROUP BY customer_id

| customer_id | total_visit |
| ----------- | ----------- |
| A           | 4           |
| B           | 6           |
| C           | 2           |

**3. What was the first item from the menu purchased by each customer?**

    WITH customer_order AS (
      SELECT customer_id, order_date, product_name,
      RANK() OVER(PARTITION BY customer_id ORDER BY order_date asc) as order_rank
      FROM sales s
      JOIN menu m
      ON s.product_id = m.product_id
     )
     
     SELECT DISTINCT customer_id, order_date, product_name
     FROM customer_order
     WHERE order_rank = 1


| customer_id | order_date | product_name |
| ----------- | ---------- | ------------ |
| A           | 2021-01-01 | curry        |
| A           | 2021-01-01 | sushi        |
| B           | 2021-01-01 | curry        |
| C           | 2021-01-01 | ramen        |

