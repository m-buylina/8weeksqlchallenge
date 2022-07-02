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

[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65)
