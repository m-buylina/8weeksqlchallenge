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

[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65)
