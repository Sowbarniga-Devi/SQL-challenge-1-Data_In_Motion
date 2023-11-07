##SQL Case Study 1: Tiny Shop Sales – Data in Motion (d-i-motion.com)

###Sales analysis of a tiny store in PostgreSQL

1) Which product has the highest price? Only return a single row.

	#SELECT *
	#FROM products
	#ORDER BY price DESC
	#LIMIT 1
--------------------------------------------------------------------------------------------------
2) Which customer has made the most orders?

**With order_total as
(
SELECT customer_id,count(order_id) total_orders,
	rank() OVER (ORDER BY count(order_id) desc) 
FROM orders
GROUP BY 1
ORDER BY 2 DESC
)
SELECT A.customer_id,A.first_name,A.last_name,B.total_orders
FROM customers A**
JOIN order_total B
ON A.customer_id = B.customer_id
WHERE rank =1
ORDER BY 1
--------------------------------------------------------------------------------------------------
3) What’s the total revenue per product?
 
SELECT 
		A.product_id
		,A.product_name
		,sum(A.price*B.quantity) total_revenue 
FROM products A
INNER JOIN order_items B
ON A.product_id =B.product_id
GROUP BY 1,2
ORDER BY 1
--------------------------------------------------------------------------------------------------
4) Find the day with the highest revenue.
 
SELECT 
		CAST(A.order_date AS varchar)
		,SUM(B.quantity*C.price) revenue_per_day
FROM orders A
JOIN order_items B	
ON A.order_id = B.order_id
JOIN products C
ON C.product_id = B.product_id
GROUP BY 1
ORDER BY 2 DESC
LIMIT 1
--------------------------------------------------------------------------------------------------
5.Find the first order (by date) for each customer.
 
WITH first_order as
(
  SELECT 
		customer_id,
		CAST(order_date AS VARCHAR) order_date,
		RANK() OVER(partition by customer_id order by order_date) 
FROM orders
ORDER BY 1
 ) 
 SELECT first_order.customer_id,first_order.order_date first_order
 FROM first_order
 WHERE first_order.rank = 1
 --------------------------------------------------------------------------------------------------
 6) Find the top 3 customers who have ordered the most distinct products

SELECT 
		customer_id
		,count(distinct product_id) distint_prod_count
FROM order_items A
JOIN orders B
ON A.order_id=B.order_id
GROUP BY B.customer_id
HAVING count(distinct product_id) = 3
ORDER BY 1 DESC
--------------------------------------------------------------------------------------------------
7) Which product has been bought the least in terms of quantity?

SELECT 
		o.product_id
		,p.product_name
FROM order_items o
JOIN products p
ON o.product_id = p.product_id
GROUP BY 1,2
HAVING count(o.quantity) =2 (-- least ordered count was 2(SELECT product_id,count(quantity) FROM order_items group by 1 order by 2))
ORDER BY 1
--------------------------------------------------------------------------------------------------
8)What is the median order total?

SELECT 
		ROUND(AVG(O.quantity * P.price),2) median_order_total
FROM order_items O
JOIN products P
ON O.product_id = P.product_id
----------------------------------------------------------------------------------------------------
9) For each order, determine if it was ‘Expensive’ (total over 300), ‘Affordable’ (total over 100), or ‘Cheap’.

SELECT 
		O.order_id,sum(O.quantity*P.price) total_price,
		CASE when sum(O.quantity*P.price)>=300 THEN 'Expensive'
        		WHEN sum(O.quantity*P.price) >100 THEN 'Affordable'
        ELSE 'Cheap'
        END as Price_range
FROM order_items O
JOIN products P
ON O.order_id = P.product_id
GROUP BY 1
ORDER BY 1
----------------------------------------------------------------------------------------------------
10) Find customers who have ordered the product with the highest price.

SELECT 
		OO.customer_id
		 ,SUM(O.quantity * P.price) highest_price
FROM ORDER_items O
JOIN products P
ON O.product_id = P.product_id
JOIN orders OO
ON OO.order_id = O.order_id
GROUP BY 1
ORDER BY 2 DESC
LIMIT 1




