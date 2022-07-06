# Case Study #1 - Danny's Diner

### Full description of the task and relationship diagram: https://8weeksqlchallenge.com/case-study-1/

---
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

---
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

---
6. Which item was purchased first by the customer after they became a member?
``` SQL
SELECT customer_id, join_date, order_date, product_name FROM (
SELECT s.customer_id, join_date, MIN(product_id) AS product_id, MIN(order_date) AS order_date
FROM dannys_diner.sales as s
JOIN dannys_diner.members AS memb ON s.customer_id = memb.customer_id
WHERE order_date >= join_date 
GROUP BY s.customer_id, join_date) AS joined
JOIN dannys_diner.menu AS menu ON menu.product_id = joined.product_id
```


| customer_id | join_date                | order_date               | product_name |
| ----------- | ------------------------ | ------------------------ | ------------ |
| B           | 2021-01-09               | 2021-01-11               | sushi        |
| A           | 2021-01-07               | 2021-01-07               | curry        |

---
7. Which item was purchased just before the customer became a member?
``` SQL
SELECT customer_id, join_date, product_name, order_date FROM (
SELECT s.customer_id, join_date, product_name, order_date,
RANK() OVER(PARTITION BY s.customer_id ORDER BY order_date DESC) AS rnk
FROM dannys_diner.sales AS s
JOIN dannys_diner.members AS mem ON mem.customer_id = s.customer_id
JOIN dannys_diner.menu AS m ON s.product_id = m.product_id
WHERE order_date < join_date) AS ranked
WHERE rnk = 1
```

| customer_id | join_date                | product_name | order_date               |
| ----------- | ------------------------ | ------------ | ------------------------ |
| A           | 2021-01-07               | sushi        | 2021-01-01               |
| A           | 2021-01-07               | curry        | 2021-01-01               |
| B           | 2021-01-09               | sushi        | 2021-01-04               |

---
8. What is the total items and amount spent for each member before they became a member?
``` SQL
SELECT sales.customer_id, SUM(price) AS total_amount, COUNT(*) AS total_items
FROM dannys_diner.sales AS sales
JOIN dannys_diner.members AS members
ON sales.customer_id = members.customer_id
JOIN dannys_diner.menu AS menu
ON menu.product_id = sales.product_id
WHERE order_date < join_date
GROUP BY sales.customer_id
```

| customer_id | total_amount | total_items |
| ----------- | ------------ | ----------- |
| B           | 40           | 3           |
| A           | 25           | 2           |

---
9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
``` SQL
SELECT customer_id, SUM (points) AS total_points FROM (
	SELECT customer_id, product_name,
	CASE
		WHEN product_name = 'sushi' THEN price * 20
		ELSE price * 10
	END AS points
	FROM dannys_diner.sales AS sales
	JOIN dannys_diner.menu AS menu
	ON sales.product_id = menu.product_id) AS p
GROUP BY customer_id
ORDER BY customer_id
```
| customer_id | total_points |
| ----------- | --- |
| A           | 860 |
| B           | 940 |
| C           | 360 |

---
10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
``` SQL
SELECT customer_id, SUM(points) AS total_points FROM (
	SELECT sales.customer_id, order_date, join_date, product_name, 
  	CASE
    	WHEN (6 >= (order_date - join_date) AND (order_date - join_date) >= 0)
  				OR product_name = 'sushi'
        	THEN price * 10 * 2
  		ELSE price * 10
  	END AS points
	FROM dannys_diner.sales AS sales
	JOIN dannys_diner.members AS members
	ON members.customer_id = sales.customer_id
	JOIN dannys_diner.menu AS menu
    ON sales.product_id = menu.product_id
  	WHERE order_date <= '2021-01-31'
) AS points_members
GROUP BY customer_id
```

| customer_id | total_points |
| ----------- | ------------ |
| A           | 1370         |
| B           | 820          |

---
