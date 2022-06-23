
# Data cleaning and tidying

Let's look at table ```customer_orders```:
columns ```exclusions``` and ```extras``` contain different forms for Null value, it should be cast to a one form

| order_id | customer_id | pizza_id | exclusions | extras | order_time               |
| -------- | ----------- | -------- | ---------- | ------ | ------------------------ |
| 1        | 101         | 1        |            |        | 2020-01-01T18:05:02.000Z |
| 2        | 101         | 1        |            |        | 2020-01-01T19:00:52.000Z |
| 3        | 102         | 1        |            |        | 2020-01-02T23:51:23.000Z |
| 3        | 102         | 2        |            |        | 2020-01-02T23:51:23.000Z |
| 4        | 103         | 1        | 4          |        | 2020-01-04T13:23:46.000Z |
| 4        | 103         | 1        | 4          |        | 2020-01-04T13:23:46.000Z |
| 4        | 103         | 2        | 4          |        | 2020-01-04T13:23:46.000Z |
| 5        | 104         | 1        | null       | 1      | 2020-01-08T21:00:29.000Z |
| 6        | 101         | 2        | null       | null   | 2020-01-08T21:03:13.000Z |
| 7        | 105         | 2        | null       | 1      | 2020-01-08T21:20:29.000Z |

- Replace 'null' with ''
``` SQL
CREATE TEMP TABLE customer_orders_temp AS 
SELECT order_id, customer_id, pizza_id,
	CASE
    	WHEN exclusions IS NULL or exclusions = 'null' THEN ''
        ELSE exclusions
    END AS exclusions,
    CASE
    	WHEN extras IS NULL or extras = 'null' THEN ''
        ELSE extras
    END AS extras,
    order_time
FROM pizza_runner.customer_orders;
```
Result:

| order_id | customer_id | pizza_id | exclusions | extras | order_time               |
| -------- | ----------- | -------- | ---------- | ------ | ------------------------ |
| 1        | 101         | 1        |            |        | 2020-01-01T18:05:02.000Z |
| 2        | 101         | 1        |            |        | 2020-01-01T19:00:52.000Z |
| 3        | 102         | 1        |            |        | 2020-01-02T23:51:23.000Z |
| 3        | 102         | 2        |            |        | 2020-01-02T23:51:23.000Z |
| 4        | 103         | 1        | 4          |        | 2020-01-04T13:23:46.000Z |
| 4        | 103         | 1        | 4          |        | 2020-01-04T13:23:46.000Z |
| 4        | 103         | 2        | 4          |        | 2020-01-04T13:23:46.000Z |
| 5        | 104         | 1        |            | 1      | 2020-01-08T21:00:29.000Z |
| 6        | 101         | 2        |            |        | 2020-01-08T21:03:13.000Z |
| 7        | 105         | 2        |            | 1      | 2020-01-08T21:20:29.000Z |
| 8        | 102         | 1        |            |        | 2020-01-09T23:54:33.000Z |
| 9        | 103         | 1        | 4          | 1, 5   | 2020-01-10T11:22:59.000Z |
| 10       | 104         | 1        |            |        | 2020-01-11T18:34:49.000Z |
| 10       | 104         | 1        | 2, 6       | 1, 4   | 2020-01-11T18:34:49.000Z |

---

The next messy table is ```runner_orders```


| order_id | runner_id | pickup_time         | distance | duration   | cancellation            |
| -------- | --------- | ------------------- | -------- | ---------- | ----------------------- |
| 1        | 1         | 2020-01-01 18:15:34 | 20km     | 32 minutes |                         |
| 2        | 1         | 2020-01-01 19:10:54 | 20km     | 27 minutes |                         |
| 3        | 1         | 2020-01-03 00:12:37 | 13.4km   | 20 mins    |                         |
| 4        | 2         | 2020-01-04 13:53:03 | 23.4     | 40         |                         |
| 5        | 3         | 2020-01-08 21:10:57 | 10       | 15         |                         |
| 6        | 3         | null                | null     | null       | Restaurant Cancellation |
| 7        | 2         | 2020-01-08 21:30:45 | 25km     | 25mins     | null                    |
| 8        | 2         | 2020-01-10 00:15:02 | 23.4 km  | 15 minute  | null                    |
| 9        | 2         | null                | null     | null       | Customer Cancellation   |
| 10       | 1         | 2020-01-11 18:50:20 | 10km     | 10minutes  | null                    |

Let's remove messy strings from ```distance``` and ```duration```. Remove null strings from other columns.

```SQL
CREATE TEMP TABLE runner_orders_temp AS
SELECT
	order_id, runner_id,
    CASE
    	WHEN pickup_time = 'null' THEN NULL 
        ELSE pickup_time
    END AS pickup_time,
    CASE 
    	WHEN distance LIKE '%km' THEN TRIM('km' from distance)
        WHEN distance = 'null' THEN NULL
		ELSE distance
    END AS distance,
    CASE
    	WHEN duration LIKE '%minutes' THEN TRIM('minutes' from duration)
        WHEN duration LIKE '%mins' THEN TRIM('mins' from duration)
        WHEN duration LIKE '%minute' THEN TRIM('minute' from duration)
        WHEN duration = 'null' THEN NULL
        ELSE duration
    END AS duration,
    CASE
    	WHEN cancellation = 'null' or cancellation = '' THEN NULL
        ELSE cancellation
    END AS cancellation
FROM pizza_runner.runner_orders;
```
Result:

| order_id | runner_id | pickup_time              | distance | duration | cancellation            |
| -------- | --------- | ------------------------ | -------- | -------- | ----------------------- |
| 1        | 1         | 2020-01-01T18:15:34.000Z | 20       | 32       |                         |
| 2        | 1         | 2020-01-01T19:10:54.000Z | 20       | 27       |                         |
| 3        | 1         | 2020-01-03T00:12:37.000Z | 13.4     | 20       |                         |
| 4        | 2         | 2020-01-04T13:53:03.000Z | 23.4     | 40       |                         |
| 5        | 3         | 2020-01-08T21:10:57.000Z | 10       | 15       |                         |
| 6        | 3         |                          |          |          | Restaurant Cancellation |
| 7        | 2         | 2020-01-08T21:30:45.000Z | 25       | 25       |                         |
| 8        | 2         | 2020-01-10T00:15:02.000Z | 23.4     | 15       |                         |
| 9        | 2         |                          |          |          | Customer Cancellation   |
| 10       | 1         | 2020-01-11T18:50:20.000Z | 10       | 10       |                         |


Looks better, but it's need to change type of ```distance``` and ```duration``` because it is still ```string```.
Let's check types
```SQL
SELECT
    column_name,
    data_type
FROM
    information_schema.columns
WHERE
    table_name = 'runner_orders_temp';
```
| column_name  | data_type         |
| ------------ | ----------------- |
| order_id     | integer           |
| runner_id    | integer           |
| pickup_time  | character varying |
| distance     | character varying |
| duration     | character varying |
| cancellation | character varying |

Sure! Moreover, ```pickup_time``` has wrong type too.
Fix it!

```SQL
ALTER TABLE runner_orders_temp
ALTER COLUMN pickup_time TYPE TIMESTAMP using to_timestamp(pickup_time, 'YYYY-MM-DD HH24:MI:SS'),
ALTER COLUMN distance TYPE DOUBLE PRECISION using (trim(distance)::double precision),
ALTER COLUMN duration TYPE INT using (trim(duration)::int);
```
Now we can calculate this data)

---




[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65)
