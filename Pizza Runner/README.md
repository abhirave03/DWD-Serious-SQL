# Case Study 2 - Pizza Runner
![Danny's Diner](https://8weeksqlchallenge.com/images/case-study-designs/2.png)
## ER Diagram
<img width="452" alt="Pizza Runner - ER Diagran" src="https://user-images.githubusercontent.com/93120413/147558806-1f1907ad-0a05-4b2c-a0ac-b6269bfa4632.png">

##  Data Set
### Table 1: 
The runners table shows the registration_date for each new runner

<img width="113" alt="Pizza Runner - Runners" src="https://user-images.githubusercontent.com/93120413/147558833-b35cde71-d835-448c-ab45-c38159107327.png">

### Table 2: 
Customer pizza orders are captured in the customer_orders table with 1 row for each individual pizza that is part of the order. The pizza_id relates to the type of pizza which was ordered whilst the exclusions are the ingredient_id values which should be removed from the pizza and the extras are the ingredient_id values which need to be added to the pizza.Note that customers can order multiple pizzas in a single order with varying exclusions and extras values even if the pizza is the same type! The exclusions and extras columns will need to be cleaned up before using them in your queries.

<img width="289" alt="Pizza Runner - Customer Orders" src="https://user-images.githubusercontent.com/93120413/147558836-d12cdbf1-0ef4-4b7e-9025-37ec9bf6645c.png">

### Table 3: 
After each orders are received through the system - they are assigned to a runner - however not all orders are fully completed and can be cancelled by the restaurant or the customer. The pickup_time is the timestamp at which the runner arrives at the Pizza Runner headquarters to pick up the freshly cooked pizzas. The distance and duration fields are related to how far and long the runner had to travel to deliver the order to the respective customer. There are some known data issues with this table so be careful when using this in your queries - make sure to check the data types for each column in the ERD!

<img width="344" alt="Pizza Runner - Runner Orders" src="https://user-images.githubusercontent.com/93120413/147558843-84e1e3ec-0174-48c6-b97e-8929c136787a.png">

### Table 4:
At the moment - Pizza Runner only has 2 pizzas available the Meat Lovers or Vegetarian!

<img width="94" alt="Pizza Runner - Pizza Names" src="https://user-images.githubusercontent.com/93120413/147558865-764c5af3-b90a-444b-8719-698460ed67db.png">

### Table 5:
Each pizza_id has a standard set of toppings which are used as part of the pizza recipe.

<img width="112" alt="Pizza Runner - Pizza Recipes" src="https://user-images.githubusercontent.com/93120413/147558870-9b6de2b7-cc8f-4d18-b281-c831eb87f38e.png">

### Table 6:
This table contains all of the topping_name values with their corresponding topping_id value.

<img width="104" alt="Pizza Runner - Pizza Toppings" src="https://user-images.githubusercontent.com/93120413/147558874-b7e313e7-de3c-41e2-bbf2-96fd8de443f5.png">

# Case Study Question and Answer
## A. Pizza Metrics
#### 1. How many pizzas were ordered?
  ```sql
   SELECT 
      count(*) 
   FROM pizza_runner.customer_orders;
   ```
   
#### Result
   | count          | 
   |:--------------:|
   |       14       |
   
#### 2. How many unique customer orders were made?
  ```sql
  SELECT
    count(DISTINCT order_id)
  FROM pizza_runner.customer_orders;
  ```
  
#### Result
   | count          | 
   |:--------------:|
   |       10       |
   
#### 3. How many successful orders were delivered by each runner?
  ```sql
  SELECT
    runner_id, 
    count(order_id) as delivery_order
  FROM pizza_runner.runner_orders
  WHERE cancellation IS NULL or cancellation not in ('Restaurant Cancellation', 'Customer Cancellation')
  GROUP BY runner_id
  ORDER BY runner_id;
  ```
  
#### Result
   | runner_id      | delivery_order | 
   |:--------------:|:--------------:|
   |        1       |       4        |
   |        2       |       3        |
   |        3       |       1        |

#### 4. How many of each type of pizza was delivered?
  ```sql
  WITH CTE AS(
  SELECT 
  *
  FROM pizza_runner.customer_orders
  INNER JOIN pizza_runner.runner_orders
  ON customer_orders.order_id = runner_orders.order_id
  )
  SELECT
    SUM (CASE WHEN  pizza_id = 1 THEN 1 ELSE 0 END) AS MeatLover,
    SUM (CASE WHEN  pizza_id = 2 THEN 1 ELSE 0 END) AS VegLover
  FROM CTE 
  WHERE cancellation IS NULL or cancellation not in ('Restaurant Cancellation', 'Customer Cancellation');  
  ```
  
#### Result
   | meat_lover     | veg_lover      | 
   |:--------------:|:--------------:|
   |        9       |       3        |

#### 5. How many Vegetarian and Meatlovers were ordered by each customer?
  ```sql
  SELECT
    customer_id,
    SUM (CASE WHEN pizza_id = 1 THEN 1 ELSE 0 END) AS MeatLover,
    SUM (CASE WHEN pizza_id = 2 THEN 1 ELSE 0 END) AS VegLover 
  FROM pizza_runner.customer_orders
  GROUP BY customer_id	
  ORDER BY customer_id;
  ```

#### Result
   | customer_id    | meat_lover     |  meat_lover    |
   |:--------------:|:--------------:|:--------------:|
   |      101       |       2        |       1        |
   |      102       |       2        |       1        |
   |      103       |       3        |       1        |
   |      104       |       3        |       0        |
   |      105       |       0        |       1        |

#### 6. What was the maximum number of pizzas delivered in a single order?
  ```sql
  DROP TABLE IF EXISTS CTE;
  CREATE TEMP TABLE CTE AS
  SELECT 
    customer_orders.order_id,
    customer_orders.customer_id,
    customer_orders.pizza_id,
    runner_orders.cancellation
  FROM pizza_runner.customer_orders
  INNER JOIN pizza_runner.runner_orders
  ON customer_orders.order_id = runner_orders.order_id
  WHERE cancellation IS NULL or cancellation not in ('Restaurant Cancellation', 'Customer Cancellation');
  ```
  
  ```sql
  WITH CTE1 AS(
  SELECT 
    order_id,customer_id,
    count(order_id) AS Total
  FROM CTE
  GROUP BY order_id, customer_id
  ORDER BY order_id
  )
  SELECT
    max(Total) AS MaxDelivery
  FROM CTE1;
  ```

#### Result
   | maxdelivery    | 
   |:--------------:|
   |        3       |

#### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
  ```sql
  DROP TABLE IF EXISTS newcustomer1;
  CREATE TEMP TABLE newcustomer1 AS
  SELECT
    customer_orders.order_id,
    customer_orders.customer_id,
    customer_orders.pizza_id,
    runner_orders.cancellation,
    CASE 
      WHEN exclusions IN ('null', '') THEN NULL ELSE exclusions END,
    CASE 
      WHEN extras IN ('null', '') THEN NULL ELSE extras END
   FROM pizza_runner.customer_orders
   INNER JOIN pizza_runner.runner_orders
   ON customer_orders.order_id = runner_orders.order_id
   WHERE cancellation IS NULL or cancellation not in ('Restaurant Cancellation', 'Customer Cancellation');
   ```
   
   ```sql   
   SELECT
    customer_id,
    SUM(CASE WHEN exclusions IS NOT NULL OR extras IS NOT NULL THEN 1 ELSE 0 END) AS at_least_1_change,
    SUM(CASE WHEN exclusions IS NULL AND extras IS NULL THEN 1 ELSE 0 END ) AS no_changes
   FROM newcustomer1
   GROUP BY customer_id
   ORDER BY customer_id;
   ```  

#### Result
   | customer_id    |atleast_1_change|  no_changes    |
   |:--------------:|:--------------:|:--------------:|
   |      101       |       0        |       2        |
   |      102       |       0        |       3        |
   |      103       |       3        |       0        |
   |      104       |       2        |       1        |
   |      105       |       1        |       0        |

#### 8. How many pizzas were delivered that had both exclusions and extras?
  ```sql
  WITH CTE AS(
  SELECT
    order_id,
    customer_id,
    pizza_id,
    CASE 
      WHEN exclusions IN ('null', '') THEN NULL ELSE exclusions END,
    CASE 
      WHEN extras IN ('null', '') THEN NULL ELSE extras END
  FROM pizza_runner.customer_orders
  )
  SELECT 
    count(*)
  FROM CTE 
  WHERE exclusions IS NOT NULL and extras IS NOT NULL;
  ```
  
#### Result
   |      count     | 
   |:--------------:|
   |        2       |

#### 9. What was the total volume of pizzas ordered for each hour of the day?
  ```sql
  WITH CTE AS(
  SELECT
    EXTRACT (Hours FROM order_time) AS OrderTime
  FROM pizza_runner.customer_orders
  )
  SELECT
    OrderTime,
    count(*)
  FROM CTE 
  GROUP BY OrderTime
  ORDER BY OrderTime ASC;
  ```

#### Result
   |order_time      | count          | 
   |:--------------:|:--------------:|
   |        11      |        1       |
   |        13      |        3       |
   |        18      |        3       |
   |        19      |        1       |
   |        21      |        1       |
   |        23      |        3       |
   

#### 10. What was the volume of orders for each day of the week?
  ```sql
  WITH CTE AS(
  SELECT
    TO_CHAR(order_time, 'Day') AS Days
  FROM pizza_runner.customer_orders
  )
  SELECT
    Days,
    count(*) AS Total
  FROM CTE
  GROUP BY Days 
  ORDER BY Total;
  ```

#### Result
   |days            | total          | 
   |:--------------:|:--------------:|
   |     Sunday     |        1       |
   |     Saturday   |        3       |
   |     Monday     |        5       |
   |     Friday     |        5       |
 
