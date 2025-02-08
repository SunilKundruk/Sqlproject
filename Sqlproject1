Basic queries
1.BList all unique cities where customers are located.
SELECT DISTINCT customer_city FROM customers;
1.	Count the number of orders placed in 2017
SELECT COUNT(order_id) 
FROM orders 
WHERE YEAR(order_purchase_timestamp) = 2017;
2.	Find the total sales per category
SELECT 
    UPPER(products.product_category) AS category, 
    ROUND(SUM(payments.payment_value), 2) AS sales
FROM products
JOIN order_items ON products.product_id = order_items.product_id
JOIN payments ON payments.order_id = order_items.order_id
GROUP BY category;
3.	Calculate the percentage of orders that were paid in installments.
SELECT 
    (SUM(CASE WHEN payment_installments > 1 THEN 1 ELSE 0 END) / COUNT(*)) * 100 AS installment_percentage
FROM payments;
4.	Count the number of customers from each state
SELECT customer_state, COUNT(customer_id) AS customer_count
FROM customers
GROUP BY customer_state
ORDER BY customer_count DESC;
5.	give me highest state name and percentage
SELECT customer_state, 
       COUNT(customer_id) AS customer_count, 
       ROUND((COUNT(customer_id) * 100.0 / (SELECT COUNT(*) FROM customers)), 2) AS percentage
FROM customers
GROUP BY customer_state
ORDER BY customer_count DESC
LIMIT 1;
INTERMEDIATE QUERIES

-- 7.Calculate the number of orders per month in 2018

SELECT 
    MONTHNAME(order_purchase_timestamp) AS month_name, 
    COUNT(order_id) AS order_count
FROM orders 
WHERE YEAR(order_purchase_timestamp) = 2018
GROUP BY MONTH(order_purchase_timestamp), month_name
ORDER BY MONTH(order_purchase_timestamp);

-- 8.Find the average number of products per order, grouped by customer city.

WITH count_per_order AS (
    SELECT 
        orders.customer_id, 
        COUNT(order_items.order_id) AS oc
    FROM orders 
    JOIN order_items ON orders.order_id = order_items.order_id
    GROUP BY orders.order_id, orders.customer_id
)
SELECT 
    customers.customer_city, 
    ROUND(AVG(count_per_order.oc), 2) AS average_orders
FROM customers 
JOIN count_per_order ON customers.customer_id = count_per_order.customer_id
GROUP BY customers.customer_city 
ORDER BY average_orders DESC;
8. SELECT 
    customers.customer_city, 
    ROUND(AVG(order_counts.oc), 2) AS average_orders
FROM customers
JOIN (
    SELECT orders.customer_id, COUNT(order_items.order_id) AS oc
    FROM orders
    JOIN order_items ON orders.order_id = order_items.order_id
    GROUP BY orders.order_id, orders.customer_id
) AS order_counts 
ON customers.customer_id = order_counts.customer_id
GROUP BY customers.customer_city
ORDER BY average_orders DESC;
-- 9.Calculate the percentage of total revenue contributed
-- by  each product category
SELECT 
    UPPER(products.product_category) AS category, 
    ROUND(
        (SUM(payments.payment_value) / (SELECT SUM(payment_value) FROM payments)) * 100, 2
    ) AS sales_percentage
FROM products 
JOIN order_items ON products.product_id = order_items.product_id
JOIN payments ON payments.order_id = order_items.order_id
GROUP BY category 
ORDER BY sales_percentage DESC;
-- 10.Calculate the percentage of total revenue contributed by each product category
SELECT 
    UPPER(products.product_category) AS category, 
    ROUND(
        (SUM(payments.payment_value) / (SELECT SUM(payment_value) FROM payments)) * 100, 2
    ) AS sales_percentage
FROM products 
JOIN order_items ON products.product_id = order_items.product_id
JOIN payments ON payments.order_id = order_items.order_id
GROUP BY category 
ORDER BY sales_percentage DESC;
11. you can approximate the relationship between product price and purchase count using the following alternative approaches:
-- 12b. What are the top-selling products, and are they cheap or expensive? 
SELECT 
    p.product_id,
    p.product_category,
    COUNT(oi.product_id) AS purchase_count,
    ROUND(AVG(oi.price), 2) AS avg_price
FROM products p
JOIN order_items oi ON p.product_id = oi.product_id
GROUP BY p.product_id, p.product_category
ORDER BY purchase_count DESC
LIMIT 10;  -- Get top 10 most purchased products

12Here’s a combined SQL query that merges Approach #2 (Average Purchase Count for Different Price Ranges) and Approach #3 (Ranking Products by Purchase Count & Price).
WITH product_sales AS (
    SELECT 
        oi.product_id, 
        COUNT(oi.product_id) AS purchase_count, 
        AVG(oi.price) AS avg_price
    FROM order_items oi
    GROUP BY oi.product_id
)
SELECT 
    CASE 
        WHEN avg_price < 50 THEN 'Low Price (< $50)'
        WHEN avg_price BETWEEN 50 AND 150 THEN 'Medium Price ($50 - $150)'
        ELSE 'High Price (> $150)'
    END AS price_category,
    COUNT(product_id) AS total_products,
    ROUND(AVG(purchase_count), 2) AS avg_purchases_per_product
FROM product_sales
GROUP BY price_category
ORDER BY avg_purchases_per_product DESC;
What This Query Does:
✅ Categorizes products into price ranges (Low, Medium, High).
✅ Counts how many products fall into each price range.
✅ Computes the average purchase count per product in each price category.
✅ Sorts by average purchases, showing which price range sells the most.
________________________________________
How to Interpret the Results
•	If low-priced products have the highest avg purchases, there's a negative correlation (cheaper products sell more).
•	If high-priced products have a high avg purchase count, there's a positive correlation (expensive products sell well).
•	If all categories have similar purchase counts, there may be no strong relationship between price and sales.
13. -- 13.Calculate the total revenue generated by each seller, and rank them by revenue.

WITH seller_revenue AS (
    SELECT 
        oi.seller_id, 
        SUM(p.payment_value) AS revenue
    FROM order_items oi
    JOIN payments p ON oi.order_id = p.order_id
    GROUP BY oi.seller_id
)
SELECT 
    seller_id, 
    revenue, 
    DENSE_RANK() OVER (ORDER BY revenue DESC) AS rank_position
FROM seller_revenue;
What This Query Does:
✅ Calculates total revenue per seller using SUM(payments.payment_value).
✅ Uses a CTE (WITH seller_revenue AS) to make the query more readable.
✅ Ranks sellers by revenue using DENSE_RANK().
✅ Avoids using *, making it more efficient.
________________________________________
Interpreting the Results
•	seller_id → The ID of the seller.
•	revenue → The total revenue generated by the seller.
•	rank_position → Seller ranking based on revenue (ties get the same rank).
ADAVNCED QUERIES
-- 14.Calculate the moving average of order values for each customer over their order history.
WITH customer_orders AS (
    SELECT 
        o.customer_id, 
        o.order_purchase_timestamp, 
        p.payment_value AS payment
    FROM orders o
    JOIN payments p ON o.order_id = p.order_id
)
SELECT 
    customer_id, 
    order_purchase_timestamp, 
    payment,
    ROUND(
        AVG(payment) OVER (
            PARTITION BY customer_id 
            ORDER BY order_purchase_timestamp
            ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
        ), 2
    ) AS moving_avg
FROM customer_orders;
🚀 How This Query Works
1.	Step 1: We first create a list of orders (customer_orders), which contains each customer’s orders and payment values.
2.	Step 2: We then use a window function (AVG OVER()) to calculate the 3-order moving average.
o	PARTITION BY customer_id → Ensures calculations happen separately for each customer.
o	ORDER BY order_purchase_timestamp → Sorts the orders by date.
o	ROWS BETWEEN 2 PRECEDING AND CURRENT ROW → Takes the current order and the previous 2 orders to compute the average.
________________________________________
🎯 What Insights Can You Get?
•	Track how customer spending changes over time.
•	Identify customers whose spending is increasing or decreasing.
•	Find seasonal spending patterns (e.g., higher spending during holidays).
________________________________________
🙌 Final Summary
•	The moving average smooths out order value fluctuations for each customer.
•	We use SQL window functions (AVG() OVER()) to calculate a 3-order moving average.
•	The query runs separately for each customer using PARTITION BY customer_id.
•	This helps businesses track customer spending trends over time.
15. Calculate the cumulative sales per month for each year
👉 You need to partition the cumulative sum by year so that sales reset at the start of each year instead of carrying over across years.
WITH monthly_sales AS (
    SELECT 
        YEAR(o.order_purchase_timestamp) AS years,
        MONTH(o.order_purchase_timestamp) AS months,
        ROUND(SUM(p.payment_value), 2) AS payment
    FROM orders o
    JOIN payments p ON o.order_id = p.order_id
    GROUP BY years, months
)
SELECT 
    years, 
    months, 
    payment, 
    SUM(payment) OVER (
        PARTITION BY years 
        ORDER BY months
    ) AS cumulative_sales
FROM monthly_sales;
-- 16. Calculate the year-over-year growth rate of total sales.
WITH yearly_sales AS (
    SELECT 
        YEAR(o.order_purchase_timestamp) AS years,
        ROUND(SUM(p.payment_value), 2) AS total_sales
    FROM orders o
    JOIN payments p ON o.order_id = p.order_id
    GROUP BY years
    ORDER BY years
)
SELECT 
    years, 
    total_sales, 
    ROUND(
        ((total_sales - LAG(total_sales, 1) OVER (ORDER BY years)) / 
        LAG(total_sales, 1) OVER (ORDER BY years)) * 100, 2
    ) AS YoY_Growth_Percentage
FROM yearly_sales;
-- 17.Calculate the retention rate of customers, defined as the percentage of customers who make another purchase within 6 months
-- of their first purchase.

WITH first_orders AS (
    SELECT customers.customer_id, MIN(orders.order_purchase_timestamp) AS first_order
    FROM customers 
    JOIN orders ON customers.customer_id = orders.customer_id
    GROUP BY customers.customer_id
), 
repeat_customers AS (
    SELECT DISTINCT orders.customer_id
    FROM orders 
    JOIN first_orders ON orders.customer_id = first_orders.customer_id
    WHERE orders.order_purchase_timestamp > first_orders.first_order
      AND orders.order_purchase_timestamp < DATE_ADD(first_orders.first_order, INTERVAL 6 MONTH)
)

SELECT 
    100 * (COUNT(DISTINCT repeat_customers.customer_id) / NULLIF(COUNT(DISTINCT first_orders.customer_id), 0)) 
    AS retention_rate
FROM first_orders
LEFT JOIN repeat_customers 
ON first_orders.customer_id = repeat_customers.customer_id;
This SQL query calculates the customer retention rate, which is the percentage of customers who made another purchase within 6 months of their first order.
If the customer retention rate is 0, it means that no customers made a second purchase within 6 months of their first purchase

-- 18.Identify the top 3 customers who spent the most money in each year
SELECT years, customer_id, payment, d_rank
FROM (
    SELECT 
        YEAR(o.order_purchase_timestamp) AS years,
        o.customer_id,
        SUM(p.payment_value) AS payment,
        DENSE_RANK() OVER (
            PARTITION BY YEAR(o.order_purchase_timestamp) 
            ORDER BY SUM(p.payment_value) DESC
        ) AS d_rank
    FROM orders o
    JOIN payments p ON o.order_id = p.order_id
    GROUP BY years, o.customer_id
) AS ranked_customers
WHERE d_rank <= 3;
-- 18.Identify the top 3 customers who spent the most money in each year
SELECT years, customer_id, payment, d_rank
FROM (
    SELECT 
        YEAR(o.order_purchase_timestamp) AS years,
        o.customer_id,
        SUM(p.payment_value) AS payment,
        DENSE_RANK() OVER (
            PARTITION BY YEAR(o.order_purchase_timestamp) 
            ORDER BY SUM(p.payment_value) DESC
        ) AS d_rank
    FROM orders o
    JOIN payments p ON o.order_id = p.order_id
    GROUP BY years, o.customer_id
) AS ranked_customers
WHERE d_rank <= 3;
This query ensures you get the top 3 spending customers per year efficiently! 🚀 Let me know if you need any modifications
Explanation:
1.	Extracts year of purchase (YEAR(o.order_purchase_timestamp)).
2.	Aggregates total spending per customer per year (SUM(p.payment_value)).
3.	Uses DENSE_RANK() to rank customers within each year based on their total spending.
4.	Filters the top 3 customers per year (WHERE d_rank <= 3).
- 19. Most Popular Product Bundles
-- Goal: Find products that are frequently bought together in the same order.

SELECT 
    oi1.product_id AS product_1, 
    oi2.product_id AS product_2, 
    COUNT(*) AS times_purchased_together -- Number of times these two products appear in the same order
FROM order_items oi1
JOIN order_items oi2 
    ON oi1.order_id = oi2.order_id 
    AND oi1.product_id < oi2.product_id
GROUP BY oi1.product_id, oi2.product_id
ORDER BY times_purchased_together DESC
LIMIT 10;
Explanation:
1.	Joins the order_items table to itself to find products in the same order.
2.	Filters so that each pair is counted only once (AND oi1.product_id < oi2.product_id).
3.	Counts how many times each product pair appears together (COUNT(*)).
📌 Why use this?
•	Helps recommend product bundles.
•	Used in cross-selling strategies.
 20.Finding Top Sellers with the Fastest DeliveryGoal: Identify top sellers based on total revenue and average delivery time.
SELECT 
    oi.seller_id, 
    SUM(p.payment_value) AS total_revenue, -- Total revenue for this seller
    ROUND(AVG(DATEDIFF(o.order_delivered_customer_date, o.order_purchase_timestamp)), 2) AS avg_delivery_time, -- Avg delivery time in days
    RANK() OVER (ORDER BY SUM(p.payment_value) DESC) AS revenue_rank -- Rank sellers by revenue
FROM order_items oi
JOIN payments p ON oi.order_id = p.order_id
JOIN orders o ON oi.order_id = o.order_id
WHERE o.order_status = 'delivered'
GROUP BY oi.seller_id
ORDER BY revenue_rank;
Explanation:
1.	Calculates revenue per seller (SUM(p.payment_value)).
2.	Finds the average delivery time (AVG(DATEDIFF(o.order_delivered_customer_date, o.order_purchase_timestamp))).
3.	Ranks sellers based on revenue (RANK() OVER (ORDER BY SUM(p.payment_value) DESC)).
📌 Why use this?
•	Helps identify best-performing sellers.
•	Useful for partnering with reliable sellers.
21. -- 21.Products with Highest Revenue Loss Due to Cancellations Identify which products are causing the most revenue loss due to cancellations.
SELECT 
    oi.product_id, 
    p.product_category, 
    SUM(oi.price) AS revenue_lost, -- Total revenue lost from cancellations
    COUNT(oi.order_id) AS canceled_orders -- Number of times the product was canceled
FROM order_items oi
JOIN orders o ON oi.order_id = o.order_id
JOIN products p ON oi.product_id = p.product_id
WHERE o.order_status = 'canceled'
GROUP BY oi.product_id, p.product_category
ORDER BY revenue_lost DESC
LIMIT 10;
Explanation:
1.	Filters only canceled orders (WHERE o.order_status = 'canceled').
2.	SUM(oi.price) → Calculates total revenue lost due to cancellations.
3.	COUNT(oi.order_id) → Counts the number of canceled orders per product.
📌 Why use this?
•	Helps find high-risk products with high cancellation rates.
•	Can be used to improve inventory and return policies.
22. -- 22.Customer Lifetime Value (CLV) Calculation
-- Identify how much revenue each customer has generated, their average order value, and rank them based on total spending.
SELECT 
    o.customer_id, 
    COUNT(DISTINCT o.order_id) AS total_orders, -- Total orders placed by the customer
    SUM(p.payment_value) AS total_spent, -- Total revenue from this customer
    ROUND(AVG(p.payment_value), 2) AS avg_order_value, -- Average revenue per order
    RANK() OVER (ORDER BY SUM(p.payment_value) DESC) AS revenue_rank -- Ranking based on spending
FROM orders o
JOIN payments p ON o.order_id = p.order_id
GROUP BY o.customer_id;
Explanation:
1.	COUNT(DISTINCT o.order_id) → Counts the number of unique orders per customer.
2.	SUM(p.payment_value) → Calculates the total amount a customer has spent.
3.	AVG(p.payment_value) → Finds the average amount spent per order.
4.	RANK() OVER (ORDER BY SUM(p.payment_value) DESC) → Ranks customers from highest to lowest spenders.
📌 Why use this?
•	Helps identify high-value customers.
•	Useful for loyalty programs and personalized marketing.

24. Monthly Revenue Growth Rate
👉 Goal: Calculate the month-over-month revenue growth rate.
-- 24. Monthly Revenue Growth Rate Calculate the month-over-month revenue growth rate.
WITH monthly_sales AS (
    SELECT 
        YEAR(order_purchase_timestamp) AS year, 
        MONTH(order_purchase_timestamp) AS month,
        SUM(payment_value) AS revenue
    FROM orders
    JOIN payments ON orders.order_id = payments.order_id
    GROUP BY year, month
)
SELECT 
    year, 
    month, 
    revenue, 
    ROUND(((revenue - LAG(revenue) OVER (ORDER BY year, month)) / 
            LAG(revenue) OVER (ORDER BY year, month)) * 100, 2) AS growth_rate
FROM monthly_sales;
Your query calculates monthly revenue and its growth rate over time. The output consists of four columns:
1.	Year → The year of the transaction.
2.	Month → The month of the transaction.
3.	Revenue → The total sales (sum of payment_value) for that month.
4.	Growth Rate (%) → The percentage change in revenue compared to the previous month.
Let's break it down with a few key insights:
1️⃣ Initial Revenue and Growth Calculation
Year	Month	Revenue (Total Sales)	Growth Rate (%)
2016	9	252.24	NULL (First value, no comparison)
2016	10	59090.48	📈 23326.29% (Huge growth, likely due to low initial sales)
2016	12	19.62	📉 -99.97% (Massive drop in revenue, possible issue)
•	September 2016 had very low revenue (₹252.24).
•	October 2016 saw a huge increase (₹59,090.48) → Growth rate = +23,326%.
•	December 2016 revenue dropped to just ₹19.62 → Growth rate = -99.97% (almost zero revenue).
•	There is no data for November 2016, which could indicate missing records or no sales.
________________________________________
2️⃣ Monthly Revenue Trends in 2017
Year	Month	Revenue	Growth Rate (%)
2017	1	138488.04	📈 705751.35% (Recovery after a big drop)
2017	2	291908.00	📈 110.78%
2017	3	449863.60	📈 54.11%
2017	4	417788.02	📉 -7.13%
2017	5	592918.82	📈 41.92%
2017	6	511276.38	📉 -13.77%
•	January 2017 saw another massive jump in revenue (₹138,488.04) after the low December 2016.
•	February to May 2017 had consistent growth.
•	June 2017 had a drop of -13.77%, which could indicate a seasonal slowdown.
________________________________________
3️⃣ Peak Sales & Decline in 2018
Year	Month	Revenue	Growth Rate (%)
2018	1	1,115,004.18	📈 26.94%
2018	2	992,463.34	📉 -10.99%
2018	3	1,159,652.12	📈 16.85%
2018	4	1,160,785.48	📈 0.1%
2018	5	1,153,982.15	📉 -0.59%
2018	6	1,023,880.50	📉 -11.27%
2018	7	1,066,540.75	📈 4.17%
2018	8	1,022,425.32	📉 -4.14%
2018	9	4,439.54	📉 -99.57%
2018	10	589.67	📉 -86.72%
•	2018 was a peak revenue year, with January hitting over ₹1.1M in sales.
•	Steady fluctuations throughout 2018, with some negative growth months (-10.99%, -0.59%).
•	September 2018 crashed to ₹4,439, followed by October 2018 at just ₹589.67 → Huge losses (-99.57%, -86.72%).
________________________________________
🔎 Possible Reasons for the Drop in September & October 2018
1.	Seasonal Business Pattern → Maybe the product is highly seasonal, with high sales earlier in the year.
2.	Marketing or Operational Issues → A failed campaign, supply chain problems, or price hikes may have caused demand to drop.
3.	Data Issues / Missing Records → If transactions for some sellers/customers weren't recorded, it could show misleading results.
________________________________________
🚀 Key Takeaways
✅ Business Performance Evaluation
•	Growth trends show when business is booming or declining.
✅ Identifying Seasonal Effects
•	Sales peaks in Q1 of 2018 suggest demand rises early in the year.
✅ Detecting Business Problems
•	September 2018 saw a 99.57% drop in revenue → Requires investigation.
✅ Data Quality Checks
•	If revenue is suddenly ₹19 or ₹589, check if transactions were recorded properly.
________________________________________
🎯 Next Steps
•	Investigate reasons for September 2018 collapse.
•	Compare revenue with other factors (e.g., marketing spend, customer acquisition).
•	Check product categories contributing to sales.


