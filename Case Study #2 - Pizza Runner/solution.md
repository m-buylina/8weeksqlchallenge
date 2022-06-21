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
