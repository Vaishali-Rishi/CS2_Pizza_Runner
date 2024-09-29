# Case Study #2 Pizza Runner


![logo](https://github.com/user-attachments/assets/e724c024-2711-4b13-a1b7-cbf38cb8c118)

You can find all the details here - [Link](https://8weeksqlchallenge.com/case-study-2/)

### Table of Contents:
- Problem Statement
- ER Diagram
- Data Cleaning & Transformation
- Questions & Solutions

### Problem Statement
---
Danny was scrolling through his Instagram feed when something really caught his eye - “80s Retro Styling and Pizza Is The Future!” Danny was sold on the idea, but he knew that pizza alone was not going to help him get seed funding to expand his new Pizza Empire - so he had one more genius idea to combine with it - he was going to Uberize it - and so Pizza Runner was launched!
Danny started by recruiting “runners” to deliver fresh pizza from Pizza Runner Headquarters (otherwise known as Danny’s house) and also maxed out his credit card to pay freelance developers to build a mobile app to accept orders from customers.
He wants to use the data to answer a few simple questions so he can better direct his runners and optimise Pizza Runner’s operations.
Danny has shared 6 key datasets for this case study:
- runners
- customer_orders
- runner_orders
- pizza_names
- pizza_recipes
- pizza_toppings

### Entity Relationship Diagram
---
![ER](https://github.com/user-attachments/assets/2e2d6183-e3c3-4b96-81a7-135ba86c163d)

### Data Cleaning & Transformation
---
#### Data Cleaning in customer_orders table- exclusions column
```sql
UPDATE customer_orders 
SET exclusions = null 
WHERE (exclusions = 'null') OR (exclusions = '');
```
#### Data Cleaning in customer_orders table- extras column
```sql
UPDATE customer_orders 
SET extras = null 
WHERE (extras = 'null') OR (extras = '');
```
#### Data Cleaning in runner_orders table- pickup_time, distance and duration column
```sql
UPDATE runner_orders
SET pickup_time = CASE WHEN pickup_time = 'null' THEN NULL ELSE pickup_time END,
    distance = CASE WHEN distance = 'null' THEN NULL ELSE distance END,
    duration = CASE WHEN duration = 'null' THEN NULL ELSE duration END
WHERE pickup_time = 'null' 
   OR distance = 'null' 
   OR duration = 'null';

```
#### Data Cleaning in runner_orders table- cancellation column
```sql
UPDATE runner_orders 
SET cancellation = null 
WHERE (cancellation = 'null') OR (cancellation = '');

```
#### Data Transformation in runner_orders table- Remove 'km' from distance column
```sql
UPDATE runner_orders 
SET distance = REPLACE(distance, 'km', '')
WHERE distance LIKE '%km';

```
#### Data Transformation in runner_orders table- Extract only numeric part from duration column
```sql
UPDATE runner_orders 
SET duration = SUBSTRING(duration, 1, 2);

```
#### Data Transformation in runner_orders table- Modifying data types
```sql
ALTER TABLE runner_orders
MODIFY pickup_time TIMESTAMP,
MODIFY distance DECIMAL(5, 2),
MODIFY duration INT;

```
### Questions & Solutions
---
### A. Pizza Metrics
#### Question 1. How many pizzas were ordered?

#### Solution:

```sql
SELECT
  COUNT(order_id) AS total_pizzas_ordered
FROM 
  customer_orders;
```

#### Output:
![a1](https://github.com/user-attachments/assets/91b306e9-71f6-4ad0-9468-7690c5021c8a)


- Total pizzas ordered = 14.

#### Question 2. How many unique customer orders were made?

#### Solution:

```sql
SELECT
  COUNT(DISTINCT order_id) AS unique_customer_orders
FROM 
  customer_orders;
```

#### Output:
![a2](https://github.com/user-attachments/assets/23745d69-61db-43be-af06-fc39e4e485c5)


- Number of unique customer orders placed = 10.

#### Question 3. How many successful orders were delivered by each runner?

#### Solution:

```sql
SELECT
  runner_id,
  COUNT(order_id) AS orders_delivered
FROM
  runner_orders
WHERE 
  cancellation IS NULL
GROUP BY 
  runner_id;
```

#### Output:
![a3](https://github.com/user-attachments/assets/db22b5e0-fb14-4255-874b-237e82f53665)


- Runner with runner_id 1 successfully delivered 4 orders.
- Runner with runner_id 2 successfully delivered 3 orders.
- Runner with runner_id 3 successfully delivered 1 order.

#### Question 4. How many of each type of pizza was delivered?

#### Solution:

```sql
SELECT
  pizza_name,
  COUNT(customer.order_id) AS total_pizzas_delivered
FROM
  customer_orders customer
JOIN
  runner_orders runner ON customer.order_id = runner.order_id
JOIN
  pizza_names pizza ON customer.pizza_id = pizza.pizza_id
WHERE
  runner.cancellation IS NULL
GROUP BY
  pizza_name;
```

#### Output:
![a4](https://github.com/user-attachments/assets/3980f9c6-4ff0-4332-a6c7-5dc4bc09534a)


- Total number of Meatlovers pizzas delivered = 9.
- Total number of Vegetarian pizzas delivered = 3.

#### Question 5. How many Vegetarian and Meatlovers were ordered by each customer?

#### Solution:

```sql
SELECT
  customer_id, 
  SUM(CASE WHEN pizza_id = 1 THEN 1 ELSE 0 END) AS Meatlovers_pizza, 
  SUM(CASE WHEN pizza_id = 2 THEN 1 ELSE 0 END) AS Vegetarian_pizza
FROM
  customer_orders
GROUP BY
  customer_id
ORDER BY
  customer_id;
```

#### Output:
![a5](https://github.com/user-attachments/assets/547f49c5-1a53-411d-b18b-273eff0ed368)


- Customer 101 ordered 2 Meatlovers pizzas and 1 Vegetarian pizza.
- Customer 102 ordered 2 Meatlovers pizzas and 1 Vegetarian pizza.
- Customer 103 ordered 3 Meatlovers pizzas and 1 Vegetarian pizza.
- Customer 104 ordered 3 Meatlovers pizzas and 0 Vegetarian pizza.
- Customer 105 ordered 0 Meatlovers pizza and 1 Vegetarian pizza.

#### Question 6. What was the maximum number of pizzas delivered in a single order?

#### Solution:

```sql
WITH pizzas_delivered AS
   (SELECT customer.order_id, COUNT(customer.pizza_id) AS num_of_pizzas
    FROM customer_orders AS customer 
    JOIN runner_orders AS runner ON customer.order_id = runner.order_id
    WHERE runner.cancellation IS NULL
    GROUP BY customer.order_id)

SELECT
  MAX(num_of_pizzas) AS max_pizzas_in_single_order
FROM
  pizzas_delivered;
```

#### Output:
![a6](https://github.com/user-attachments/assets/3105545d-f873-4069-ac95-64f3e4982d18)

- Maximum number of pizzas ordered in single order were 3.

#### Question 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

#### Solution:

```sql
SELECT
    co.customer_id,
    SUM(CASE WHEN (co.exclusions IS NOT NULL) OR (co.extras IS NOT NULL) THEN 1 ELSE 0 END) AS pizzas_with_change,
    SUM(CASE WHEN (co.exclusions IS NULL) AND (co.extras IS NULL) THEN 1 ELSE 0 END) AS pizzas_without_change   
FROM
    customer_orders co 
JOIN
    runner_orders ro ON co.order_id = ro.order_id
WHERE
    ro.cancellation IS NULL
GROUP BY
    co.customer_id;
```

#### Output:
![a7](https://github.com/user-attachments/assets/d58f9048-c089-4c39-a860-10819379a28b)


- Customer with id 101 delivered 0 pizzas with change and 2 pizzas without change.
- Customer with id 102 delivered 0 pizzas with change and 3 pizzas without change.
- Customer with id 103 delivered 3 pizzas with change and 0 pizzas without change.
- Customer with id 104 delivered 2 pizzas with change and 1 pizzas without change.
- Customer with id 105 delivered 1 pizzas with change and 0 pizzas without change.


#### Question 8. How many pizzas were delivered that had both exclusions and extras?

#### Solution:

```sql
SELECT
    count(c.order_id) AS pizzas_with_exclusions_extras
FROM
    customer_orders c 
JOIN
    runner_orders r ON c.order_id = r.order_id
WHERE
    r.cancellation IS NULL
    AND c.exclusions IS NOT NULL
    AND c.extras IS NOT NULL;
```

#### Output:
![a8](https://github.com/user-attachments/assets/b2cae53a-b95c-4aac-9936-a8ca7e0dd3e9)

- Only 1 pizza was delivered that had both exclusions and extras.

#### Question 9. What was the total volume of pizzas ordered for each hour of the day?

#### Solution:

```sql
SELECT
    HOUR(order_time) AS hour_of_day, COUNT(order_id) AS num_pizzas_ordered
FROM
    customer_orders
GROUP BY
    HOUR(order_time)
ORDER BY
    hour_of_day;
```

#### Output:
![a9](https://github.com/user-attachments/assets/555d2e6a-8a69-4e93-8863-fc4fe7bb5300)


- At hour 11, only 1 pizza was delivered.
- At hour 13, 3 pizzas were delivered.
- At hour 18, 3 pizzas were delivered.
- At hour 19, only 1 pizza was delivered.
- At hour 21, 3 pizzas were delivered.
- At hour 23, 3 pizzas were delivered.

#### Question 10. What was the volume of orders for each day of the week?

#### Solution:

```sql
SELECT
    DAYNAME(order_time) AS `day`,
    COUNT(order_id) AS num_pizzas_ordered
FROM
    customer_orders
GROUP BY
    DAYNAME(order_time),
    WEEKDAY(order_time)
ORDER BY
    WEEKDAY(order_time);
```

#### Output:
![a10](https://github.com/user-attachments/assets/e2c2b663-dda0-40d8-b1ea-b3adccf957f1)


- On Wednesday, 5 pizzas were delivered.
- On Thursday, 3 pizzas were delivered.
- On Friday, 1 pizza was delivered.
- On Saturday, 5 pizzas were delivered.

### B. Runner and Customer Experience
--- 

#### Question 1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

#### Solution:

```sql
SELECT
    DATE_ADD('2021-01-01', INTERVAL FLOOR(DATEDIFF(registration_date, '2021-01-01') / 7) week) AS week_starting,
    COUNT(runner_id) AS num_runners_signed_up
FROM
    runners
GROUP BY
    week_starting
ORDER BY
    week_starting;
```

#### Output:
![b1](https://github.com/user-attachments/assets/374f4406-a813-4615-bc0b-968228bfe6a8)


- 2 runners signed up for the week starting '2021-01-01'.
- 1 runner signed up for the week starting '2021-01-08'.
- 1 runner signed up for the week starting '2021-01-15'.

#### Question 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

#### Solution:

```sql
SELECT
    ro.runner_id,
    ROUND(AVG(TIMESTAMPDIFF(MINUTE, co.order_time, ro.pickup_time)),2) AS avg_pickup_time_minutes
FROM
    runner_orders ro
JOIN
    customer_orders co ON ro.order_id = co.order_id
WHERE
    ro.pickup_time IS NOT NULL
GROUP BY
    ro.runner_id;
```

#### Output:
![b2](https://github.com/user-attachments/assets/9ff1bda3-287e-4c7f-bacc-b8b4f7589e8d)

- Runner with id 1 took 15.33 minutes to reach pizza HQ to pickup the order.
- Runner with id 2 took 23.40 minutes to reach pizza HQ to pickup the order.
- Runner with id 3 took 10 minutes to reach pizza HQ to pickup the order.

#### Question 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?

#### Solution:

```sql
WITH order_preparation AS
    (SELECT
        c.order_id,
        COUNT(c.pizza_id) AS num_of_pizzas,
        AVG(TIMESTAMPDIFF(MINUTE, c.order_time, r.pickup_time)) AS avg_order_prep_time
    FROM
        customer_orders c 
    JOIN
        runner_orders r ON c.order_id = r.order_id
    WHERE
        r.pickup_time IS NOT NULL
    GROUP BY
        c.order_id)

SELECT
    num_of_pizzas,
    ROUND(AVG(avg_order_prep_time)) AS avg_preparation_time_minutes
FROM
    order_preparation
GROUP BY
    num_of_pizzas;
```

#### Output:
![b3](https://github.com/user-attachments/assets/87347f8c-c071-4a8d-841c-7ab4192cd782)


- On average, it took 12 minutes to prepare 1 pizza.
- On average, it took 18 minutes to prepare 2 pizzas.
- On average, it took 29 minutes to prepare 3 pizzas.

#### Question 4. What was the average distance travelled for each customer?

#### Solution:

```sql
SELECT
    co.customer_id,
    ROUND(AVG(ro.distance)) AS avg_distance_travelled
FROM
    customer_orders co 
JOIN
    runner_orders ro ON co.order_id = ro.order_id 
WHERE
    ro.distance IS NOT NULL
GROUP BY
    co.customer_id;

```

#### Output:
![b4](https://github.com/user-attachments/assets/a1f0db1b-e7d7-4845-844b-38d8efdcf16a)


- Average distance travelled by customer with id 101 is 20 km.
- Average distance travelled by customer with id 102 is 16 km.
- Average distance travelled by customer with id 103 is 23 km.
- Average distance travelled by customer with id 104 is 10 km.
- Average distance travelled by customer with id 105 is 25 km.

#### Question 5. What was the difference between the longest and shortest delivery times for all orders?

#### Solution:

```sql
SELECT
    MAX(duration) - MIN(duration) AS delivery_time_difference
FROM
    runner_orders 
WHERE
    cancellation IS NULL;

```

#### Output:
![b5](https://github.com/user-attachments/assets/d12d93f7-e6f0-41fd-b4d7-838159aa3e8c)

- The difference between the longest and shortest delivery times for all orders is 30 minutes.

#### Question 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?

#### Solution:

```sql
SELECT
    runner_id,
    order_id,
    ROUND(AVG(distance / duration * 60),2) AS avg_speed_km_hr
FROM
    runner_orders
WHERE
    distance IS NOT NULL
GROUP BY
    runner_id, order_id
ORDER BY
    runner_id;

```

#### Output:
![b6](https://github.com/user-attachments/assets/1dcd68e8-9d5e-43ce-8391-108e66c89ad6)


#### Question 7. What is the successful delivery percentage for each runner?

#### Solution:

```sql
SELECT
    runner_id,
    ROUND(SUM(CASE WHEN cancellation IS NULL THEN 1 ELSE 0 END) / COUNT(*) * 100) AS successful_delivery_percentage
FROM
    runner_orders 
GROUP BY
    runner_id;

```

#### Output:
![b7](https://github.com/user-attachments/assets/897737eb-922a-4d81-a7fc-dbaf8e857487)

- Successful delivery percentage for runner with id 1 is 100%.
- Successful delivery percentage for runner with id 2 is 75%.
- Successful delivery percentage for runner with id 3 is 50%.


