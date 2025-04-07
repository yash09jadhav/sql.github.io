### Q1 For each customer, identify their highest order and tag it as 'Top Order', all others as 'Regular'.
SELECT *,
CASE
    WHEN RANK() OVER(PARTITION BY customer_id ORDER BY total_amount DESC) = 1 THEN 'top_order'
    ELSE 'regular'
END AS order_tag
FROM campusx.orders_sample;
 
### Q2 For every order, calculate the number of days since that customer’s previous order.
SELECT *, 
       DATEDIFF(order_date, LAG(order_date) OVER (PARTITION BY customer_id ORDER BY order_date)) AS days_since_last_order
FROM orders_sample;

### Q3 For each customer, calculate what percentage of their total spend each individual order represents.
SELECT *,SUM(total_amount) OVER(PARTITION BY customer_id) AS customer_total, 
total_amount*100/(SUM(total_amount) OVER(PARTITION BY customer_id)) AS percentage_of_total
FROM orders_sample;

### Q4 Find the top 3 customers who placed the most number of orders. In case of a tie, use total_amount sum as a tiebreaker.
SELECT *,
       RANK() OVER(ORDER BY orders DESC, total_spend DESC) AS 'rank'
FROM (
    SELECT customer_id,
           COUNT(order_id) AS orders,
           SUM(total_amount) AS total_spend
    FROM orders_sample
    GROUP BY customer_id
) t;

### Q5 For each order, calculate the average total_amount of the current and previous two orders for that customer (rolling 3-order average).
SELECT *,
AVG(total_amount) OVER(PARTITION BY customer_id ORDER BY order_date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS rolling_avg
FROM orders_sample;

### Q6 For each customer, identify the number of days between consecutive orders and flag any gap greater than 7 days.
SELECT customer_id,order_id,order_date, 
DATEDIFF(order_date,LAG(order_date) OVER(PARTITION BY customer_id ORDER BY order_date)) AS days_since_last,
CASE
    WHEN DATEDIFF(order_date,LAG(order_date) OVER(PARTITION BY customer_id ORDER BY order_date)) >=7 THEN 'yes'
    ELSE 'no'
END AS large_gap_flag
FROM campusx.orders_sample;

### Q7 For each customer, maintain a running count of orders where total_amount > 500.
SELECT *,
SUM(CASE WHEN total_amount > 500 THEN 1 ELSE 0 END)
OVER (PARTITION BY customer_id ORDER BY order_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS count_above_500
FROM orders_sample;

### Q8 For each customer, flag the first order in any sequence where total_amount started to increase compared to previous order.
SELECT *, 
CASE 
    WHEN total_amount > LAG(total_amount) OVER(PARTITION BY customer_id ORDER BY order_date) THEN 'yes'
    ELSE 'no'
END AS started_increasing
FROM orders_sample;

### Q9 For each customer, assign a session ID that increments whenever there's a gap of more than 30 days between their orders.
``` sql
SELECT *,
  SUM(CASE WHEN date_diff > 30 THEN 1 ELSE 0 END)
    OVER(PARTITION BY customer_id ORDER BY order_date) AS session_id
FROM (
  SELECT *,
    DATEDIFF(order_date, LAG(order_date) OVER(PARTITION BY customer_id ORDER BY order_date)) AS date_diff
  FROM orders_sample
) t


