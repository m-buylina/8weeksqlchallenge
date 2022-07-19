# D. Pricing and Ratings
1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money
 has Pizza Runner made so far if there are no delivery fees?

``` SQL
CREATE TABLE IF NOT EXISTS prices (
  pizza_id INT NOT NULL,
  price INT NOT NULL
);
INSERT INTO prices
  ("pizza_id", "price")
VALUES
  (1, 12),
  (2, 10);
  
SELECT SUM(price)
FROM customer_orders_temp
JOIN prices
ON prices.pizza_id = customer_orders_temp.pizza_id
JOIN runner_orders_temp
ON runner_orders_temp.order_id = customer_orders_temp.order_id
WHERE cancellation IS NULL
```
| sum |
| --- |
| 138 |

---

2. What if there was an additional $1 charge for any pizza extras? Add cheese is $1 extra
``` SQL
SELECT 
  SUM(COALESCE(array_length(string_to_array(extras, ', '), 1), 0) + price) AS total_price
FROM customer_orders_temp 
JOIN prices
ON customer_orders_temp.pizza_id = prices.pizza_id
JOIN runner_orders_temp
ON runner_orders_temp.order_id = customer_orders_temp.order_id
WHERE cancellation IS NULL
```

| total_price |
| ----------- |
| 142         |

---
3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.

```SQL
CREATE TABLE IF NOT EXISTS runner_rates (
  order_id int NOT NULL,
  rate int NOT NULL
);

INSERT INTO runner_rates ("order_id", "rate")
VALUES
  (1, 4),
  (2, 5),
  (3, 5),
  (4, 2),
  (5, 4),
  (7, 5),
  (8, 5),
  (10, 3);

SELECT * FROM runner_rates
```

| order_id | rate |
| -------- | ---- |
| 1        | 4    |
| 2        | 5    |
| 3        | 5    |
| 4        | 2    |
| 5        | 4    |
| 7        | 5    |
| 8        | 5    |
| 10       | 3    |

---
4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful   deliveries?
- customer_id
- order_id
- runner_id
- rating
- order_time
- pickup_time
- Time between order and pickup
- Delivery duration
- Average speed
- Total number of pizzas

``` SQL
WITH grouped_pizza AS (
  SELECT order_id, customer_id, order_time, COUNT(*) AS total_pizzas
  FROM customer_orders_temp
  GROUP BY order_id, customer_id, order_time
)

SELECT 
  customer_id, grouped_pizza.order_id, runner_orders_temp.runner_id, 
  rating, order_time, pickup_time,
  EXTRACT('minute' from pickup_time - order_time) AS time_order_pickup,
  duration,
  ROUND(distance / duration * 60) AS avg_speed_km_h,
  total_pizzas
FROM grouped_pizza
JOIN runner_orders_temp
ON runner_orders_temp.order_id = grouped_pizza.order_id
JOIN runner_rates
ON runner_rates.order_id = runner_orders_temp.order_id
```

| customer_id | order_id | runner_id | rating | order_time               | pickup_time              | time_order_pickup | duration | avg_speed_km_h | total_pizzas |
| ----------- | -------- | --------- | ------ | ------------------------ | ------------------------ | ----------------- | -------- | -------------- | ------------ |
| 101         | 1        | 1         | 4      | 2020-01-01T18:05:02.000Z | 2020-01-01T18:15:34.000Z | 10                | 32       | 38             | 1            |
| 101         | 2        | 1         | 5      | 2020-01-01T19:00:52.000Z | 2020-01-01T19:10:54.000Z | 10                | 27       | 44             | 1            |
| 102         | 3        | 1         | 5      | 2020-01-02T23:51:23.000Z | 2020-01-03T00:12:37.000Z | 21                | 20       | 40             | 2            |
| 103         | 4        | 2         | 2      | 2020-01-04T13:23:46.000Z | 2020-01-04T13:53:03.000Z | 29                | 40       | 35             | 3            |
| 104         | 5        | 3         | 4      | 2020-01-08T21:00:29.000Z | 2020-01-08T21:10:57.000Z | 10                | 15       | 40             | 1            |
| 105         | 7        | 2         | 5      | 2020-01-08T21:20:29.000Z | 2020-01-08T21:30:45.000Z | 10                | 25       | 60             | 1            |
| 102         | 8        | 2         | 5      | 2020-01-09T23:54:33.000Z | 2020-01-10T00:15:02.000Z | 20                | 15       | 94             | 1            |
| 104         | 10       | 1         | 3      | 2020-01-11T18:34:49.000Z | 2020-01-11T18:50:20.000Z | 15                | 10       | 60             | 2            |

---
5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?

``` SQL
SELECT SUM(price + distance * 0.3) AS total_sum
FROM customer_orders_temp
JOIN prices ON prices.pizza_id = customer_orders_temp.pizza_id
JOIN runner_orders_temp ON runner_orders_temp.order_id = customer_orders_temp.order_id
WHERE cancellation IS NULL
```
IS NULL;

| total_sum |
| --------- |
| 202.62    |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65)
