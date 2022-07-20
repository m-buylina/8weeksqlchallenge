
# A. Customer Journey

Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.

Try to keep it as short as possible - you may also want to run some sort of join to make your explanations a bit easier!

```SQL
SELECT customer_id, plan_name, start_date
FROM foodie_fi.subscriptions AS subscriptions
JOIN foodie_fi.plans AS plans
ON plans.plan_id = subscriptions.plan_id
WHERE customer_id < 9
ORDER BY customer_id, start_date
```
   
| customer_id | plan_name     | start_date               |
| ----------- | ------------- | ------------------------ |
| 1           | trial         | 2020-08-01T00:00:00.000Z |
| 1           | basic monthly | 2020-08-08T00:00:00.000Z |
| 2           | trial         | 2020-09-20T00:00:00.000Z |
| 2           | pro annual    | 2020-09-27T00:00:00.000Z |
| 3           | trial         | 2020-01-13T00:00:00.000Z |
| 3           | basic monthly | 2020-01-20T00:00:00.000Z |
| 4           | trial         | 2020-01-17T00:00:00.000Z |
| 4           | basic monthly | 2020-01-24T00:00:00.000Z |
| 4           | churn         | 2020-04-21T00:00:00.000Z |
| 5           | trial         | 2020-08-03T00:00:00.000Z |
| 5           | basic monthly | 2020-08-10T00:00:00.000Z |
| 6           | trial         | 2020-12-23T00:00:00.000Z |
| 6           | basic monthly | 2020-12-30T00:00:00.000Z |
| 6           | churn         | 2021-02-26T00:00:00.000Z |
| 7           | trial         | 2020-02-05T00:00:00.000Z |
| 7           | basic monthly | 2020-02-12T00:00:00.000Z |
| 7           | pro monthly   | 2020-05-22T00:00:00.000Z |
| 8           | trial         | 2020-06-11T00:00:00.000Z |
| 8           | basic monthly | 2020-06-18T00:00:00.000Z |
| 8           | pro monthly   | 2020-08-03T00:00:00.000Z |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/rHJhRrXy5hbVBNJ6F6b9gJ/16)
