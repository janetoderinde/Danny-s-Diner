# Danny-s-Diner
8 Week SQL Challenge - Case Study 1
![image](https://user-images.githubusercontent.com/98917179/168800087-a107a92f-0be0-43d2-9452-bb0c4ac12687.png)
This is my solution to Danny Ma's 8 Week SQL Challenge Case Study 1 using PostgreSQL. Danny wants to use data from his restaurant to answer some questions about his customers. He plans on using the insights derived to make better business decisions. You could check out the challenge [here](https://8weeksqlchallenge.com/case-study-1/) 

# Challenge Description
There are 3 tables from a restaurant database; sales, menu, and members. 
![image](https://user-images.githubusercontent.com/98917179/168802906-4a29738e-6e7e-4915-84b1-4d395b189ba3.png)

I want to help Danny gain insight from this by answering the questions below.

# Questions and Solutions
Using PostgreSQL, I provided the following solution:

Question 1: What is the total amount each customer spent at the restaurant?

Solution: Here, I joined two tables: sales and menu. Returned customer_id and SUM of prices aliased as total_amount, then grouped by customer_id.
```SQL
SELECT customer_id, SUM(price) AS total_amount 
	FROM dannys_diner.sales AS sales 
	JOIN dannys_diner.menu AS menu 
	ON sales.product_id = menu.product_id
	GROUP BY customer_id
	ORDER BY customer_id;
```
 
 Question 2: How many days has each customer visited the restaurant?

Solution: Using COUNT() and DISTINCT() function, I got the different number of days each customer visited the restaurant by customer_id.
```SQL
SELECT customer_id, COUNT(DISTINCT(order_date))
	FROM dannys_diner.sales
	GROUP BY customer_id;
```

Question 3: What was the first item from the menu purchased by each customer?

Solution: To answer this question, a subquery was introduced. Menu and Sales table were also joined to get the product_name. However, one of the customers had two items from the menu on their first visit to the restaurant. It's not possible to determine which item was gotten first since we do not have the needed data. 
```SQL
SELECT DISTINCT(customer_id), product_name FROM dannys_diner.sales AS s
	JOIN dannys_diner.menu AS m ON m.product_id = s.product_id
	WHERE s.order_date IN (SELECT MIN(order_date) FROM dannys_diner.sales)
    GROUP BY customer_id, product_name
    ORDER BY customer_id;
```

Question 4: What is the most purchased item on the menu and how many times was it purchased by ALL customers?

Solution: Here, I joined sales and menu table on product_id to return the product_name with the highest total_number of times purchased. 
```SQL
SELECT product_name, COUNT(product_name) AS total_number
FROM dannys_diner.sales AS s 
	JOIN dannys_diner.menu AS m 
    ON s.product_id = m.product_id
	GROUP BY product_name 
    ORDER BY total_number DESC
    LIMIT 1;a
```

QUESTION 5: Which item is the most popular for each customer? 

Solution: The following functions were used - WITH(),DENSE_RANK() OVER. A rank table named r in which the customers were ranked by the Count of product bought (count) was created. Then SELECT customer_id, product_name, count.
```SQL
WITH r AS 
	(SELECT s.customer_id,
		m.product_name,
		COUNT(s.product_id) as count,
        DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY COUNT(s.product_id) DESC) AS r
	FROM dannys_diner.menu AS m 
	JOIN dannys_diner.sales s 
	ON s.product_id = m.product_id
	GROUP BY s.customer_id, m.product_name) 
SELECT customer_id, product_name, count
FROM r
WHERE r = 1;
```

Question 6: Which item was purchased first by the customer after they became a member?

Solution: I created a rank table- 'firsts' where customers were ranked (using DENSE_RANK()) by their order date. Then it was filtered using the WHERE clause to have order_date greater than or equal to join_date (date the customer became a member)
```SQL
 WITH firsts AS
	(SELECT s.customer_id,
       m1.product_name,
	DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date) AS firsts
	FROM dannys_diner.sales AS s
	JOIN dannys_diner.menu AS m1 
  	 ON s.product_id = m1.product_id
	JOIN dannys_diner.members AS m2
 	ON m2.customer_id = s.customer_id
	WHERE s.order_date >= m2.join_date)
SELECT * FROM firsts
WHERE firsts = 1;
```

Question 7: Which item was purchased just before the customer became a member?

Solution:
```SQL
 WITH firsts AS
(
SELECT s.customer_id,
       m1.product_name,
 DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date DESC) AS firsts, s.order_date, m2.join_date
FROM dannys_diner.sales AS s
JOIN dannys_diner.menu AS m1 
   ON s.product_id = m1.product_id
JOIN dannys_diner.members AS m2
 ON m2.customer_id = s.customer_id
WHERE s.order_date < m2.join_date)
SELECT * FROM firsts
WHERE firsts = 1;
```

Question 8: What are the total items and amount spent for each member before they became a member?

Solution: To answer this question, I joined the sales, menu , and members table. Then using the WHERE clause, I filtered the returned rows to have order_date less than join_date.
```SQL
SELECT s.customer_id, 
 count(s.product_id) AS total_items, 
        SUM(price) AS amount_spent
FROM dannys_diner.sales AS s
JOIN dannys_diner.menu AS m1 ON m1.product_id = s.product_id
JOIN dannys_diner.members AS m2 ON s.customer_id = m2.customer_id
WHERE s.order_date < m2.join_date
GROUP BY s.customer_id
ORDER BY s.customer_id;
```

Question 9: If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

Solution: CASE WHEN, and THEN worked for this. A new table: points_gained was created from menu. This was then JOINed to sales table on product_id to get the customer_id and SUM of points gained.
```SQL
WITH points_gained AS 
(
SELECT *,
    CASE 
    WHEN m.product_name = 'sushi' THEN price * 20
    WHEN m.product_name != 'sushi' THEN price * 10
    END AS points_gained
FROM dannys_diner.menu AS m
    )
SELECT customer_id, SUM(points_gained) AS total_points_gained
FROM dannys_diner.sales AS s
JOIN points_gained AS p ON p.product_id = s.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id;
```

Question 10:In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customers A and B have at the end of January?

Solution: A new table: points was created here. Calculated order date minus join date to get the difference in days. With the CASE() WHEN and THEN functions, I got to multiply points accordingly. Finally, the EXTRACT function for the month of January.

```SQL
SELECT customer_id, SUM(total_points)
FROM 
   (WITH points AS
               (SELECT s.customer_id, 
               (s.order_date - m2.join_date) AS first_week,
                  m1.price,
                  m1.product_name,
                s.order_date
                FROM dannys_diner.sales AS s
             JOIN dannys_diner.menu AS m1 ON s.product_id =                 m1.product_id
              JOIN dannys_diner.members AS m2
              ON m2.customer_id = s.customer_id)
            SELECT customer_id,
         CASE   
             WHEN first_week BETWEEN 0 AND 7 THEN price * 20
             WHEN (first_week > 7 OR first_week < 0) AND product_name = 'sushi' THEN price * 20
             WHEN (first_week > 7 OR first_week < 0) AND product_name != 'sushi' THEN price * 10
         END AS total_points,
        order_date
FROM points
WHERE EXTRACT(MONTH FROM order_date) = 1) as t
GROUP BY customer_id;
```
