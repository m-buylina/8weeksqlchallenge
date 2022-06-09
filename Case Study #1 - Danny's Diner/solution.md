
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
