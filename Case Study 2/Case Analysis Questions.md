# CASE STUDY 2 - Pizza Runners

### 1. How many pizzas were ordered? 

```sql
SELECT COUNT(*) AS pizza_order_count
FROM pizza_runner.customer_orders;
```

Solution:

![solution](https://picc.io/W-sX0lh.PNG)

### 2. How many unique customer orders were made?

```sql
SELECT COUNT(DISTINCT order_id) AS unique_orders
FROM customer_orders_new;
```

Solution:

![solution](https://picc.io/II2Lj_C.PNG)

### 3. How many successful orders were delivered by each runner?

```sql
SELECT 
  runner_id, 
  COUNT(order_id) AS successful_orders
FROM pizza_runner.runner_orders
WHERE distance != 0
GROUP BY runner_id;
```

Alternative Query:

```sql
SELECT runner_id, COUNT( DISTINCT order_id) AS total_orders_delivered
FROM runner_orders_new
WHERE cancellation NOT IN ('Restaurant Cancellation', 'Customer Cancellation') OR cancellation IS NULL
GROUP BY runner_id;
```

Solution:

![solution](https://picc.io/0sARETS.PNG)


### 4. How many of each type of pizza was delivered?

```sql
SELECT pizza_name, COUNT(orders.pizza_id) AS total_pizzas
FROM customer_orders_new orders 
JOIN pizza_runner.pizza_names names 
ON orders.pizza_id = names.pizza_id
JOIN runner_orders_new r_orders 
ON orders.order_id = r_orders.order_id
WHERE cancellation NOT IN ('Restaurant Cancellation', 'Customer Cancellation') OR cancellation IS NULL
GROUP BY pizza_name;
```

Solution:

![solution](https://picc.io/6nbFtKP.PNG)


### 5. How many Vegetarian and Meatlovers were ordered by each customer?
