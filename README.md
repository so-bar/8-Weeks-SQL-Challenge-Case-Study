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

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**

      SELECT product_name, COUNT(product_name) AS total_orders
      FROM sales s
      JOIN menu m
      ON s.product_id = m.product_id
      GROUP BY product_name
      ORDER BY total_orders DESC
      LIMIT 1;

| product_name | total_orders |
| ------------ | ------------ |
| ramen        | 8            |

**5. Which item was the most popular for each customer?**

    WITH order_list AS(
      SELECT customer_id, product_name,
      COUNT(product_name) AS total_orders
      FROM sales s
      JOIN menu m
      ON s.product_id = m.product_id
      GROUP BY customer_id, product_name
    ),
    ranked_list AS(
      SELECT *, DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY total_orders DESC) AS ranking
      FROM order_list
    )
      
     SELECT customer_id, product_name
     FROM ranked_list
     WHERE ranking = 1;

| customer_id | product_name |
| ----------- | ------------ |
| A           | ramen        |
| B           | curry        |
| B           | sushi        |
| B           | ramen        |
| C           | ramen        |

**6. Which item was purchased first by the customer after they became a member?**
    
    WITH ranked_member_sales AS(
      SELECT s.customer_id, product_name, order_date,
      DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY order_date ASC) as ranking
      FROM sales s
      JOIN menu mn
      ON s.product_id = mn.product_id
      JOIN members m
      ON s.customer_id = m.customer_id AND s.order_date >= m.join_date
    )
     
    SELECT customer_id, product_name, order_date
    FROM ranked_member_sales
    WHERE ranking = 1

| customer_id | product_name | order_date | ranking |
| ----------- | ------------ | ---------- | ------- |
| A           | curry        | 2021-01-07 | 1       |
| B           | sushi        | 2021-01-11 | 1       |


 **7. Which item was purchased just before the customer became a member?**
 
    WITH ranked_member_sales AS(
      SELECT s.customer_id, product_name, order_date,
      DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY order_date DESC) as ranking
      FROM sales s
      JOIN menu mn
      ON s.product_id = mn.product_id
      JOIN members m
      ON s.customer_id = m.customer_id AND s.order_date < m.join_date
    )
    
    SELECT customer_id, product_name, order_date
    FROM ranked_member_sales
    WHERE ranking = 1

| customer_id | product_name | order_date |
| ----------- | ------------ | ---------- |
| A           | sushi        | 2021-01-01 |
| A           | curry        | 2021-01-01 |
| B           | sushi        | 2021-01-04 |

**8. What is the total items and amount spent for each member before they became a member?**

     SELECT s.customer_id, count(s.product_id) AS total_items, sum(price) AS total_amount_spent
     FROM sales s
     JOIN menu mn
     ON s.product_id = mn.product_id
     JOIN members m
     ON s.customer_id = m.customer_id AND s.order_date < m.join_date
     GROUP BY s.customer_id
     ORDER BY customer_id

| customer_id | total_items | total_amount_spent |
| ----------- | ----------- | ------------------ |
| A           | 2           | 25                 |
| B           | 3           | 40                 |

**9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**

