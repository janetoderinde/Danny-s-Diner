# Danny-s-Diner
8 Week SQL Challenge - Week 1
![image](https://user-images.githubusercontent.com/98917179/168800087-a107a92f-0be0-43d2-9452-bb0c4ac12687.png)
Danny wants to use data from his restaurant to answer some questions about his customers. He plans on using the insights derived to make better business decisions. You could check out the challenge here: https://8weeksqlchallenge.com/case-study-1/

# Challenge Description
We have 3 tables from a restaurant database; sales, menu, and members. 
![image](https://user-images.githubusercontent.com/98917179/168802906-4a29738e-6e7e-4915-84b1-4d395b189ba3.png)

We want to help Danny gain insight from this by answering the questions below.

# Challenges and Solutions
Using PostgreSQL, I provided the following solution:

Question 1: What is the total amount each customer spent at the restaurant?

Solution: Here, I joined two tables: sales and menu. Returned customer_id and SUM of prices aliased as total_amount, then grouped by customer_id.

SELECT customer_id, 
  		SUM(price) AS total_amount
 FROM dannys_diner.sales AS sales
 JOIN dannys_diner.menu AS menu
 ON sales.product_id = menu.product_id
 GROUP BY customer_id
 ORDER BY customer_id;
 
 Question 2:
