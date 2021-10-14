
![Alt text](https://picc.io/aNqgKIb.PNG)

# Introduction 

This case study is the part of the 8 week SQL challenge from Danny Ma in his Serious SQL course. 

## Context 

Danny is a huge fan of Japanese cuisine, so in the spring of 2021, he take a chance and starts a lovely small restaurant that serves his 
three favourite dishes: sushi, curry, and ramen. 

## Problem Statement

Danny wants to utilise the information to answer a few simple questions about his clients, such as their visit patterns, the amount of 
money they've spent, and their favourite menu items. Having a closer relationship with his consumers would enable him to provide a better, 
more personalised experience for his loyal customers. 

He intends to utilise these findings to determine whether or not to expand the existing customer loyalty programme; he also requires 
assistance in generating some simple datasets so that his team can quickly analyse the data without having to use SQL.

## Dataset

There are 3 key datasets for this case study.
* Sales
* Menu
* Members

##### Below is an image of Entity Relationship Diagram

![image](C:\Users\vshaa\Desktop\Serious SQL)

We will start looking at the tables now to get an idea about the data. 

#### Table-1 Sales

```sql 
SELECT * FROM dannys_diner.sales
LIMIT 5;
```

![Alt text](https://picc.io/uqgm1q9.PNG)

#### Table-2 Menu

```sql
SELECT * FROM dannys_diner.menu
LIMIT 5;
```
![Alt text](https://picc.io/4uF_nD4.PNG)

#### Table-3 Members

```sql
SELECT * FROM dannys_diner.members;
```

![Alt text](https://picc.io/-F0iuEB.PNG)

## Case Study Questions

1. What is the total amount each customer spent at the restaurant?

```sql 
SELECT customer_id, SUM(menu.price) AS amount
FROM dannys_diner.sales s 
INNER JOIN dannys_diner.menu menu ON s.product_id = menu.product_id
GROUP BY customer_id;
```

2. How many days has each customer visited the restaurant?

```sql
SELECT customer_id, COUNT(DISTINCT order_date) AS num_days
FROM dannys_diner.sales
GROUP BY customer_id;
```

3. What was the first item from the menu purchased by each customer?

```sql
WITH rank_products AS (
SELECT customer_id, product_name, RANK() OVER(PARTITION BY customer_id ORDER BY order_date) AS rank_number
FROM dannys_diner.sales s 
INNER JOIN dannys_diner.menu m ON s.product_id = m.product_id
)
SELECT * FROM rank_products
WHERE rank_number = 1;
```

4. What is the most purchased item on the menu and how many times was it purchased by all customers?

```sql
SELECT  product_name, COUNT(order_date) AS num_orders
FROM dannys_diner.sales s
INNER JOIN dannys_diner.menu m ON s.product_id = m.product_id
GROUP BY  product_name
ORDER BY COUNT(order_date) DESC
LIMIT 1; 
```

5. Which item was the most popular for each customer? 

```sql
SELECT customer_id, product_name, COUNT(order_date) AS num_orders
FROM dannys_diner.sales s 
INNER JOIN dannys_diner.menu m ON s.product_id = m.product_id
GROUP BY customer_id, product_name
ORDER BY customer_id, COUNT(order_date) DESC ;
```

6. Which item was purchased first by the customer after they became a member?

```sql
DROP TABLE IF EXISTS member_sales;
CREATE TEMP TABLE member_sales AS
SELECT sales.customer_id,
    sales.order_date,
    menu.product_name,
    members.join_date,
    RANK() OVER(PARTITION BY sales.customer_id ORDER BY sales.order_date) AS order_rank
FROM dannys_diner.sales sales 
INNER JOIN dannys_diner.menu menu ON sales.product_id = menu.product_id
INNER JOIN dannys_diner.members members ON sales.customer_id = members.customer_id
WHERE order_date >= join_date::DATE;


SELECT customer_id AS customer_ID, order_date, join_date, product_name AS item_name, order_rank
FROM member_sales;
```

7. Which item was purchased just before the customer became a member?

```sql
DROP TABLE IF EXISTS non_member_sales;
CREATE TEMP TABLE non_member_sales AS
SELECT sales.customer_id,
    sales.order_date,
    menu.product_name,
    members.join_date,
    RANK() OVER(PARTITION BY sales.customer_id ORDER BY sales.order_date DESC) AS order_rank
FROM dannys_diner.sales sales 
INNER JOIN dannys_diner.menu menu ON sales.product_id = menu.product_id
INNER JOIN dannys_diner.members members ON sales.customer_id = members.customer_id
WHERE order_date < join_date::DATE;


SELECT * FROM non_member_sales
WHERE order_rank = 1;
```

8. What is the total items and amount spent for each member before they became a member?

```sql
DROP TABLE IF EXISTS total_items_amount;
CREATE TEMP TABLE total_items_amount AS
SELECT sales.customer_id, sales.order_date, RANK() OVER(PARTITION BY sales.customer_id ORDER BY order_date DESC) AS order_rank,
       menu.price, sales.product_id
FROM dannys_diner.sales sales 
INNER JOIN dannys_diner.menu menu ON sales.product_id = menu.product_id
INNER JOIN dannys_diner.members members ON sales.customer_id = members.customer_id
WHERE order_date < join_date::DATE;

SELECT * FROM total_items_amount;

SELECT customer_id, COUNT(DISTINCT product_id) AS total_items, SUM(price) AS amount
FROM total_items_amount
GROUP BY customer_id;
```

9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```sql 
DROP TABLE IF EXISTS customer_points;
CREATE TEMP TABLE customer_points AS
SELECT sales.customer_id, menu.price, 
       CASE WHEN menu.product_name = 'sushi' THEN 20*price ELSE 10*price END AS points
FROM dannys_diner.sales sales
INNER JOIN dannys_diner.menu menu ON sales.product_id = menu.product_id;

SELECT customer_id, SUM(points) AS total_points
FROM customer_points
GROUP BY customer_id
ORDER BY SUM(points) DESC;
```

10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - 
how many points do customer A and B have at the end of January?

```sql
DROP TABLE IF EXISTS membership_points;
CREATE TEMP TABLE membership_points AS
SELECT sales.customer_id, menu.price, members.join_date, sales.order_date,
       (20*price) AS points
FROM dannys_diner.sales sales 
INNER JOIN dannys_diner.menu menu ON sales.product_id = menu.product_id
INNER JOIN dannys_diner.members members ON sales.customer_id = members.customer_id
WHERE order_date >= join_date AND order_date <= '2021-01-31';


SELECT * FROM membership_points;

SELECT customer_id, SUM(points) AS total_points
FROM membership_points
GROUP BY customer_id;
```

**Bonus Question**

11. Try to recreate the table given below. 

![Alt text](https://picc.io/YD4WeH4.PNG)

```sql

SELECT sales.customer_id, sales.order_date, menu.product_name, menu.price, 
       CASE WHEN join_date <= order_date THEN 'Y' ELSE 'N' END AS membership_status
FROM dannys_diner.sales sales 
LEFT JOIN dannys_diner.menu menu ON sales.product_id = menu.product_id
LEFT JOIN dannys_diner.members members ON sales.customer_id = members.customer_id
ORDER BY sales.customer_id, sales.order_date;
```




