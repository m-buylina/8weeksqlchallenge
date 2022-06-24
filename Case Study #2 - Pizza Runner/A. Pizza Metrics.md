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

---

[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65)
