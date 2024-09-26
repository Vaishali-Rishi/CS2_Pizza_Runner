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
