## sql script for sample data  
```sql
CREATE SCHEMA dannys_diner;
USE dannys_diner;

CREATE TABLE sales (
  customer_id VARCHAR(1),
  order_date DATE,
  product_id INT
);

INSERT INTO sales
  (customer_id, order_date, product_id)
VALUES
  ('A', '2021-01-01', 1),
  ('A', '2021-01-01', 2),
  ('A', '2021-01-07', 2),
  ('A', '2021-01-10', 3),
  ('A', '2021-01-11', 3),
  ('A', '2021-01-11', 3),
  ('B', '2021-01-01', 2),
  ('B', '2021-01-02', 2),
  ('B', '2021-01-04', 1),
  ('B', '2021-01-11', 1),
  ('B', '2021-01-16', 3),
  ('B', '2021-02-01', 3),
  ('C', '2021-01-01', 3),
  ('C', '2021-01-01', 3),
  ('C', '2021-01-07', 3);
 

CREATE TABLE menu (
  product_id INT,
  product_name VARCHAR(5),
  price INT
);

INSERT INTO menu
  (product_id, product_name, price)
VALUES
  (1, 'sushi', 10),
  (2, 'curry', 15),
  (3, 'ramen', 12);
  

CREATE TABLE members (
  customer_id VARCHAR(1),
  join_date DATE
);

INSERT INTO members
  (customer_id, join_date)
VALUES
  ('A', '2021-01-07'),
  ('B', '2021-01-09');
```

## Question & Solution  
1. What is the total amount each customer spent at the restaurant?  
```sql
SELECT ds.customer_id, sum(dm.price) as total_amount
FROM dannys_diner.sales as ds
INNER JOIN dannys_diner.menu as dm ON ds.product_id = dm.product_id
GROUP BY ds.customer_id;
```
| customer_id | total_amount |
| ----------- | ------------ |
| A           | 76           |
| B           | 74           |
| C           | 36           |

2. How many days has each customer visited the restaurant?
```sql
SELECT customer_id, count(distinct(order_date)) as visited_days
FROM dannys_diner.sales
GROUP BY customer_id;
```
| customer_id | visited_days |
| ----------- | ------------ |
| A           | 4           |
| B           | 6           |
| C           | 2           |

3. What was the first item from the menu purchased by each customer?
```sql
SELECT DISTINCT customer_id, product_name
FROM sales
INNER JOIN menu ON sales.product_id = menu.product_id 
WHERE (customer_id, order_date) in (
    SELECT customer_id, min(order_date)
    FROM sales
    GROUP BY customer_id
);
```
| customer_id | product_name |
| ----------- | ------------ |
| A           | sushi           |
| A           | curry           |
| B           | curry           |
| C           | ramen           |
4. What is the most purchased item on the menu and how many times was it purchased by all customers?
```sql
SELECT product_name, count(*) as purchased_count
FROM sales
INNER JOIN menu ON sales.product_id = menu.product_id
GROUP BY product_name
ORDER BY purchased_count DESC
LIMIT 1;
```
| product_name | purchased_count |
| ------------ | --------------- |
| ramen        | 8               |
5. Which item was the most popular for each customer?
```sql
WITH most_popular as (
    SELECT 
        customer_id, 
        product_id, 
        count(*) OVER (PARTITION BY customer_id, product_id) as count_per_product
    FROM sales
    ORDER BY customer_id, product_id
)

SELECT DISTINCT customer_id, product_name, count_per_product
FROM most_popular
INNER JOIN menu ON most_popular.product_id = menu.product_id
WHERE (customer_id, count_per_product) in (
    SELECT customer_id, max(count_per_product)
    FROM most_popular
    GROUP BY customer_id
)
ORDER BY customer_id;
```
| customer_id | product_name | count_per_product |
| ----------- | ------------ | ----------------- |
| A           | ramen        | 3                 |
| B           | sushi        | 2                 |
| B           | curry        | 2                 |
| B           | ramen        | 2                 |
| C           | ramen        | 3                 |
6. Which item was purchased first by the customer after they became a member?
```sql
WITH purchased_first as (
    SELECT 
        sales.customer_id, 
        product_name, 
        RANK() OVER (PARTITION BY sales.customer_id ORDER BY order_date) as order_rank
    FROM sales
    INNER JOIN members ON sales.customer_id = members.customer_id
    INNER JOIN menu ON sales.product_id = menu.product_id
    WHERE order_date > join_date
)

SELECT customer_id, product_name
FROM purchased_first
WHERE order_rank = 1;
```
| customer_id | product_name |
| ----------- | ------------ |
| A           | ramen           |
| B           | sushi           |
7. Which item was purchased just before the customer became a member?
```sql
WITH purchased_first as (
    SELECT 
        sales.customer_id, 
        product_name,
        order_date,
        join_date, 
        RANK() OVER (PARTITION BY sales.customer_id ORDER BY order_date DESC) as order_rank
    FROM sales
    INNER JOIN members ON sales.customer_id = members.customer_id
    INNER JOIN menu ON sales.product_id = menu.product_id
    WHERE order_date < join_date
)

SELECT customer_id, product_name
FROM purchased_first
WHERE order_rank = 1;
```
| customer_id | product_name |
| ----------- | ------------ |
| A           | sushi           |
| A           | curry           |
| B           | sushi           |
8. What is the total items and amount spent for each member before they became a member?
```sql
SELECT 
    sales.customer_id, 
    count(*) as total_items, 
    sum(price) as total_amount
FROM sales
LEFT JOIN members ON sales.customer_id = members.customer_id
INNER JOIN menu ON sales.product_id = menu.product_id
WHERE sales.order_date < members.join_date OR members.join_date is NULL
GROUP BY sales.customer_id;
```
| customer_id | total_items | total_amount |
| ----------- | ------------ | ----------------- |
| A           | 2        | 25                 |
| B           | 3        | 40                 |
| C           | 3        | 36                 |
9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```sql
WITH point_per_customer as (
    SELECT 
        customer_id, 
        CASE
            WHEN product_name = "sushi"
            THEN price * 20
            ELSE price * 10
        END as price_point
    FROM sales
    INNER JOIN menu on sales.product_id = menu.product_id
)

SELECT customer_id, sum(price_point) as total_point
FROM point_per_customer
GROUP BY customer_id;
```
| customer_id | total_point |
| ----------- | ------------ |
| A           | 860           |
| B           | 940           |
| C           | 360           |
10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
```sql
WITH point_per_customer as (
    SELECT 
        sales.customer_id,
        CASE
            WHEN DATEDIFF(order_date, join_date) BETWEEN 0 AND 6
            THEN price * 20
            WHEN product_name = "sushi"
            THEN price * 20
            ELSE price * 10
        END as price_point
    FROM sales
    INNER JOIN members ON sales.customer_id = members.customer_id
    INNER JOIN menu ON sales.product_id = menu.product_id
    WHERE order_date < "2021-02-01"
)

SELECT customer_id, sum(price_point) as total_point
FROM point_per_customer
GROUP BY customer_id
ORDER BY customer_id;
```
| customer_id | total_point |
| ----------- | ------------ |
| A           | 1370           |
| B           | 820           |