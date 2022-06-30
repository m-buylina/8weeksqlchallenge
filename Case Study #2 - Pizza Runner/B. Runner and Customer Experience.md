# B. Runner and Customer Experience

1. How many runners signed up for each 1 week period?
```SQL
SELECT week, count(runner_id) FROM (
  SELECT
	runner_id,
	ceil(DATE_PART('doy', registration_date) / 7) AS week
  FROM pizza_runner.runners) AS runners_week
GROUP BY week
ORDER BY week
```
I decided to calculate week by myself because in this case 2021-01-01 is 53 week.


| week | count |
| ---- | ----- |
| 1    | 2     |
| 2    | 1     |
| 3    | 1     |

---
2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
``` SQL
SELECT AVG(arrive_time) FROM (
  SELECT AVG(EXTRACT(MINUTE FROM (pickup_time - order_time))) AS arrive_time,
    runner_orders_temp.order_id
  FROM customer_orders_temp
  JOIN runner_orders_temp
  ON customer_orders_temp.order_id = runner_orders_temp.order_id
  WHERE runner_orders_temp.cancellation is NULL
  GROUP BY runner_orders_temp.order_id
) as runner_time;
```

| avg    |
| ------ |
| 15.625 |

---
3. Is there any relationship between the number of pizzas and how long the order takes to prepare?
```SQL
SELECT count_pizzas, AVG(time_prepare) FROM (
  SELECT
    customer_orders_temp.order_id,
    COUNT(pizza_id) as count_pizzas,
    EXTRACT('minutes' FROM (pickup_time - order_time)) AS time_prepare
  FROM customer_orders_temp
  JOIN runner_orders_temp
  ON customer_orders_temp.order_id = runner_orders_temp.order_id
  WHERE cancellation IS NULL
  GROUP BY customer_orders_temp.order_id, EXTRACT('minutes' FROM (pickup_time - order_time))) AS prep
GROUP BY count_pizzas
ORDER BY count_pizzas;
```
| count_pizzas | avg |
| ------------ | --- |
| 1            | 12  |
| 2            | 18  |
| 3            | 29  |

---


[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65)
