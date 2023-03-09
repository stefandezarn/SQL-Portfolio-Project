# Business Questions and SQL Queries
It is no mystery why SQL is one of the most utilized technologies in Data Analysis and Data Science today. The logical architecture and powerful simplicity of the querying language make it an invaluable tool in handling and investigating large amounts of relational data.

In this project I use Azure Data Studio and SQL to investigate and answer business questions about a fictional pizzeria. The dataset is comprised of four tables: orders.csv, order_details.csv, pizzas.csv, pizza_types.csv.

[Link to the dataset](https://www.kaggle.com/datasets/ylenialongo/pizza-sales)

After importing the CSV files, I created the database with the following schema:

<img width="632" alt="Screenshot 2023-03-08 at 2 31 57 PM" src="https://user-images.githubusercontent.com/116113763/223855177-8a4f3cdf-b425-4f67-a88d-955519654a3f.png">

Once the database was built, I could start exploring the data and answering business questions via querying. I used this opportunity to imagine what information a stakeholder might request. As the data analyst for this fictional pizzeria, I came up with ad-hoc query code to provide this information.

## Let's look at the distribution of total order cost. How many orders are in our database? What is the mean, median, minimum and maximum order total?
```
--Total number of orders

SELECT
    COUNT(DISTINCT  order_id) as total_orders
FROM
    orders
```
<img width="175" alt="Screenshot 2023-03-08 at 2 55 46 PM" src="https://user-images.githubusercontent.com/116113763/223860753-ac7dd64d-113a-419d-9794-22e7eb9e65b2.png">

 
```
--Mean, median, min and max of order totals

with order_totals as (
SELECT 
    od.order_id,
    SUM(od.quantity  * p.price) as order_total
FROM
    order_details od join pizzas p on od.pizza_id = p.pizza_id
GROUP BY 
    od.order_id
)

SELECT
    AVG(order_total) as mean_order_total,
    MIN(order_total) as minimum_order_total,
    MAX(order_total) as maximum_order_total
FROM
    order_totals
```
<img width="568" alt="Screenshot 2023-03-08 at 2 56 17 PM" src="https://user-images.githubusercontent.com/116113763/223891490-435cedc1-57e4-46df-91d2-693191e70ef0.png">

```
-- We need to order ingredients, list every ingredient and how many pizzas were ordered this year with that indgredient

WITH m1 as (
SELECT
    *
FROM
    order_details od 
    JOIN pizzas p on od.pizza_id = p.pizza_id 
    JOIN pizza_types pt on p.pizza_type_id = pt.pizza_type_id
)

, M2 as (
    SELECT
    i
)

SELECT ingredients
FROM pizza_types
```

```
-- What is the popularity ranking of the pizza types?

SELECT
    pt.name,
    SUM(od.quantity) as total_pizzas,
    RANK() OVER( ORDER BY SUM(od.quantity) DESC) as rank
FROM
    order_details od 
    JOIN pizzas p on od.pizza_id = p.pizza_id
    JOIN pizza_types pt on p.pizza_type_id = pt.pizza_type_id
GROUP BY
    pt.name
ORDER BY
    SUM(od.quantity) DESC
```
<img width="467" alt="Screenshot 2023-03-08 at 3 04 52 PM" src="https://user-images.githubusercontent.com/116113763/223891560-b68b278e-695b-440c-9756-65f9237f4b43.png">

```
-- Chicken Shortage

SELECT
    COUNT(DISTINCT od.order_id) num_of_orders
        
FROM
    order_details od 
    JOIN pizzas p on od.pizza_id = p.pizza_id
    JOIN pizza_types pt on p.pizza_type_id = pt.pizza_type_id
WHERE
    ingredients LIKE '%Chicken%'
```
<img width="190" alt="Screenshot 2023-03-08 at 3 06 21 PM" src="https://user-images.githubusercontent.com/116113763/223891653-8c9ba0ca-697a-47e2-a05f-a07450290d58.png">

```
-- What is the monthly distribution of orders ?
SELECT
    FORMAT(date, 'yyyy-MM') month,
    COUNT(DISTINCT order_id) num_of_orders
FROM
    orders
GROUP BY
    FORMAT(date, 'yyyy-MM')
ORDER BY 1 
```
<img width="265" alt="Screenshot 2023-03-08 at 3 07 47 PM" src="https://user-images.githubusercontent.com/116113763/223891704-7a793476-1a93-4875-9f61-cadc5459105f.png">

```
-- What are the most popular pizza catergories? What are the associated revenues?

SELECT
    pt.category,
    SUM(od.quantity) total_pizzas,
    RANK() OVER(ORDER BY SUM(od.quantity) DESC) quantity_rank,
    CONCAT('$',FORMAT(SUM(od.quantity * p.price),'N')) revenue,
    RANK() OVER(ORDER BY SUM(od.quantity * p.price) DESC) revenue_rank
FROM
    order_details od 
    JOIN pizzas p ON od.pizza_id = p.pizza_id
    JOIN pizza_types pt ON p.pizza_type_id = pt.pizza_type_id
GROUP BY
    pt.category
ORDER BY
    SUM(od.quantity) DESC
```

```
-- When 10 or more pizzas are ordered together, special delivery accomodations must be made. What percentage of the orders required these accomdations?

with m1 as (
SELECT
    order_id
FROM
    order_details
GROUP BY
    order_id
HAVING SUM(quantity) > 10
)
SELECT
    (CAST(COUNT(DISTINCT order_id) AS FLOAT)/(SELECT COUNT(DISTINCT order_id) FROM order_details)) *100 percent_large_orders
FROM
    m1
```

```
-- In order to prepare for these large orders, we need to know when they occur. What is the distribution of order times for these large orders?

with m1 as (
SELECT
    order_id
FROM
    order_details
GROUP BY
    order_id
HAVING SUM(quantity) > 10
)

SELECT
    DATEPART(hour,time) hour,
    COUNT(*)
FROM
    m1 join orders o on m1.order_id = o.order_id
GROUP BY
    DATEPART(hour,time)
ORDER BY 1
```

```
-- What hours are the most popular?

SELECT
    DATEPART(hour,time) hour,
    COUNT(*) num_of_orders
FROM
    orders
GROUP BY
    DATEPART(hour,time)
ORDER BY
    2 DESC
```

```
-- Show monthly revenue and percentage change in revenue month over month.

WITH t1 as (
SELECT
    FORMAT(o.date, 'yyyy-MM') month
    , ROUND(SUM(od.quantity * p.price),2) revenue
FROM
    orders o 
    join order_details od on o.order_id = od.order_id
    join pizzas p on od.pizza_id = p.pizza_id
GROUP BY FORMAT(o.date, 'yyyy-MM')
)

SELECT
    month
    , revenue
    , ROUND(((revenue - LAG(revenue, 1) OVER(ORDER BY month))/LAG(revenue, 1) OVER(ORDER BY month))*100, 2) pct_chng_in_rev
FROM t1
```
