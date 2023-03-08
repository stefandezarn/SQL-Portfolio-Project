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

