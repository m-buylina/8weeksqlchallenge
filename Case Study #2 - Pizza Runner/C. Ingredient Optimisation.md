# C. Ingredient Optimisation

1. What are the standard ingredients for each pizza?
``` SQL
SELECT pizza_name, array_agg(topping_name) AS ingredients FROM (
  SELECT 
  	  pizza_name,
      regexp_split_to_array(toppings, ', ')::int[] AS toppings_id
  FROM pizza_runner.pizza_recipes
  JOIN pizza_runner.pizza_names AS pizza_name
  ON pizza_name.pizza_id = pizza_recipes.pizza_id
) AS s
JOIN pizza_runner.pizza_toppings AS pizza_toppings
ON pizza_toppings.topping_id = ANY (s.toppings_id)
GROUP BY pizza_name;
```

| pizza_name | ingredients                                                    |
| ---------- | -------------------------------------------------------------- |
| Meatlovers | Bacon,BBQ Sauce,Beef,Cheese,Chicken,Mushrooms,Pepperoni,Salami |
| Vegetarian | Cheese,Mushrooms,Onions,Peppers,Tomatoes,Tomato Sauce          |

---
2. What was the most commonly added extra?
``` SQL
SELECT pizza_toppings.topping_name, COUNT(*) AS count FROM (
  SELECT string_to_array(extras, ', ')::int[] AS extras
  FROM customer_orders_temp
  WHERE extras != ''
) AS l
JOIN pizza_runner.pizza_toppings AS pizza_toppings
ON pizza_toppings.topping_id = ANY(l.extras)
GROUP BY topping_name
ORDER BY count DESC
```

| topping_name | count |
| ------------ | ----- |
| Bacon        | 4     |
| Chicken      | 1     |
| Cheese       | 1     |

---
3. What was the most common exclusion?
``` SQL
SELECT topping_name, COUNT(*) AS count FROM (
  SELECT string_to_array(exclusions, ', ')::int[] AS exclusions
  FROM customer_orders_temp
  WHERE exclusions != ''
) AS l
JOIN pizza_runner.pizza_toppings AS pizza_toppings
ON pizza_toppings.topping_id = ANY(l.exclusions)
GROUP BY topping_name
ORDER BY count DESC;
```

| topping_name | count |
| ------------ | ----- |
| Cheese       | 4     |
| Mushrooms    | 1     |
| BBQ Sauce    | 1     |

---

4. Generate an order item for each record in the customers_orders table in the format of one of the following:
- Meat Lovers
- Meat Lovers - Exclude Beef
- Meat Lovers - Extra Bacon
- Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

---
5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
6. For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"
7. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?

[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65)
