# Understanding-customer-orders-at-Target

Target – Case Study using SQL <br>

Target is one of the world’s most recognized brands and one of America’s leading 
retailers. Target makes itself a preferred shopping destination by offering 
outstanding value, inspiration, innovation and an exceptional guest experience 
that no other retailer can deliver.
This business case has information of 100k orders from 2016 to 2018 made at 
Target in Brazil. Its features allows viewing an order from multiple dimensions: 
from order status, price, payment and freight performance to customer location, 
product attributes and finally reviews written by customers.

SELECT distinct customer_city, customer_state FROM `my-project-target-382813.Target.customers` ;

select extract(year from min(order_purchase_timestamp)) as min_year, 
extract(year from max(order_purchase_timestamp)) as max_year from `my-project-target-382813.Target.orders` ; 
SELECT * FROM `my-project-target-382813.Target.orders` ;

select distinct customer_state from `my-project-target-382813.Target.customers` ; 

SELECT  extract(year from order_purchase_timestamp) as year,
extract(month from order_purchase_timestamp) as month , count(order_id) as No_of_orders  FROM `my-project-target-382813.Target.orders` group by year, month order by  year,month;

select u.* from (select  t.*, avg(No_of_orders)over(partition by year) as avg_orders
 from (SELECT  extract(year from order_purchase_timestamp) as year,
extract(month from order_purchase_timestamp) as month , count(order_id) as No_of_orders,
  FROM `my-project-target-382813.Target.orders` group by year, month order by  year,month) t)u
  where No_of_orders > avg_orders order by year, month ;


select distinct Time_tend_to_buy,sum(No_of_orders)over(partition by Time_tend_to_buy) as sum_of_orders
 from (with c as (SELECT  extract(hour from order_purchase_timestamp) as hour,
count(order_id) as No_of_orders  FROM `my-project-target-382813.Target.orders` 
group by hour order by  hour)
select *, case when hour >= 0 and hour < 7 then 'dawn'
                          when hour >= 7 and hour < 11 then 'morning'
                          when hour >= 11 and hour < 18 then 'afternoon'
                          when hour >=18 and hour <= 23 then 'night'
                          end as Time_tend_to_buy
from c) u;

SELECT  extract(month from order_purchase_timestamp) as month , B.customer_state, count(A.order_id) as No_of_orders  FROM `my-project-target-382813.Target.orders` A
join `my-project-target-382813.Target.customers` B on A.customer_id = B.customer_id
group by B.customer_state, month order by  month, B.customer_state;

select customer_state, count(customer_id) as no_of_customers from `my-project-target-382813.Target.customers` group by customer_state order by no_of_customers;

select order_item_id, extract(year from shipping_limit_date) as year,  avg(price) as avg_price, avg(freight_value) as avg_freight_val from `Target.order_items` group by year, order_item_id order by order_item_id, year;

with c as(select year, sum(payment_value) as cost_of_orders  from (select extract (year from shipping_limit_date) as year,extract (month from shipping_limit_date) as month, p.payment_value from `Target.payments` p join `Target.order_items` o on o.order_id = p.order_id) t where (month >=1 and month<= 8) and year<=2018 group by year order by year)
select round(((cost_of_orders - lag(cost_of_orders)over(order by year asc ))/lag(cost_of_orders)over(order by year asc )) * 100) as percentage_increase from c  ;

select distinct customer_state, round(avg(price)over(partition by customer_state),2) as avg_price, round(sum(price)over(partition by customer_state),2) as sum_price, round(avg(freight_value)over(partition by customer_state),2) as avg_freight_value, round(sum(freight_value)over(partition by customer_state),2) as sum_freight_value from Target.customers c join Target.orders o on c.customer_id = o.customer_id join Target.order_items oi on o.order_id = oi.order_id order by customer_state;
 
select order_id, date_diff( order_estimated_delivery_date , order_purchase_timestamp, day) as estimated_days, date_diff(order_delivered_customer_date , order_purchase_timestamp, day) as delivered_days  from Target.orders order by order_id;

select order_id, date_diff( order_delivered_customer_date , order_purchase_timestamp, day) as time_to_delivery, date_diff(order_estimated_delivery_date , order_delivered_customer_date, day) as diff_estimated_delivery  from Target.orders order by order_id;

select distinct state, mean_freight_val, round(avg(time_to_delivery)over(partition by state),2) as mean_time_to_delivery, round(avg(diff_estimated_delivery)over(partition by state),2) as mean_diff_estimated_delivery from
 (select customer_state as state, round(avg(freight_value)over(partition by customer_state),2) as mean_freight_val, 
date_diff( order_delivered_customer_date , order_purchase_timestamp, day) as time_to_delivery, date_diff(order_estimated_delivery_date , order_delivered_customer_date, day) as diff_estimated_delivery
  from Target.customers c join Target.orders o on c.customer_id = o.customer_id join Target.order_items oi on o.order_id = oi.order_id order by customer_state)t order by state;
  
select distinct state, mean_freight_val from
 (select customer_state as state, round(avg(freight_value)over(partition by customer_state),2) as mean_freight_val
  from Target.customers c join Target.orders o on c.customer_id = o.customer_id join Target.order_items oi on o.order_id = oi.order_id order by customer_state)t order by mean_freight_val desc limit 5;

select distinct state, round(avg(time_to_delivery)over(partition by state),2) as mean_time_to_delivery from
 (select customer_state as state, date_diff( order_delivered_customer_date , order_purchase_timestamp, day) as time_to_delivery
  from Target.customers c join Target.orders o on c.customer_id = o.customer_id join Target.order_items oi on o.order_id = oi.order_id order by customer_state)t order by mean_time_to_delivery asc limit 5;

 select state , mean_time_to_delivery, mean_diff_estimated_delivery, case when (mean_time_to_delivery < mean_diff_estimated_delivery) or (mean_time_to_delivery - mean_diff_estimated_delivery)<1 then "fast_delivery" else "not_so_fast" end as delivery
  from (select distinct state, round(avg(time_to_delivery)over(partition by state),2) as mean_time_to_delivery, round(avg(diff_estimated_delivery)over(partition by state),2) as mean_diff_estimated_delivery from
 (select customer_state as state, 
date_diff( order_delivered_customer_date , order_purchase_timestamp, day) as time_to_delivery, date_diff(order_estimated_delivery_date , order_delivered_customer_date, day) as diff_estimated_delivery
  from Target.customers c join Target.orders o on c.customer_id = o.customer_id  order by customer_state)) order by delivery;

  select  extract(month from(order_purchase_timestamp)) as month, count(o.order_id) as no_of_orders, payment_type from Target.orders o join Target.payments p on o.order_id = p.order_id group by month,payment_type order by month;

  select  count(o.order_id) as no_of_orders, payment_installments from Target.orders o join Target.payments p on o.order_id = p.order_id group by payment_installments order by payment_installments;
