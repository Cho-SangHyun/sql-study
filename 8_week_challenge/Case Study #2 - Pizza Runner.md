## sql script for sample data 
```sql
CREATE SCHEMA IF NOT EXISTS pizza_runner;
USE pizza_runner;

DROP TABLE IF EXISTS runners;
CREATE TABLE runners (
  `runner_id` INT,
  `registration_date` DATE
);

INSERT INTO runners
  (`runner_id`, `registration_date`)
VALUES
  (1, '2021-01-01'),
  (2, '2021-01-03'),
  (3, '2021-01-08'),
  (4, '2021-01-15');

DROP TABLE IF EXISTS customer_orders;
CREATE TABLE customer_orders (
  `order_id` INT,
  `customer_id` INT,
  `pizza_id` INT,
  `exclusions` VARCHAR(4),
  `extras` VARCHAR(4),
  `order_time` TIMESTAMP
);

INSERT INTO customer_orders
  (`order_id`, `customer_id`, `pizza_id`, `exclusions`, `extras`, `order_time`)
VALUES
  (1, 101, 1, '', '', '2020-01-01 18:05:02'),
  (2, 101, 1, '', '', '2020-01-01 19:00:52'),
  (3, 102, 1, '', '', '2020-01-02 23:51:23'),
  (3, 102, 2, '', NULL, '2020-01-02 23:51:23'),
  (4, 103, 1, '4', '', '2020-01-04 13:23:46'),
  (4, 103, 1, '4', '', '2020-01-04 13:23:46'),
  (4, 103, 2, '4', '', '2020-01-04 13:23:46'),
  (5, 104, 1, NULL, '1', '2020-01-08 21:00:29'),
  (6, 101, 2, NULL, NULL, '2020-01-08 21:03:13'),
  (7, 105, 2, NULL, '1', '2020-01-08 21:20:29'),
  (8, 102, 1, NULL, NULL, '2020-01-09 23:54:33'),
  (9, 103, 1, '4', '1, 5', '2020-01-10 11:22:59'),
  (10, 104, 1, NULL, NULL, '2020-01-11 18:34:49'),
  (10, 104, 1, '2, 6', '1, 4', '2020-01-11 18:34:49');

DROP TABLE IF EXISTS runner_orders;
CREATE TABLE runner_orders (
  `order_id` INT,
  `runner_id` INT,
  `pickup_time` VARCHAR(19),
  `distance` VARCHAR(7),
  `duration` VARCHAR(10),
  `cancellation` VARCHAR(23)
);

INSERT INTO runner_orders
  (`order_id`, `runner_id`, `pickup_time`, `distance`, `duration`, `cancellation`)
VALUES
  (1, 1, '2020-01-01 18:15:34', '20km', '32 minutes', ''),
  (2, 1, '2020-01-01 19:10:54', '20km', '27 minutes', ''),
  (3, 1, '2020-01-03 00:12:37', '13.4km', '20 mins', NULL),
  (4, 2, '2020-01-04 13:53:03', '23.4', '40', NULL),
  (5, 3, '2020-01-08 21:10:57', '10', '15', NULL),
  (6, 3, NULL, NULL, NULL, 'Restaurant Cancellation'),
  (7, 2, '2020-01-08 21:30:45', '25km', '25mins', NULL),
  (8, 2, '2020-01-10 00:15:02', '23.4 km', '15 minute', NULL),
  (9, 2, NULL, NULL, NULL, 'Customer Cancellation'),
  (10, 1, '2020-01-11 18:50:20', '10km', '10minutes', NULL);

DROP TABLE IF EXISTS pizza_names;
CREATE TABLE pizza_names (
  `pizza_id` INT,
  `pizza_name` TEXT
);

INSERT INTO pizza_names
  (`pizza_id`, `pizza_name`)
VALUES
  (1, 'Meatlovers'),
  (2, 'Vegetarian');

DROP TABLE IF EXISTS pizza_recipes;
CREATE TABLE pizza_recipes (
  `pizza_id` INT,
  `toppings` TEXT
);

INSERT INTO pizza_recipes
  (`pizza_id`, `toppings`)
VALUES
  (1, '1, 2, 3, 4, 5, 6, 8, 10'),
  (2, '4, 6, 7, 9, 11, 12');

DROP TABLE IF EXISTS pizza_toppings;
CREATE TABLE pizza_toppings (
  `topping_id` INT,
  `topping_name` TEXT
);

INSERT INTO pizza_toppings
  (`topping_id`, `topping_name`)
VALUES
  (1, 'Bacon'),
  (2, 'BBQ Sauce'),
  (3, 'Beef'),
  (4, 'Cheese'),
  (5, 'Chicken'),
  (6, 'Mushrooms'),
  (7, 'Onions'),
  (8, 'Pepperoni'),
  (9, 'Peppers'),
  (10, 'Salami'),
  (11, 'Tomatoes'),
  (12, 'Tomato Sauce');
```

## Question & Solution 
### A - Pizza Metrics
1. How many pizzas were ordered?
```sql
SELECT count(*) as order_count
FROM customer_orders;
```
| order_count |
| ----------- |
| 14          |
2. How many unique customer orders were made?
```sql
select count(DISTINCT order_id) as order_count
FROM customer_orders;
```
| order_count |
| ----------- |
| 10          |
3. How many successful orders were delivered by each runner?
```sql
SELECT runner_id, count(*) as success_count
FROM runner_orders
WHERE cancellation is NULL OR cancellation = ''
GROUP BY runner_id;
```
| runner_id | success_count |
| --------- | ------------- |
| 1         | 4             |
| 2         | 3             |
| 3         | 1             |
4. How many of each type of pizza was delivered?
```sql
SELECT pizza_name, count(*) as delivered_count
FROM runner_orders
INNER JOIN customer_orders ON runner_orders.order_id = customer_orders.order_id
INNER JOIN pizza_names ON customer_orders.pizza_id = pizza_names.pizza_id
WHERE cancellation is NULL OR cancellation = ''
GROUP BY pizza_name;
```
| pizza_name | delivered_count |
| ---------- | --------------- |
| Meatlovers | 9               |
| Vegetarian | 3               |
5. How many Vegetarian and Meatlovers were ordered by each customer?
```sql
SELECT customer_id, pizza_name, count(*) as order_count
FROM customer_orders
INNER JOIN pizza_names ON customer_orders.pizza_id = pizza_names.pizza_id
WHERE pizza_name in ("Meatlovers", "Vegetarian")
GROUP BY customer_id, pizza_name
ORDER BY customer_id, pizza_name;
```
| customer_id | pizza_name | order_count |
| ----------- | ---------- | ----------- |
| 101         | Meatlovers | 2           |
| 101         | Vegetarian | 1           |
| 102         | Meatlovers | 2           |
| 102         | Vegetarian | 1           |
| 103         | Meatlovers | 3           |
| 103         | Vegetarian | 1           |
| 104         | Meatlovers | 3           |
| 105         | Vegetarian | 1           |
6. What was the maximum number of pizzas delivered in a single order?
```sql
SELECT count(*) as pizza_count
FROM customer_orders
GROUP BY order_id
ORDER BY pizza_count DESC
LIMIT 1;
```
| pizza_count |
| ----------- |
| 3           |
7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
```sql
WITH changed AS (
    SELECT 
        customer_id, 
        CASE
            WHEN exclusions is not NULL AND exclusions != ''
            THEN 1
            WHEN extras is not NULL AND extras != ''
            THEN 1
            ELSE 0
        END as is_changed
    FROM customer_orders
    INNER JOIN runner_orders ON customer_orders.order_id = runner_orders.order_id
    WHERE distance is not NULL
)

SELECT 
    customer_id, 
    sum(is_changed) as changed_count,
    count(*) - sum(is_changed) as no_changed_count
FROM changed
GROUP BY customer_id
ORDER BY customer_id;
```
| customer_id | changed_count | no_changed_count |
| ----------- | ------------- | ---------------- |
| 101         | 0             | 2                |
| 102         | 0             | 3                |
| 103         | 3             | 0                |
| 104         | 2             | 1                |
| 105         | 1             | 0                |

8. How many pizzas were delivered that had both exclusions and extras?
```sql
SELECT count(*) as pizza_count
FROM customer_orders
JOIN runner_orders on customer_orders.order_id = runner_orders.order_id
WHERE 
    exclusions is not NULL AND exclusions != '' 
    AND extras is not NULL AND extras != ''
    AND distance is not NULL;
```
| pizza_count |
| ----------- |
| 1           |
9. What was the total volume of pizzas ordered for each hour of the day?
```sql
SELECT HOUR(order_time) as order_hour, count(*) as pizza_count
FROM customer_orders
GROUP BY HOUR(order_time)
ORDER BY order_hour;
```
| order_hour | pizza_count |
| ---------- | ----------- |
| 11         | 1           |
| 13         | 3           |
| 18         | 3           |
| 19         | 1           |
| 21         | 3           |
| 23         | 3           |
10. What was the volume of orders for each day of the week?
```sql
SELECT 
    DAYNAME(order_time) as order_day_of_the_week, 
    count(*) as pizza_count
FROM customer_orders
GROUP BY DAYNAME(order_time)
ORDER BY order_day_of_the_week;
```
| order_day_of_the_week | pizza_count |
| --------------------- | ----------- |
| Friday                | 1           |
| Saturday              | 5           |
| Thursday              | 3           |
| Wednesday             | 5           |

### B - Runner and Customer Experience
1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
```sql
SELECT 
    DATEDIFF(registration_date, '2021-01-01') DIV 7 + 1 as week_number, 
    count(*) as runners_count
FROM runners
GROUP BY DATEDIFF(registration_date, '2021-01-01') DIV 7 + 1
ORDER BY week_number;
```
| week_number | runners_count |
| ----------- | ------------- |
| 1           | 2             |
| 2           | 1             |
| 3           | 1             |
2. What was the average time in minutes it took for each runner to arrive at the Pizza  Runner HQ to pickup the order?
```sql
SELECT 
    runner_id, 
    AVG(TIMESTAMPDIFF(MINUTE, order_time, pickup_time)) as average_time
FROM customer_orders
INNER JOIN runner_orders ON customer_orders.order_id = runner_orders.order_id
WHERE pickup_time is not NULL
GROUP BY runner_id
ORDER BY runner_id;
```
| runner_id | average_time |
| --------- | ------------ |
| 1         | 15.3333      |
| 2         | 23.4000      |
| 3         | 10.0000      |
3. Is there any relationship between the number of pizzas and how long the order takes to prepare?
```sql
WITH avg_prepare_time AS (
    SELECT 
        count(*) as pizza_count,
        AVG(TIMESTAMPDIFF(MINUTE, order_time, pickup_time)) as prepare_time
    FROM customer_orders
    INNER JOIN runner_orders ON customer_orders.order_id = runner_orders.order_id
    WHERE pickup_time is not NULL
    GROUP BY customer_orders.order_id
)

SELECT pizza_count, AVG(prepare_time) as prepare_time
FROM avg_prepare_time
GROUP BY pizza_count
ORDER BY pizza_count;
```
| pizza_count | prepare_time |
| ----------- | ------------ |
| 1           | 12.00000000  |
| 2           | 18.00000000  |
| 3           | 29.00000000  |
4. What was the average distance travelled for each customer?
```sql
SELECT 
    customer_id, 
    AVG(CONVERT(REGEXP_REPLACE(distance, '[^0-9.]', ''), FLOAT)) AS average_distance
FROM customer_orders
INNER JOIN runner_orders ON customer_orders.order_id = runner_orders.order_id
WHERE distance is not NULL
GROUP BY customer_id
ORDER BY customer_id;
```
| customer_id | average_distance |
| ----------- | ---------------- |
| 101         | 20               |
| 102         | 16.733332951863606 |
| 103         | 23.399999618530273 |
| 104         | 10               |
| 105         | 25               |
5. What was the difference between the longest and shortest delivery times for all orders?
```sql
WITH delivery_times AS (
  SELECT TIMESTAMPDIFF(MINUTE, order_time, pickup_time) as delivery_time
  FROM customer_orders
  INNER JOIN runner_orders ON customer_orders.order_id = runner_orders.order_id
  WHERE order_time is not NULL AND pickup_time is not NULL
)

SELECT MAX(delivery_time) - MIN(delivery_time) as difference
FROM delivery_times;
```
| difference  |
| ----------- |
| 19          |
6. What was the average speed for each runner for each delivery and do you notice any trend for these values?
```sql
SELECT
  runner_id,
  AVG(
    CONVERT(
      CONVERT(REGEXP_REPLACE(distance, '[^0-9.]', ''), FLOAT)
      / CONVERT(REGEXP_REPLACE(duration, '[^0-9]', ''), FLOAT),
      DECIMAL(5, 3)
    )
  ) as avg_speed
FROM runner_orders
WHERE distance is not NULL and duration is not NULL
GROUP BY runner_id
ORDER BY runner_id;
```
| runner_id | avg_speed |
| --------- | --------- |
| 1         | 0.7590000 |
| 2         | 1.0483333 |
| 3         | 0.6670000 |
7. What is the successful delivery percentage for each runner?
```sql
WITH delivery AS (
  SELECT runner_id, IF(cancellation is NULL OR cancellation = '', 1, 0) as is_success
  FROM runner_orders
)

SELECT runner_id, SUM(is_success) / count(*) as success_percentage
FROM delivery
GROUP BY runner_id
ORDER BY runner_id;
```
| runner_id | success_percentage |
| --------- | ------------------ |
| 1         | 1.0000             |
| 2         | 0.7500             |  
| 3         | 0.5000             |