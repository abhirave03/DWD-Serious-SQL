# Case Study 1 - Danny's Diner
![Danny's Diner](https://8weeksqlchallenge.com/images/case-study-designs/1.png)
## Entity Relationship Diagram
<img width="625" alt="Danny's Diner - ER Diagram" src="https://user-images.githubusercontent.com/93120413/147490759-c03d8538-0e5d-402a-87a9-f71206624f52.png">

##  Data Set
### Table 1: Sales
The sales table captures all customer_id level purchases with an corresponding order_date and product_id information for when and what menu items were ordered.

<img width="153" alt="Danny's Diner - Sales Table" src="https://user-images.githubusercontent.com/93120413/147489705-829e945e-490a-498a-a22a-1af0a47a905a.png">

### Table 2: Menu

The menu table maps the product_id to the actual product_name and price of each menu item.

<img width="131" alt="Danny's Diner - Menu Table" src="https://user-images.githubusercontent.com/93120413/147489715-8a147e72-a2af-44b9-a248-0e739a073b6b.png">

### Table 3: Members

The final members table captures the join_date when a customer_id joined the beta version of the Danny’s Diner loyalty program.

<img width="104" alt="Danny's Diner - Members Table" src="https://user-images.githubusercontent.com/93120413/147489723-a39ac29a-be0d-43cf-99c9-8b9eb76e0eba.png"> 

## Case Study Question and Answer
#### 1. What is the total amount each customer spent at the restaurant?
   ```sql
   SELECT 
      sales.customer_id AS customer_id, 
      SUM(menu.price) AS TotalAmount 
   FROM dannys_diner.menu 
   INNER JOIN dannys_diner.sales 
   ON menu.product_id = sales.product_id 
   GROUP BY customer_id 
   ORDER BY customer_id;
   ```

#### Result
   | customer_id    | totalamount    |
   |:--------------:|:--------------:|
   |       A        |      76        | 
   |       B        |      74        | 
   |       C        |      36        | 

#### 2. How many days has each customer visited the restaurant?
   ```sql
   WITH CTE AS(
   SELECT 
      customer_id, 
      order_date, 
      COUNT(*) AS Total 
   FROM dannys_diner.sales 
   GROUP BY customer_id, order_date 
   ORDER BY customer_id
   ) 
   SELECT 
      customer_id, 
      COUNT(total) AS TotalDays 
   FROM CTE 
   GROUP BY customer_id;
   ```
   
#### Result
   | customer_id    | totaldays      |
   |:--------------:|:--------------:|
   |       A        |       4        | 
   |       B        |       6        | 
   |       C        |       2        | 

#### 3. What was the first item from the menu purchased by each customer?
   ```sql
   WITH cte AS(
   SELECT 
      customer_id, 
      RANK() OVER (PARTITION BY customer_id 
      ORDER BY order_date
   ) AS order_rank, 
   menu.product_name 
   FROM dannys_diner.sales 
   INNER JOIN dannys_diner.menu 
   ON sales.product_id = menu.product_id
   ) 
   SELECT 
      DISTINCT product_name, customer_id 
   FROM cte 
   WHERE order_rank = 1;
   ```
   
#### Result
   | product_id     | customer_id    |
   |:--------------:|:--------------:|
   |     sushi      |       A        | 
   |     curry      |       A        | 
   |     curry      |       B        | 
   |     ramen      |        C       | 
   
#### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
   ```sql
   WITH CTE AS(
   SELECT 
      product_id, 
      count(*) AS Total 
   FROM dannys_diner.sales 
   GROUP BY product_id
   ) 
   SELECT 
      menu.product_name, Total 
   FROM dannys_diner.menu 
   INNER JOIN CTE 
   ON menu.product_id = CTE.product_id 
   ORDER BY Total DESC 
   LIMIT 1;
   ```
#### Result
   | product_name   | total          |
   |:--------------:|:--------------:|
   |     ramen      |       8        | 

#### 5. Which item was the most popular for each customer?
   ```sql
   WITH CTE AS(
   SELECT 
      sales.customer_id, 
      sales.product_id, 
      menu.product_name, 
      RANK() over( PARTITION BY sales.customer_id 
      ORDER BY count(sales.product_id) desc
   ) AS rank_order 
   FROM dannys_diner.sales 
   INNER JOIN dannys_diner.menu ON sales.product_id = menu.product_id 
   GROUP BY sales.customer_id, sales.product_id, menu.product_name 
   ORDER BY sales.customer_id
   ) 
   SELECT 
      customer_id, 
      product_name 
   FROM cte 
   WHERE rank_order = 1;
   ```
    
#### Result
   | customert_id   | product_name   |
   |:--------------:|:--------------:|
   |       A        |      ramen     | 
   |       B        |      sushi     | 
   |       B        |      ramen     | 
   |       B        |      curry     | 
   |       C        |      ramen     | 

#### 6. Which item was purchased first by the customer after they became a member?
   ```sql
   WITH CTE AS(
   SELECT 
      sales.customer_id, 
      sales.product_id, 
      sales.order_date, 
      menu.product_name, 
      DENSE_RANK() OVER(PARTITION BY sales.customer_id 
      ORDER BY sales.order_date
   ) AS rank_order 
   FROM dannys_diner.sales 
   INNER JOIN dannys_diner.members ON sales.customer_id = members.customer_id 
   INNER JOIN dannys_diner.menu ON sales.product_id = menu.product_id 
   WHERE sales.order_date >= members.join_date
   ) 
   SELECT 
      customer_id, 
      order_date, 
      product_name 
   FROM CTE 
   where rank_order = 1;
   ```
    
#### Result
   | customert_id   |  order_date    |  product_name  |
   |:--------------:|:--------------:|:--------------:|
   |       A        |  2021-01-07    |      curry     |
   |       B        |  2021-01-11    |      sushi     |

#### 7. Which menu item(s) was purchased just before the customer became a member and when?
   ```sql
   WITH CTE AS(
   SELECT 
      sales.customer_id, 
      sales.product_id, 
      sales.order_date, 
      menu.product_name, 
      DENSE_RANK() OVER(PARTITION BY sales.customer_id 
      ORDER BY sales.order_date desc
   ) AS rank_order 
   FROM 
      dannys_diner.sales 
   INNER JOIN dannys_diner.members ON sales.customer_id = members.customer_id 
   INNER JOIN dannys_diner.menu ON sales.product_id = menu.product_id 
   WHERE sales.order_date < members.join_date
   ) 
   SELECT 
      customer_id, 
      order_date, 
      product_name 
   FROM CTE 
   where rank_order = 1;
   ```
 
 #### Result
   | customert_id   |  order_date    |  product_name  |
   |:--------------:|:--------------:|:--------------:|
   |       A        |  2021-01-01    |      sushi     |
   |       A        |  2021-01-01    |      curry     |
   |       B        |  2021-01-04    |      sushi     |
   
#### 8. What is the number of unique menu items and total amount spent for each member before they became a member?
   ```sql
   WITH CTE AS(
   SELECT 
      sales.customer_id, 
      sales.product_id, 
      sales.order_date, 
      menu.product_name, 
      menu.price, 
      DENSE_RANK() OVER(PARTITION BY sales.customer_id 
      ORDER BY sales.order_date desc
   ) AS rank_order 
   FROM dannys_diner.sales 
   INNER JOIN dannys_diner.members ON sales.customer_id = members.customer_id 
   INNER JOIN dannys_diner.menu ON sales.product_id = menu.product_id 
   WHERE sales.order_date < members.join_date
   ) 
   SELECT 
      customer_id, 
      COUNT(DISTINCT product_id), 
      SUM(price) 
   FROM CTE 
   GROUP BY customer_id 
   ORDER BY customer_id;
   ```
   
#### Result
   | customert_id   |  count         |  sum           |
   |:--------------:|:--------------:|:--------------:|
   |       A        |        2       |       25       |
   |       B        |        2       |       40       |
   
#### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
   ```sql
   WITH CTE AS (
   SELECT 
      customer_id, 
      order_date, 
      CASE 
         WHEN menu.product_name = 'curry' 
         or menu.product_name = 'ramen' THEN menu.price * 10 ELSE 2 * 10 * price END AS product_result 
   FROM 
      dannys_diner.menu 
   INNER JOIN dannys_diner.sales ON sales.product_id = menu.product_id 
   ORDER BY customer_id
   ) 
   SELECT 
      customer_id, 
      SUM(product_result) AS points 
   FROM CTE 
   GROUP BY customer_id 
   ORDER BY customer_id;
   ```
#### Result
   | customert_id   | points         |
   |:--------------:|:--------------:|
   |       A        |      860       |
   |       B        |      940       |
   |       C        |      360       | 

#### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
   ```sql
   DROP TABLE IF EXISTS dinner;
   CREATE TEMP TABLE dinner AS
   WITH CTE AS(
   SELECT
      sales.customer_id,
      sales.product_id,
      sales.order_date,
      menu.product_name,
      menu.price,
      DENSE_RANK() OVER(PARTITION BY sales.customer_id ORDER BY sales.order_date desc) AS rank_order,
      CASE
         WHEN product_name = 'curry' or product_name = 'ramen' THEN price * 10 
         ELSE 2 * 10 * price
      END AS product_result
   FROM dannys_diner.sales
   INNER JOIN dannys_diner.members
   ON sales.customer_id = members.customer_id
   INNER JOIN dannys_diner.menu
   ON sales.product_id = menu.product_id
   WHERE sales.order_date < members.join_date
   )
   SELECT
      customer_id,
      SUM(product_result) AS Result
   FROM CTE
   GROUP BY customer_id
   ORDER BY customer_id;
   ```

   ```sql
   DROP TABLE IF EXISTS dinner1;
   CREATE TEMP TABLE dinner1 AS
   WITH CTE1 AS(
   SELECT
      sales.customer_id,
      sales.product_id,
      sales.order_date,
      menu.product_name,
      menu.price,
      DENSE_RANK() OVER(PARTITION BY sales.customer_id ORDER BY sales.order_date desc) AS rank_order,
      CASE
         WHEN product_name = 'curry' or product_name = 'ramen' THEN price * 2 * 10 
         ELSE 2 * 10 * price
   END AS product_result1
   FROM dannys_diner.sales
   INNER JOIN dannys_diner.members
   ON sales.customer_id = members.customer_id
   INNER JOIN dannys_diner.menu
   ON sales.product_id = menu.product_id
   WHERE sales.order_date >= members.join_date
   )
   SELECT
      customer_id,
      SUM(product_result1) AS Result1
   FROM CTE1
   GROUP BY customer_id
   ORDER BY customer_id;
   ```
   ```sql
   select * from dinner;
   select * from dinner1;
   ```
    
   ```sql
   SELECT
      dinner.customer_id,
      dinner1.customer_id,
      SUM(Result + Result1)
   FROM dinner
   JOIN dinner1
   ON dinner.customer_id = dinner1.customer_id
   GROUP BY  dinner.customer_id,
   dinner1.customer_id;
   ```

#### Result
   | customert_id   | points         |
   |:--------------:|:--------------:|
   |       A        |      1370      |
   |       A        |      1180      |
   
#### Bonus Question
#### 11. Danny and his team can use to quickly derive insights without needing to join the underlying tables using SQL Recreate the following table output using the available data.
   ```sql
   SELECT
      sales.customer_id,
      sales.product_id,
      sales.order_date,
      members.join_date,
      CASE
         WHEN sales.order_date >= members.join_date THEN 'Y'
         ELSE 'N'
      END AS member
   FROM dannys_diner.sales
   FULL JOIN dannys_diner.members
   ON sales.customer_id = members.customer_id;
   ```
 
#### Result
   | customert_id   | product_id     | order_date     | join_date      |  member        |
   |:--------------:|:--------------:|:--------------:|:--------------:|:--------------:|
   |        A       |       1        |   2021-01-01   |   2021-01-07   |       N        |
   |        A       |       2        |   2021-01-01   |   2021-01-07   |       N        |
   |        A       |       2        |   2021-01-07   |   2021-01-07   |       Y        |
   |        A       |       3        |   2021-01-10   |   2021-01-07   |       Y        |
   |        A       |       3        |   2021-01-11   |   2021-01-07   |       Y        |
   |        A       |       3        |   2021-01-11   |   2021-01-07   |       Y        |
   |        B       |       2        |   2021-01-01   |   2021-01-09   |       N        |
   |        B       |       2        |   2021-01-02   |   2021-01-09   |       N        |
   |        B       |       1        |   2021-01-04   |   2021-01-09   |       N        |
   |        B       |       1        |   2021-01-11   |   2021-01-09   |       Y        |
   |        B       |       3        |   2021-01-16   |   2021-01-09   |       Y        |
   |        B       |       3        |   2021-02-01   |   2021-01-09   |       Y        |
   |        C       |       3        |   2021-01-01   |       null     |       N        |
   |        C       |       3        |   2021-01-01   |       null     |       N        |
   |        C       |       3        |   2021-02-07   |       null     |       N        |
             
#### 12.Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.
   ```sql
   DROP TABLE IF EXISTS purchases;
   CREATE TEMP TABLE purchases AS
   WITH CTE AS(
   SELECT
      sales.customer_id,
      sales.product_id,
      sales.order_date,
      members.join_date,
      CASE
         WHEN sales.order_date >= members.join_date THEN 'Y'
         ELSE 'N'
      END AS member
   FROM dannys_diner.sales
   FULL JOIN dannys_diner.members
   ON sales.customer_id = members.customer_id)
   SELECT
      customer_id,
      product_id,
      order_date,
      join_date,
      member,
      RANK() OVER (PARTITION BY customer_id ORDER BY member) AS ranked
   FROM CTE;
   ```
   ```sql
   SELECT customer_id,
      product_id,
      order_date,
      join_date,
      member,
      CASE WHEN 
         ranked = 1 THEN null 
         ELSE  DENSE_RANK() OVER (PARTITION BY customer_id, member ORDER BY order_date)
      END AS ranked1
   FROM purchases;
   ```
#### Result
   | customert_id   | product_id     | order_date     | join_date      |  member        |  ranked1       |
   |:--------------:|:--------------:|:--------------:|:--------------:|:--------------:|:--------------:|
   |        A       |       1        |   2021-01-01   |   2021-01-07   |       N        |       null     |
   |        A       |       2        |   2021-01-01   |   2021-01-07   |       N        |       null     |
   |        A       |       2        |   2021-01-07   |   2021-01-07   |       Y        |        1       |
   |        A       |       3        |   2021-01-10   |   2021-01-07   |       Y        |        2       |
   |        A       |       3        |   2021-01-11   |   2021-01-07   |       Y        |        3       |
   |        A       |       3        |   2021-01-11   |   2021-01-07   |       Y        |        3       |
   |        B       |       2        |   2021-01-01   |   2021-01-09   |       N        |       null     |
   |        B       |       2        |   2021-01-02   |   2021-01-09   |       N        |       null     |
   |        B       |       1        |   2021-01-04   |   2021-01-09   |       N        |       null     |
   |        B       |       1        |   2021-01-11   |   2021-01-09   |       Y        |        1       |
   |        B       |       3        |   2021-01-16   |   2021-01-09   |       Y        |        2       |
   |        B       |       3        |   2021-02-01   |   2021-01-09   |       Y        |        3       |
   |        C       |       3        |   2021-01-01   |       null     |       N        |       null     |
   |        C       |       3        |   2021-01-01   |       null     |       N        |       null     |
   |        C       |       3        |   2021-02-07   |       null     |       N        |       null     |

