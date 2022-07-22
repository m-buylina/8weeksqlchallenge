# B. Data Analysis Questions.md

1. How many customers has Foodie-Fi ever had?

```SQL
SELECT COUNT(DISTINCT customer_id)
FROM foodie_fi.subscriptions
```
| count |
| ----- |
| 1000  |
