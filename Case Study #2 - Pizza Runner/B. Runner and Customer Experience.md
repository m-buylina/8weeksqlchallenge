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

[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65)
