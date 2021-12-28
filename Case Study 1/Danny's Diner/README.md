# Case Study 1 - Danny's Diner
### 1st Case Study from the Course
![Danny's Diner](https://8weeksqlchallenge.com/images/case-study-designs/1.png)
## ER Diagram
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
1. What is the total amount each customer spent at the restaurant?
   ```sql
   SELECT 
      sales.customer_id AS customer_id, 
      Sum(menu.price) AS TotalAmount 
   FROM dannys_diner.menu 
   INNER JOIN dannys_diner.sales 
   ON menu.product_id = sales.product_id 
   GROUP BY customer_id 
   ORDER BY customer_id;

  #### Result
   | customer_id    | totalamount   |
   | -------------  |:-------------:|
   |       A        |      76       | 
   |       B        |      74       | 
   |       C        |      36       | 

2. How many days has each customer visited the restaurant?
   ```sql
   WITH CTE AS(
   SELECT 
      customer_id, 
      order_date, 
      count(*) AS Total 
   FROM dannys_diner.sales 
   GROUP BY customer_id, order_date 
   ORDER BY customer_id
   ) 
   SELECT 
      customer_id, 
      count(total) AS TotalDays 
   FROM CTE 
   GROUP BY customer_id;

  #### Result
   | customer_id    | totaldays     |
   | -------------  |:-------------:|
   |       A        |       4       | 
   |       B        |       6       | 
   |       C        |       2       | 

3. What was the first item from the menu purchased by each customer?
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

  #### Result
   | product_id     | customer_id   |
   | -------------  |:-------------:|
   |     sushi      |       A       | 
   |     curry      |       A       | 
   |     curry      |       B       | 
   |     ramen      |       C       | 
   
4. What is the most purchased item on the menu and how many times was it purchased by all customers?
   ```sql
   WITH CTE AS(
   SELECT 
      product_id, 
      count(*) AS Total 
   FROM dannys_diner.sales 
   GROUP BY 
      product_id
   ) 
   SELECT 
      menu.product_name, Total 
   FROM dannys_diner.menu 
   INNER JOIN CTE 
   ON menu.product_id = CTE.product_id 
   ORDER BY Total DESC 
   LIMIT 1;
   
  #### Result
   | product_name   | total         |
   | -------------  |:-------------:|
   |     ramen      |       8       | 

5. Which item was the most popular for each customer?
   ```sql
   WITH CTE AS(
   SELECT 
    sales.customer_id, 
    sales.product_id, 
    menu.product_name, 
    rank() over( PARTITION BY sales.customer_id 
    ORDER BY count(sales.product_id) desc
    ) as rank_order 
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
    
 #### Result
   | customert_id   | product_name  |
   | -------------  |:-------------:|
   |       A        |      ramen    | 
   |       B        |      sushi    | 
   |       B        |      ramen    | 
   |       B        |      curry    | 
   |       C        |      ramen    | 

6. Which item was purchased first by the customer after they became a member?
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

 #### Result
   | customert_id   |  order_date   |  product_name   |
   | -------------  |:-------------:|  -------------  |
   |       A        |      ramen    |                 |
