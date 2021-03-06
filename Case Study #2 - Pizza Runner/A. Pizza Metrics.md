## A. Pizza Metrics

1. How many pizzas were ordered?
``` SQL
SELECT COUNT(*)
FROM pizza_runner.customer_orders
```
| count |
| ----- |
| 14    |

---
2. How many unique customer orders were made?
``` SQL
SELECT COUNT(DISTINCT order_id)
FROM pizza_runner.customer_orders
```

| count |
| ----- |
| 10    |

---
3. How many successful orders were delivered by each runner?
```SQL
SELECT runner_id, COUNT(order_id) FROM runner_orders_temp
WHERE cancellation IS NULL OR distance IS NOT NULL
GROUP BY runner_id;
```

| runner_id | count |
| --------- | ----- |
| 3         | 1     |
| 2         | 3     |
| 1         | 4     |

4. How many of each type of pizza was delivered?
``` SQL
SELECT customer_orders_temp.pizza_id, pizza_name, COUNT(*) FROM runner_orders_temp
JOIN customer_orders_temp
ON runner_orders_temp.order_id = customer_orders_temp.order_id
JOIN pizza_runner.pizza_names AS pizza_names
ON customer_orders_temp.pizza_id = pizza_names.pizza_id
WHERE cancellation IS NULL
GROUP BY customer_orders_temp.pizza_id, pizza_name
```

| pizza_id | pizza_name | count |
| -------- | ---------- | ----- |
| 1        | Meatlovers | 9     |
| 2        | Vegetarian | 3     |

5. How many Vegetarian and Meatlovers were ordered by each customer?
``` SQL
SELECT customer_id, pizza_names, COUNT(pizza_names) FROM customer_orders_temp
JOIN pizza_runner.pizza_names AS pizza_names
ON customer_orders_temp.pizza_id = pizza_names.pizza_id
GROUP BY customer_id, pizza_names
ORDER BY customer_id
```

| customer_id | pizza_names    | count |
| ----------- | -------------- | ----- |
| 101         | (1,Meatlovers) | 2     |
| 101         | (2,Vegetarian) | 1     |
| 102         | (1,Meatlovers) | 2     |
| 102         | (2,Vegetarian) | 1     |
| 103         | (1,Meatlovers) | 3     |
| 103         | (2,Vegetarian) | 1     |
| 104         | (1,Meatlovers) | 3     |
| 105         | (2,Vegetarian) | 1     |

---
6. What was the maximum number of pizzas delivered in a single order?
```SQL
SELECT order_id, COUNT(*) as count_p FROM customer_orders_temp
GROUP BY order_id
ORDER BY count_p DESC 
LIMIT 1
```

| order_id | count_p |
| -------- | ------- |
| 4        | 3       |

---
7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
```SQL
SELECT
customer_id,
SUM(CASE WHEN exclusions != '' OR extras != '' THEN 1 ELSE 0 END) AS changed,
SUM(CASE WHEN exclusions = '' AND extras = '' THEN 1 ELSE 0 END) AS not_changed
FROM customer_orders_temp
JOIN runner_orders_temp ON customer_orders_temp.order_id = runner_orders_temp.order_id
WHERE cancellation IS NULL
GROUP BY customer_id
```

| customer_id | changed | not_changed |
| ----------- | ------- | ----------- |
| 101         | 0       | 2           |
| 102         | 0       | 3           |
| 103         | 3       | 0           |
| 104         | 2       | 1           |
| 105         | 1       | 0           |

---
8. How many pizzas were delivered that had both exclusions and extras?
``` SQL
SELECT COUNT(*) FROM customer_orders_temp
JOIN runner_orders_temp ON customer_orders_temp.order_id = runner_orders_temp.order_id
WHERE exclusions != '' AND extras != '' AND cancellation IS NULL
```

| count |
| ----- |
| 1     |

---
9. What was the total volume of pizzas ordered for each hour of the day?
``` SQL
SELECT
  DATE_PART('hour', order_time) AS hour_order,
  COUNT(*)
FROM customer_orders_temp
GROUP BY hour_order
ORDER BY hour_order
```
| hour_order | count |
| ---------- | ----- |
| 11         | 1     |
| 13         | 3     |
| 18         | 3     |
| 19         | 1     |
| 21         | 3     |
| 23         | 3     |

---
10. What was the volume of orders for each day of the week?
``` SQL
SELECT
  to_char(order_time, 'Day') AS week_day,
  COUNT(DISTINCT order_id)
FROM customer_orders_temp
GROUP BY to_char(order_time, 'Day')
```

| week_day  | count |
| --------- | ----- |
| Friday    | 1     |
| Saturday  | 2     |
| Thursday  | 2     |
| Wednesday | 5     |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65)
