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

``` SQL
WITH prep_orders AS (
  SELECT 
  	ROW_NUMBER () OVER(ORDER BY order_id) as row_id,
    order_id,
    customer_id,
    pizza_id, 
    string_to_array(exclusions, ', ')::int[] AS exclusions,
    string_to_array(extras, ', ')::int[] AS extras
  FROM customer_orders_temp
)

SELECT
order_id, pizza_id, exclusions, extras,
CONCAT(
CASE 
  WHEN pizza_id = 1 THEN 'Meat Lovers'
  WHEN pizza_id = 2 THEN 'Vegetarian'
END,
CASE 
  WHEN array_length(exclusions_str, 1) IS NOT NULL THEN ' - Exclude ' || 
  array_to_string(exclusions_str, ', ')
END,
  CASE 
  WHEN array_length(extras_str, 1) IS NOT NULL THEN ' - Extra ' || 
  array_to_string(extras_str, ', ')
END
) AS lovers

FROM (
  SELECT 
  row_id, order_id, pizza_id, exclusions, extras, exclusions_str,
  array_remove(array_agg(topping_name), NULL) AS extras_str
  FROM (
    SELECT row_id, order_id, pizza_id, exclusions, extras,
    array_remove(array_agg(topping_name), NULL) AS exclusions_str FROM prep_orders
    LEFT JOIN pizza_runner.pizza_toppings AS pizza_toppings
    ON pizza_toppings.topping_id = ANY(prep_orders.exclusions)
    GROUP BY row_id, order_id, pizza_id, exclusions, extras
  ) AS l
  LEFT JOIN pizza_runner.pizza_toppings AS pizza_toppings
  ON pizza_toppings.topping_id = ANY(l.extras)
  GROUP BY row_id, order_id, pizza_id, exclusions, extras, exclusions_str
) AS r;
```

| order_id | pizza_id | exclusions | extras | lovers                                                           |
| -------- | -------- | ---------- | ------ | ---------------------------------------------------------------- |
| 7        | 2        |            | 1      | Vegetarian - Extra Bacon                                         |
| 9        | 1        | 4          | 1,5    | Meat Lovers - Exclude Cheese - Extra Bacon, Chicken              |
| 1        | 1        |            |        | Meat Lovers                                                      |
| 4        | 2        | 4          |        | Vegetarian - Exclude Cheese                                      |
| 3        | 2        |            |        | Vegetarian                                                       |
| 5        | 1        |            | 1      | Meat Lovers - Extra Bacon                                        |
| 2        | 1        |            |        | Meat Lovers                                                      |
| 3        | 1        |            |        | Meat Lovers                                                      |
| 10       | 1        |            |        | Meat Lovers                                                      |
| 10       | 1        | 2,6        | 1,4    | Meat Lovers - Exclude BBQ Sauce, Mushrooms - Extra Bacon, Cheese |
| 4        | 1        | 4          |        | Meat Lovers - Exclude Cheese                                     |
| 6        | 2        |            |        | Vegetarian                                                       |
| 4        | 1        | 4          |        | Meat Lovers - Exclude Cheese                                     |
| 8        | 1        |            |        | Meat Lovers                                                      |


---
5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
For example: "2xBacon, Beef, ... , Salami"

``` SQL
WITH pizza_orig_toppings AS (
  SELECT pizza_id, array_agg(topping_name) AS toppings
  FROM (
    SELECT  pizza_id, string_to_array(toppings, ', ')::int[] AS toppings
    FROM pizza_runner.pizza_recipes
  ) AS p
  JOIN pizza_runner.pizza_toppings AS pizza_toppings
  ON pizza_toppings.topping_id = ANY(p.toppings)
  GROUP BY pizza_id
),
customer_orders_arr AS (
  SELECT order_id, customer_id, pizza_id,
    ARRAY_REMOVE(array_agg(topping_name), NULL) AS extras_str,
    extras
  FROM (
    SELECT 
      ROW_NUMBER() OVER(ORDER BY order_id) AS row_id,
      order_id, customer_id, pizza_id,
    string_to_array(extras, ', ')::int[] AS extras
  FROM customer_orders_temp) AS f
  LEFT JOIN pizza_runner.pizza_toppings AS pizza_toppings
  ON pizza_toppings.topping_id = ANY(f.extras)
  GROUP BY row_id, order_id, customer_id, pizza_id, extras
)

SELECT 
  order_id, customer_id, customer_orders_arr.pizza_id,
  extras,
  array_to_string (
  CASE 
  	WHEN extras_str[1] IS NOT NULL
    THEN array_replace(array_replace(toppings, extras_str[1], concat('2x',extras_str[1])), extras_str[2], concat('2x',extras_str[2]))
    ELSE toppings
  END,  ', ')AS resullt
FROM customer_orders_arr
JOIN pizza_orig_toppings
ON customer_orders_arr.pizza_id = pizza_orig_toppings.pizza_id
```

| order_id | customer_id | pizza_id | extras | resullt                                                                   |
| -------- | ----------- | -------- | ------ | ------------------------------------------------------------------------- |
| 10       | 104         | 1        | 1,4    | 2xBacon, BBQ Sauce, Beef, 2xCheese, Chicken, Mushrooms, Pepperoni, Salami |
| 4        | 103         | 1        |        | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami     |
| 4        | 103         | 1        |        | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami     |
| 4        | 103         | 2        |        | Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce                |
| 10       | 104         | 1        |        | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami     |
| 6        | 101         | 2        |        | Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce                |
| 2        | 101         | 1        |        | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami     |
| 5        | 104         | 1        | 1      | 2xBacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami   |
| 9        | 103         | 1        | 1,5    | 2xBacon, BBQ Sauce, Beef, Cheese, 2xChicken, Mushrooms, Pepperoni, Salami |
| 1        | 101         | 1        |        | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami     |
| 3        | 102         | 1        |        | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami     |
| 8        | 102         | 1        |        | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami     |
| 7        | 105         | 2        | 1      | Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce                |
| 3        | 102         | 2        |        | Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce                |

---

7. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?

[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65)
