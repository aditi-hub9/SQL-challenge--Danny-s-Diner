# SQL-challenge--Danny's-Diner

Case Study #1 - Danny's Diner
![image](https://github.com/aditi-hub9/SQL-challenge--Danny-s-Diner/assets/114562936/1cffd16b-f527-495a-a3aa-a658bd3eecfc)


Introduction

Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: 
sushi, curry and ramen.
Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.

Problem Statement

Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.
He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.
Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!
Danny has shared with you 3 key datasets for this case study:
•	sales
•	menu
•	members
You can inspect the entity relationship diagram and example data below.
![image](https://github.com/aditi-hub9/SQL-challenge--Danny-s-Diner/assets/114562936/64da3a32-a32f-4b49-8d72-62a0be4325d6)

 
Example Datasets
All datasets exist within the dannys_diner database schema - be sure to include this reference within your SQL scripts as you start exploring the data and answering the case study questions.

Table 1: sales
The sales table captures all customer_id level purchases with a corresponding order_date and product_id information for when and what menu items were ordered.
 ![image](https://github.com/aditi-hub9/SQL-challenge--Danny-s-Diner/assets/114562936/0ef79ef1-7ae4-4819-9fdf-01b398f5d158)

Table 2: menu
The menu table maps the product_id to the actual product_name and price of each menu item.
 ![image](https://github.com/aditi-hub9/SQL-challenge--Danny-s-Diner/assets/114562936/b745ed74-e8b6-4729-af57-6fbf9891f731)

Table 3: members
The final members table captures the join_date when a customer_id joined the beta version of the Danny’s Diner loyalty program.
 ![image](https://github.com/aditi-hub9/SQL-challenge--Danny-s-Diner/assets/114562936/ebe811a7-1b51-48bd-9039-9c79b34c88f3)

One can find the link to source here 
https://8weeksqlchallenge.com/case-study-1/
  
Case Study Questions: 
1.	What is the total amount each customer spent at the restaurant?
   
select customer_id, sum(price)
from menu
inner join sales
on menu.product_id= sales.product_id
group by customer_id;


2.	How many days has each customer visited the restaurant?

select* , count(distinct(order_date)) as days_visited from sales
group by customer_id;


3.	What was the first item from the menu purchased by each customer?

with cte_temp as ( select *, row_number() over (partition by customer_id order by order_date) as first_order
from sales
inner join menu
using (product_id))

select customer_id,
    product_name,
    first_order from cte_temp
    where first_order = 1;
	
4.	What is the most purchased item on the menu and how many times was it purchased by all customers?

select product_name , count(product_name) as items_purchased from sales
inner join menu
on sales.product_id = menu.product_id
group by product_name
ORDER BY
	items_purchased DESC;
    
5.	Which item was the most popular for each customer?

select product_name, customer_id, count(product_id) as popular_dish ,
dense_rank() over(partition by customer_id order by count(product_id) desc) as a from sales 
inner join menu
using (product_id)
group by customer_id, product_name) -- can use reference also like group by 1,2

SELECT customer_id, product_name
FROM cte_table
WHERE a = 1; -- filter the data, where rank is 1

6.	Which item was purchased first by the customer after they became a member?

WITH new_table AS
(SELECT
	*, DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY order_date) AS rnk
FROM sales
INNER JOIN menu USING(product_id)
INNER JOIN members USING(customer_id)
WHERE
	order_Date >= join_date)
SELECT
	*FROM
	new_table
WHERE
	rnk = 1;
 
7.	Which item was purchased just before the customer became a member?

WITH new_table AS
(SELECT
	*, DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY order_date) AS rnk
FROM sales
INNER JOIN menu USING(product_id)
INNER JOIN members USING(customer_id)
WHERE
	order_Date < join_date)
SELECT
	*FROM
	new_table
WHERE
	rnk = 1;
 
8.	What is the total items and amount spent for each member before they became a member?

select customer_id, product_name
,  count(product_id) as total_items ,sum(price) as total_price from sales 
inner join menu
using (product_id)
inner join members
using (customer_id)
where order_date < join_date
group by customer_id;

9.	If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

select customer_id , sum(points)as points from  (
select * , 
case 
when product_name ="sushi" then price*20
else price*10
end as points
 from sales
inner join menu
using(product_id)) as t
group by customer_id;

10.	In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi
- how many points do customer A and B have at the end of January?

with cte_dates as (
	select*,
date_add(join_date, interval 6 day) as valid_date, -- creating range
last_day("2021-01-31") as last_date-- end of jan 
from members)
select customer_id, sum(
case when product_name = "sushi" then price*20 -- sushi*20
 when order_date between join_date and valid_date-- in 1st wk
 then price*20
 else price*10
 end) as points
 from cte_dates
inner join sales
using(customer_id)
inner join menu using(product_id)
where order_date< last_date
group by customer_id;  




























