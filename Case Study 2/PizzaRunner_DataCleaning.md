# Data Cleaning 

### Customer Order Table

Looking at the customer order table, we can see that the column 'exclusions' and 'extras' has some discrepancies. 

- In *exclusions* column, there are some missing values and blank spaces.
- In *extras* column, there are some ' ', null and missing spaces. 

We will replace all the 'null', missing values with ' '.


##### Cleaning our table 

```sql
CREATE TEMP TABLE customer_orders_new AS
SELECT 
  order_id, 
  customer_id, 
  pizza_id, 
  CASE
	  WHEN exclusions IS null OR exclusions LIKE 'null' THEN ' '
	  ELSE exclusions
	  END AS exclusions,
  CASE
	  WHEN extras IS NULL or extras LIKE 'null' THEN ' '
	  ELSE extras
	  END AS extras,
	order_time
FROM pizza_runner.customer_orders;
```

Looking at the cleaned table. 

![cleaned table](https://picc.io/Uo2NqTf.PNG)

### Runners TABLE

Looking at the runner_orders table below, we can see that there are


- Replacing null with blank space in multiple columns such as *pickup_time* and *cancellation*.
- Removing text such *km*, or *minutes* from the numeric values in columns, *distance*, and *duration*.

##### Cleaning our table 


```sql 

CREATE TEMP TABLE runner_orders_new AS
SELECT order_id, runner_id, 
	CASE WHEN pickup_time LIKE 'null' THEN ' ' ELSE pickup_time END AS pickup_time,
	CASE WHEN cancellation IS NULL OR cancellation LIKE 'null' THEN ' ' ELSE cancellation END AS cancellation,
	CASE WHEN distance LIKE 'null' THEN ' ' 
	WHEN distance LIKE '%km' THEN TRIM('km' FROM distance) ELSE distance END AS distance,
	CASE WHEN duration LIKE '%mins' THEN TRIM('mins' FROM duration
		WHEN duration LIKE 'null' THEN ' '
		WHEN duration LIKE '%minute' THEN TRIM('minute' FROM duration
		WHEN duration LIKE '%minutes' THEN TRIM('minutes' FROM duration
		ELSE duration
		END AS duration
FROM pizza_runner.runner_orders;
```

Looking at the cleaned runners table.

![cleaned table](https://picc.io/9SCdmlQ.PNG)




