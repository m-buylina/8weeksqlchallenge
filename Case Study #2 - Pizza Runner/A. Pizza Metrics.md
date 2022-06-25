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

---

[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65)
---

[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65)
