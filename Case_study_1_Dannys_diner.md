# 8WeeksSQLChallenge

**Case Study #1 - Danny's Diner**

**Introduction**

Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.

**Problem Statement**

Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

Danny has provided with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!

Danny has shared with **3 key datasets for this case study and the Schema for the datasets** is as follows:

***************************************************************************************************************************************************************
CREATE SCHEMA dannys_diner;
SET search_path = dannys_diner;

CREATE TABLE sales (
  "customer_id" VARCHAR(1),
  "order_date" DATE,
  "product_id" INTEGER
);

INSERT INTO sales
  ("customer_id", "order_date", "product_id")
VALUES
  ('A', '2021-01-01', '1'),
  ('A', '2021-01-01', '2'),
  ('A', '2021-01-07', '2'),
  ('A', '2021-01-10', '3'),
  ('A', '2021-01-11', '3'),
  ('A', '2021-01-11', '3'),
  ('B', '2021-01-01', '2'),
  ('B', '2021-01-02', '2'),
  ('B', '2021-01-04', '1'),
  ('B', '2021-01-11', '1'),
  ('B', '2021-01-16', '3'),
  ('B', '2021-02-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-07', '3');
 

CREATE TABLE menu (
  "product_id" INTEGER,
  "product_name" VARCHAR(5),
  "price" INTEGER
);

INSERT INTO menu
  ("product_id", "product_name", "price")
VALUES
  ('1', 'sushi', '10'),
  ('2', 'curry', '15'),
  ('3', 'ramen', '12');
  

CREATE TABLE members (
  "customer_id" VARCHAR(1),
  "join_date" DATE
);

INSERT INTO members
  ("customer_id", "join_date")
VALUES
  ('A', '2021-01-07'),
  ('B', '2021-01-09');
 ************************************************************************************************************************************************************** 
  
**Case Study Questions**
  
1. What is the total amount each customer spent at the restaurant?

Answer:
      SELECT s.customer_id, 
      SUM(m.price) as total_amount
      FROM dannys_diner.sales AS s
      LEFT JOIN dannys_diner.menu AS m
      ON m.product_id = s.product_id
      GROUP BY s.customer_id
      ORDER BY total_amount DESC;

2. How many days has each customer visited the restaurant?

Answer: 
      SELECT s.customer_id
      COUNT(DISTINCT order_date) AS number_of_days
      FROM dannys_diner.sales AS s
      GROUP BY s.customer_id
      ORDER BY number_of_days DESC;

3. What was the first item from the menu purchased by each customer?

Answer: 
        WITH first_item AS (
            SELECT s.customer_id,
                   s.order_date,
                   m.product_name, 
                   DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date) AS RANK
                   FROM dannys_diner.sales AS s
                   LEFT JOIN dannys_diner.menu AS m
                   ON m.product_id = s.product_id
                   )
        SELECT 
              f.customer_id
              f.product_name
              FROM first_item as f
              WHERE RANK = 1
              GROUP BY 
              f.customer_id, f.product_name;

4. What is the most purchased item on the menu and how many times was it purchased by all customers?

Answer: 

SELECT m.product_name, COUNT(s.product_id) AS most_purchased_item
FROM dannys_diner.sales AS s
INNER JOIN dannys_diner.menu AS m 
ON m.product_id = s.product_id
GROUP BY s.product_id, m.product_name
ORDER BY most_purchased_item DESC
LIMIT 1;

5. Which item was the most popular for each customer?

Answer:
        WITH Product AS (
                        SELECT s.customer_id ,product_id, COUNT(s.product_id) AS items 
                        FROM dannys_diner.sales as s
                        GROUP BY customer_id, product_id
                        ),
                        ProductRanking AS (
                                          SELECT p.customer_id , m.product_name, p.items,  
                                          DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY p.items ) AS ranking 
                                          FROM Product AS p
                                          LEFT JOIN dannys_diner.menu AS m 
                                          ON p.product_id = m.product_id
                                          )
        SELECT customer_id , product_name 
        FROM ProductRanking 
        WHERE ranking = 1 ;

6. Which item was purchased first by the customer after they became a member?
Answer:
      WITH Product AS (
                      SELECT s.customer_id, m.join_date, s.order_date, s.product_id, 
                      DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date) as ranking
                      FROM dannys_diner.sales AS s
                      LEFT JOIN dannys_diner.members AS m
                      ON m.customer_id = s.customer_id 
                      where s.order_date > m.join_date
                      )
  
    SELECT p.customer_id, p.join_date, p.order_date, me.product_name
    FROM product AS p
    LEFT JOIN dannys_diner.menu AS me
    ON me.product_id = p.product_id
    WHERE ranking = 1
    ORDER BY customer_id

7. Which item was purchased just before the customer became a member?
Answer:
 WITH Product AS (
                      SELECT s.customer_id, m.join_date, s.order_date, s.product_id, 
                      DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date) as ranking
                      FROM dannys_diner.sales AS s
                      LEFT JOIN dannys_diner.members AS m
                      ON m.customer_id = s.customer_id 
                      where s.order_date < m.join_date
                      )
  
    SELECT p.customer_id, p.join_date, p.order_date, me.product_name
    FROM product AS p
    LEFT JOIN dannys_diner.menu AS me
    ON me.product_id = p.product_id
    WHERE ranking = 1
    ORDER BY customer_id                  

      
8. What is the total items and amount spent for each member before they became a member?
Answer:
         SELECT s.customer_id, COUNT(me.product_name) AS total_item, SUM(me.price) AS amount_spent
         FROM dannys_diner.sales AS s
         LEFT JOIN dannys_diner.members AS m
         ON m.customer_id = s.customer_id 
         LEFT JOIN dannys_diner.menu AS me
         ON me.product_id = s.product_id
         WHERE s.order_date < m.join_date
         GROUP BY s.customer_id
         
9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

Answer: 
       WITH CustomerPoints AS (
                              SELECT s.customer_id, s.product_id, SUM(
                                                                      CASE 
                                                                          WHEN s.product_id = 1 THEN m.price*20 
                                                                          ELSE m.price*10 END
                                                                          ) AS points_customer
                              FROM dannys_diner.sales AS s
                              LEFT JOIN dannys_diner.menu AS m 
                              ON m.product_id = s.product_id
                              GROUP BY s.customer_id, s.product_id
                              )
                                        
      SELECT p.customer_id, SUM(p.points_customer) AS total_points
      FROM CustomerPoints AS p
      GROUP BY p.customer_id
      ORDER BY total_points DESC;
      
10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

Answer:     








