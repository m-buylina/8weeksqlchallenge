
1. What is the total amount each customer spent at the restaurant?

```SQL
SELECT customer_id, SUM(price) AS total
FROM dannys_diner.sales AS sales
JOIN dannys_diner.menu AS menu
ON sales.product_id = menu.product_id
GROUP BY customer_id

```

| customer_id | total |
| ----------- | ----- |
| B           | 74    |
| C           | 36    |
| A           | 76    |

---
2. How many days has each customer visited the restaurant?

```SQL
SELECT customer_id, COUNT(DISTINCT(order_date)) AS days
FROM dannys_diner.sales AS sales
GROUP BY customer_id;
```
| customer_id | days |
| ----------- | ---- |
| A           | 4    |
| B           | 6    |
| C           | 2    |

---
3. What was the first item from the menu purchased by each customer?
```SQL
SELECT DISTINCT
	sales.customer_id,
    product_name
FROM dannys_diner.sales AS sales
JOIN (SELECT customer_id, MIN(order_date) AS min_date
      FROM dannys_diner.sales GROUP BY customer_id) AS m
ON sales.customer_id = m.customer_id
JOIN dannys_diner.menu AS menu ON menu.product_id = sales.product_id
WHERE order_date = min_date
ORDER BY customer_id
```

| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| A           | sushi        |
| B           | curry        |
| C           | ramen        |

---
4. What is the most purchased item on the menu and how many times was it purchased by all customers?
``` SQL
SELECT product_name, COUNT(*) FROM dannys_diner.sales AS sales
JOIN dannys_diner.menu AS menu
ON sales.product_id = menu.product_id
GROUP BY product_name
```

| product_name | count |
| ------------ | ----- |
| ramen        | 8     |
| sushi        | 3     |
| curry        | 4     |

Ramen is the most purchased item!

5. Which item was the most popular for each customer?
``` SQL
SELECT customer_id, product_name, count_prod FROM (
SELECT customer_id, product_name, count_prod, RANK() OVER(PARTITION BY customer_id ORDER BY count_prod DESC) AS rnk
FROM (
SELECT customer_id, product_name, COUNT(*) AS count_prod FROM dannys_diner.sales AS s
JOIN dannys_diner.menu AS m ON s.product_id = m.product_id
GROUP BY customer_id, product_name
  ) AS grouped
  ) AS ranked
WHERE rnk = 1
```

| customer_id | product_name | count_prod |
| ----------- | ------------ | ---------- |
| A           | ramen        | 3          |
| B           | ramen        | 2          |
| B           | curry        | 2          |
| B           | sushi        | 2          |
| C           | ramen        | 3          |


