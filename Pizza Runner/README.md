# Case Study 2 - Pizza Runner
![Danny's Diner](https://8weeksqlchallenge.com/images/case-study-designs/2.png)
## ER Diagram
<img width="452" alt="Pizza Runner - ER Diagran" src="https://user-images.githubusercontent.com/93120413/147558806-1f1907ad-0a05-4b2c-a0ac-b6269bfa4632.png">

##  Data Set
### Table 1: 
The runners table shows the registration_date for each new runner

<img width="113" alt="Pizza Runner - Runners" src="https://user-images.githubusercontent.com/93120413/147558833-b35cde71-d835-448c-ab45-c38159107327.png">

### Table 2: 
- Customer pizza orders are captured in the customer_orders table with 1 row for each individual pizza that is part of the order. 
- The pizza_id relates to the type of pizza which was ordered whilst the exclusions are the ingredient_id values which should be removed from the pizza and the extras are the ingredient_id values which need to be added to the pizza.
- Note that customers can order multiple pizzas in a single order with varying exclusions and extras values even if the pizza is the same type! The exclusions and extras columns will need to be cleaned up before using them in your queries.

<img width="289" alt="Pizza Runner - Customer Orders" src="https://user-images.githubusercontent.com/93120413/147558836-d12cdbf1-0ef4-4b7e-9025-37ec9bf6645c.png">

### Table 3: 
- After each orders are received through the system - they are assigned to a runner - however not all orders are fully completed and can be cancelled by the restaurant or the customer. 
- The pickup_time is the timestamp at which the runner arrives at the Pizza Runner headquarters to pick up the freshly cooked pizzas. The distance and duration fields are related to how far and long the runner had to travel to deliver the order to the respective customer. 
- There are some known data issues with this table so be careful when using this in your queries - make sure to check the data types for each column in the ERD!

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
      COUNT(*) AS pizzaorder
   FROM pizza_runner.customer_orders;
   ```
   
#### Result
   | pizzaorder     | 
   |:--------------:|
   |       14       |
   
#### 2. How many unique customer orders were made?
  ```sql
  SELECT
    COUNT(DISTINCT order_id) AS uniquepizzaorder
  FROM pizza_runner.customer_orders;
  ```
  
#### Result
   |  uniquepizzaorder  | 
   |:------------------:|
   |         10         |
   
#### 3. How many successful orders were delivered by each runner?
  ```sql
  SELECT
    runner_id, 
    COUNT(order_id) AS delivery_order
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
    SUM(CASE WHEN  pizza_id = 1 THEN 1 ELSE 0 END) AS MeatLover,
    SUM(CASE WHEN  pizza_id = 2 THEN 1 ELSE 0 END) AS VegLover
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
    SUM(CASE WHEN pizza_id = 1 THEN 1 ELSE 0 END) AS MeatLover,
    SUM(CASE WHEN pizza_id = 2 THEN 1 ELSE 0 END) AS VegLover 
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
    COUNT(order_id) AS Total
  FROM CTE
  GROUP BY order_id, customer_id
  ORDER BY order_id
  )
  SELECT
    MAX(Total) AS MaxDelivery
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
    COUNT(*) AS deliveredpizza
  FROM CTE 
  WHERE exclusions IS NOT NULL and extras IS NOT NULL;
  ```
  
#### Result
   | deliveredpizza | 
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
    COUNT(*) AS orderperhour
  FROM CTE 
  GROUP BY OrderTime
  ORDER BY OrderTime ASC;
  ```

#### Result
   |order_time      |  orderperhour  | 
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
    COUNT(*) AS Total
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
 
## B. Runner and Customer Experience
#### 1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
  ```sql
  SELECT
    DATE_TRUNC('Week', registration_date) + INTERVAL '3 Days' AS registration,
    COUNT(*) AS signedrunner
  FROM pizza_runner.runners
  GROUP BY registration 
  ORDER BY registration;
  ```
  
#### Result
   |registration    |  signedrunner  | 
   |:--------------:|:--------------:|
   |   2020-12-31   |        2       |
   |   2021-01-07   |        1       |
   |   2021-01-14   |        1       |

#### 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
  ```sql
  DROP TABLE IF EXISTS CTE1;
  CREATE TEMP TABLE CTE1 AS
  WITH CTE AS(
  SELECT
    customer_orders.order_id,
    AGE(runner_orders.pickup_time::TIMESTAMP, customer_orders.order_time)AS pickup_minutes
  FROM pizza_runner.customer_orders
  INNER JOIN pizza_runner.runner_orders
  ON customer_orders.order_id = runner_orders.order_id
  WHERE runner_orders.pickup_time != 'null'
  )
  SELECT
    order_id,
    DATE_PART('Minute', pickup_minutes) AS diff,
    DATE_PART('Second', pickup_minutes) AS diff1
  FROM CTE;
  ```
  ```sql
  WITH CTE AS(
  SELECT
    order_id,
    CONCAT(diff, '.', diff1)::NUMERIC AS Total
  FROM CTE1
  )
  SELECT
    AVG(Total) AS average
  FROM CTE;
  ```
  
#### Result
   |     average    | 
   |:--------------:|
   | 18.47166666667 |

#### 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?
  ```sql
  WITH CTE AS(
  SELECT
    customer_orders.customer_id,
    customer_orders.order_id,
    customer_orders.order_time,
    AGE(runner_orders.pickup_time::TIMESTAMP, customer_orders.order_time)AS pickup_minutes
  FROM pizza_runner.customer_orders
  INNER JOIN pizza_runner.runner_orders
  ON customer_orders.order_id = runner_orders.order_id
  WHERE runner_orders.pickup_time != 'null'
  ORDER BY order_time
  )
  SELECT
    order_id,
    DATE_PART('Minutes', pickup_minutes),
    COUNT(order_id) AS PizzaCount
  FROM CTE
  GROUP BY order_id, pickup_minutes
  ORDER BY order_id, PizzaCount;
  ```
  
#### Result
   | order_id       |  date_part     | pizzacount     |
   |:--------------:|:--------------:|:--------------:|
   |       1        |       10       |       1        |
   |       2        |       10       |       1        |
   |       3        |       21       |       2        |
   |       4        |       29       |       3        |
   |       5        |       10       |       1        |
   |       7        |       10       |       1        |
   |       8        |       20       |       1        |
   |      10        |       15       |       2        |
   
#### 4. What was the average distance travelled for each customer?
  ```sql
  WITH CTE AS(
  SELECT
    customer_orders.customer_id,
    customer_orders.order_id,
    runner_orders.runner_id,
    runner_orders.distance,
    UNNEST(REGEXP_MATCH(runner_orders.distance, '(^[0-9,.]+)')) ::NUMERIC AS distance1
  FROM pizza_runner.customer_orders
  INNER JOIN pizza_runner.runner_orders
  ON customer_orders.order_id = runner_orders.order_id
  WHERE runner_orders.distance != 'null'
  )
  SELECT
    customer_id,
    ROUND(AVG(distance1),1) AS average
  FROM CTE
  GROUP BY customer_id
  ORDER BY customer_id;
  ```
  
#### Result
   | customer_id    | average        | 
   |:--------------:|:--------------:|
   |      101       |      20.0      |
   |      102       |      16.7      |
   |      103       |      23.4      |
   |      104       |      10.0      |
   |      105       |      25.0      |

#### 5. What was the difference between the longest and shortest delivery times for all orders?
  ```sql
  WITH CTE AS(
  SELECT
    runner_orders.duration,
    UNNEST(REGEXP_MATCH(runner_orders.duration, '(^[0-9,.]+)')) ::NUMERIC AS duration1
  FROM pizza_runner.runner_orders
  WHERE runner_orders.distance != 'null'
  )
  SELECT
    MAX(duration1)-min(duration1) AS max_diff
  FROM CTE;
  ```

#### Result
   |    max_diff    | 
   |:--------------:|
   |       30       |
   
#### 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?
  ```sql
  WITH CTE AS(
  SELECT
    runner_orders.runner_id,
    runner_orders.order_id,
    DATE_PART('Hour', customer_orders.order_time) AS Hour_of_Day,
    UNNEST(REGEXP_MATCH(runner_orders.distance, '(^[0-9,.]+)')) ::NUMERIC AS distance1,
    UNNEST(REGEXP_MATCH(runner_orders.duration, '(^[0-9,.]+)')) ::NUMERIC AS duration1
  FROM pizza_runner.customer_orders
  INNER JOIN pizza_runner.runner_orders
  ON customer_orders.order_id = runner_orders.order_id
  )
  SELECT
    runner_id,
    order_id,
    Hour_of_Day,
    distance1,
    duration1,
    ROUND(distance1/(duration1/60)) AS Speed
  FROM CTE;
  ```

#### Result
   |    runner_id   |    order_id    |  hour_of_day   |    distance1   |    duration1   |      speed     |
   |:--------------:|:--------------:|:--------------:|:--------------:|:--------------:|:--------------:|
   |        1       |        1       |       18       |       20       |       32       |       38       |
   |        1       |        2       |       19       |       20       |       27       |       44       |
   |        1       |        3       |       23       |      13.4      |       20       |       40       |
   |        1       |        3       |       23       |      13.4      |       20       |       40       |
   |        2       |        4       |       13       |      23.4      |       40       |       35       |
   |        2       |        4       |       13       |      23.4      |       40       |       35       |
   |        2       |        4       |       13       |      23.4      |       40       |       35       |
   |        3       |        5       |       21       |       10       |       15       |       40       |
   |        2       |        7       |       21       |       25       |       25       |       60       |
   |        2       |        8       |       23       |      23.4      |       15       |       94       |
   |        1       |       10       |       18       |       10       |       10       |       60       |
   |        1       |       10       |       18       |       10       |       10       |       60       |
   
#### 7. What is the successful delivery percentage for each runner?
  ```sql
  SELECT
    runner_id,
    (ROUND(100*SUM(CASE WHEN pickup_time != 'null' THEN 1 ELSE 0 END))/count(*)) AS successpercent
  FROM pizza_runner.runner_orders
  GROUP BY runner_id
  ORDER BY runner_id;
  ```

#### Result
   | runner_id      | successpercent | 
   |:--------------:|:--------------:|
   |        1       |       100      |
   |        2       |       75       |
   |        3       |       60       |
   
## C. Ingredient Optimisation
#### 1. What are the standard ingredients for each pizza?
  ```sql
  DROP TABLE IF EXISTS NEWW;
  CREATE TEMP TABLE NEWW AS
  WITH CTE AS(
  SELECT
    pizza_id,
    REGEXP_SPLIT_TO_TABLE(toppings, '[,\s]+')::INTEGER AS topping_id
  FROM pizza_runner.pizza_recipes
  )
  SELECT
    pizza_id,
    string_agg(cte.topping_id::VARCHAR, ',') as topping_id,
    topping_name
  FROM CTE 
  INNER JOIN pizza_runner.pizza_toppings AS t2
  ON CTE.topping_id = t2.topping_id
  GROUP BY pizza_id, topping_name
  ORDER BY pizza_id;
  ```
  
  ```sql
  CREATE TEMP TABLE NEWW1 AS
  SELECT 
    pizza_id,
    STRING_AGG(NEWW.topping_name, ',') as topping_id
  FROM NEWW
  GROUP BY pizza_id;
  ```
  
#### Result
   |     pizza_id   |                                  topping_id                           | 
   |:--------------:|:---------------------------------------------------------------------:|
   |        2       |         Cheese,Mushrooms,Onions,Peppers,Tomatoes,Tomato Sauce         |
   |        1       |     Bacon,BBQ Sauce,Beef,Cheese,Chicken,Mushrooms,Pepperoni,Salami    |

#### 2. What was the most commonly added extra?
  ```sql
  DROP TABLE IF EXISTS NEW1;
  CREATE TEMP TABLE NEW1 AS 
  WITH CTE AS(
  SELECT
    order_id,
    customer_id,
    pizza_id,
    CASE 
      WHEN extras IN ('null', '') THEN NULL ELSE extras END
  FROM pizza_runner.customer_orders
  )
  SELECT 
    order_id,
    customer_id,
    pizza_id,
    REGEXP_SPLIT_TO_TABLE(extras, '[,\s]+')::INTEGER AS topping_id
  FROM CTE 
  WHERE extras IS NOT NULL;
  ```
  ```sql
  SELECT 
    NEW1.topping_id,
    COUNT(NEW1.topping_id) AS common_extras,
    pizza_toppings.topping_name AS Toppings_names
  FROM NEW1
  INNER JOIN pizza_runner.pizza_toppings
  ON NEW1.topping_id = pizza_toppings.topping_id
  GROUP BY Toppings_names, NEW1.topping_id
  ORDER BY common_extras DESC;
  ```
  
#### Result
   | topping_id     |  common_extras | toppings_names |
   |:--------------:|:--------------:|:--------------:|
   |       1        |        4       |     Bacon      |
   |       5        |        1       |    Chicken     |
   |       4        |        1       |     Cheese     |
   
#### 3. What was the most common exclusion?
  ```sql
  DROP TABLE IF EXISTS NEW2;
  CREATE TEMP TABLE NEW2 AS 
  WITH CTE AS(
  SELECT
    order_id,
    customer_id,
    pizza_id,
    CASE 
      WHEN exclusions IN ('null', '') THEN NULL ELSE exclusions END
  FROM pizza_runner.customer_orders
  )
  SELECT 
    order_id,
    customer_id,
    pizza_id,
    REGEXP_SPLIT_TO_TABLE(exclusions, '[,\s]+')::INTEGER AS topping_id
  FROM CTE 
  WHERE exclusions IS NOT NULL;
  ```
  
  ```sql
  SELECT 
    NEW2.topping_id,
    COUNT(NEW2.topping_id) AS common_extras,
    pizza_toppings.topping_name AS Toppings_names
  FROM NEW2
  INNER JOIN pizza_runner.pizza_toppings
  ON NEW2.topping_id = pizza_toppings.topping_id
  GROUP BY Toppings_names, NEW2.topping_id
  ORDER BY common_extras DESC, NEW2.topping_id;
  ```  
  
#### Result
   | topping_id     |  common_extras | toppings_names |
   |:--------------:|:--------------:|:--------------:|
   |       4        |        4       |    Cheese      |
   |       2        |        1       |   BBQ Sauce    |
   |       6        |        1       |   Mushroom     |
   
#### 4. Generate an order item for each record in the customers_orders table in the format of one of the following: Meat Lovers, Meat Lovers - Exclude Beef, Meat Lovers - Extra Bacon, Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers
  ```sql
  WITH cte_cleaned_customer_orders AS (
  SELECT
    order_id,
    customer_id,
    pizza_id,
    CASE 
      WHEN exclusions IN ('', 'null') THEN NULL ELSE exclusions END AS exclusions,
    CASE 
      WHEN extras IN ('', 'null') THEN NULL ELSE extras END AS extras,
    order_time,
    ROW_NUMBER() OVER () AS original_row_number
  FROM pizza_runner.customer_orders
  ),
  cte_extras_exclusions AS (
    SELECT
    order_id,
    customer_id,
    pizza_id,
    REGEXP_SPLIT_TO_TABLE(exclusions, '[,\s]+')::INTEGER AS exclusions_topping_id,
    REGEXP_SPLIT_TO_TABLE(extras, '[,\s]+')::INTEGER AS extras_topping_id,
    order_time,
    original_row_number
  FROM cte_cleaned_customer_orders
  UNION
  SELECT
    order_id,
    customer_id,
    pizza_id,
    NULL AS exclusions_topping_id,
    NULL AS extras_topping_id,
    order_time,
    original_row_number
  FROM cte_cleaned_customer_orders
  WHERE exclusions IS NULL AND extras IS NULL
  ),
  cte_complete_dataset AS (
  SELECT
    base.order_id,
    base.customer_id,
    base.pizza_id,
    names.pizza_name,
    base.order_time,
    base.original_row_number,
    STRING_AGG(exclusions.topping_name, ', ') AS exclusions,
    STRING_AGG(extras.topping_name, ', ') AS extras
  FROM cte_extras_exclusions AS base
  INNER JOIN pizza_runner.pizza_names AS names
  ON base.pizza_id = names.pizza_id
  LEFT JOIN pizza_runner.pizza_toppings AS exclusions
  ON base.exclusions_topping_id = exclusions.topping_id
  LEFT JOIN pizza_runner.pizza_toppings AS extras
  ON base.exclusions_topping_id = extras.topping_id
  GROUP BY base.order_id, base.customer_id, base.pizza_id, names.pizza_name, base.order_time, base.original_row_number 
  ),
  cte_parsed_string_outputs AS (
  SELECT
    order_id,
    customer_id,
    pizza_id,
    order_time,
    original_row_number,
    pizza_name,
    CASE 
      WHEN exclusions IS NULL THEN '' ELSE ' - Exclude ' || exclusions END AS exclusions,
    CASE 
      WHEN extras IS NULL THEN '' ELSE ' - Extra ' || exclusions END AS extras
  FROM cte_complete_dataset
  ),
  final_output AS (
  SELECT
    order_id,
    customer_id,
    pizza_id,
    order_time,
    original_row_number,
    pizza_name || exclusions || extras AS order_item
  FROM cte_parsed_string_outputs
  )
  SELECT
    order_id,
    customer_id,
    pizza_id,
    order_time,
    order_item
  FROM final_output
  ORDER BY original_row_number;
  ```
  
#### Result
   |    order_id    |   customer_id  |    pizza_id    |  order_time                |   order_item   | 
   |:--------------:|:--------------:|:--------------:|:--------------------------:|:--------------:|
   |        1       |       101      |       1        |  2021-01-01 18:05:02.000   |    Meatlovers  |
   |        2       |       101      |       1        |  2021-01-01 19:00:52.000   |    Meatlovers  |
   |        3       |       102      |       1        |  2021-01-01 18:05:02.000   |    Meatlovers  |
   |        3       |       102      |       2        |  2021-01-01 18:05:02.000   |    Meatlovers  |
   |        4       |       103      |       1        |  2021-01-01 18:05:02.000   |    Meatlovers  |
   |        4       |       103      |       1        |  2021-01-01 18:05:02.000   |    Meatlovers  |
   |        4       |       103      |       2        |  2021-01-01 18:05:02.000   |    Meatlovers  |
   |        5       |       104      |       1        |  2021-01-01 18:05:02.000   |    Meatlovers  |
   |        6       |       101      |       2        |  2021-01-01 18:05:02.000   |    Meatlovers  |
   |        7       |       105      |       2        |  2021-01-01 18:05:02.000   |    Meatlovers  |
   |        8       |       102      |       1        |  2021-01-01 18:05:02.000   |    Meatlovers  |
   |        9       |       103      |       1        |  2021-01-01 18:05:02.000   |    Meatlovers  |
   |       10       |       104      |       1        |  2021-01-01 18:05:02.000   |    Meatlovers  |
   |       10       |       104      |       1        |  2021-01-01 18:05:02.000   |    Meatlovers  |
   
#### 5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients + For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"
  ```sql
  WITH cte_cleaned_customer_orders AS (
  SELECT
    order_id,
    customer_id,
    pizza_id,
    CASE 
      WHEN exclusions IN ('', 'null') THEN NULL ELSE exclusions END AS exclusions,
    CASE 
      WHEN extras IN ('', 'null') THEN NULL ELSE extras END AS extras,
    order_time,
    ROW_NUMBER() OVER () AS original_row_number
  FROM pizza_runner.customer_orders
  ),
  cte_regular_toppings AS (
  SELECT
    pizza_id,
    REGEXP_SPLIT_TO_TABLE(toppings, '[,\s]+')::INTEGER AS topping_id
  FROM pizza_runner.pizza_recipes
  ),
  cte_base_toppings AS (
  SELECT
    cte_cleaned_customer_orders.order_id,
    cte_cleaned_customer_orders.customer_id,
    cte_cleaned_customer_orders.pizza_id,
    cte_cleaned_customer_orders.order_time,
    cte_cleaned_customer_orders.original_row_number,
    cte_regular_toppings.topping_id
  FROM cte_cleaned_customer_orders
  LEFT JOIN cte_regular_toppings
  ON cte_cleaned_customer_orders.pizza_id = cte_regular_toppings.pizza_id
  ),
  cte_extras AS (
  SELECT
    order_id,
    customer_id,
    pizza_id,
    order_time,
    original_row_number,
    REGEXP_SPLIT_TO_TABLE(extras, '[,\s]+')::INTEGER AS topping_id
  FROM cte_cleaned_customer_orders
  WHERE extras IS NOT NULL
  ),
  cte_combined_orders AS (
  SELECT * FROM cte_base_toppings
  UNION ALL
  SELECT * FROM cte_extras
  ),
  cte_joined_toppings AS (
  SELECT
    t1.order_id,
    t1.customer_id,
    t1.pizza_id,
    t1.order_time,
    t1.original_row_number,
    t1.topping_id,
    t2.pizza_name,
    t3.topping_name,
    COUNT(t1.*) AS topping_count
  FROM cte_combined_orders AS t1
  INNER JOIN pizza_runner.pizza_names AS t2
  ON t1.pizza_id = t2.pizza_id
  INNER JOIN pizza_runner.pizza_toppings AS t3
  ON t1.topping_id = t3.topping_id
  GROUP BY t1.order_id, t1.customer_id, t1.pizza_id, t1.order_time, t1.original_row_number, t1.topping_id, t2.pizza_name, t3.topping_name
  )
  SELECT
    order_id,
    customer_id,
    pizza_id,
    order_time,
    original_row_number,
    topping_count,
    pizza_name||':'||STRING_AGG(
    CASE
      WHEN topping_count > 1 THEN topping_count || 'x ' || topping_name
      ELSE topping_name
    END,
    ','
    ) AS ingredients_list
  FROM cte_joined_toppings
  GROUP BY order_id, customer_id, pizza_id, order_time, original_row_number, pizza_name, topping_count;
  ```
  
#### Result
   |    order_id    |   customer_id  |    pizza_id    |  order_time                |original_row_number|  topping_count |                            ingredients_list                              |
   |:--------------:|:--------------:|:--------------:|:--------------------------:|:-----------------:|:--------------:|:------------------------------------------------------------------------:|
   |        1       |       101      |       1        |  2021-01-01 18:05:02.000   |         1         |       1        |Meatlovers:BBQ Sauce,Beef,Cheese,Chicken,Mushrooms,Bacon,Pepperoni,Salami |
   |        2       |       101      |       1        |  2021-01-01 19:00:52.000   |         2         |       1        |Meatlovers:BBQ Sauce,Beef,Cheese,Chicken,Mushrooms,Bacon,Pepperoni,Salami |
   |        3       |       102      |       1        |  2021-01-01 18:05:02.000   |         3         |       1        |Meatlovers:BBQ Sauce,Beef,Cheese,Chicken,Mushrooms,Bacon,Pepperoni,Salami |
   |        3       |       102      |       2        |  2021-01-01 18:05:02.000   |         4         |       1        |Vegetarian:Tomato Sauce,Cheese,Mushrooms,Onions,Peppers,Tomatoes          |
   |        4       |       103      |       1        |  2021-01-01 18:05:02.000   |         5         |       1        |Meatlovers:Bacon,BBQ Sauce,Beef,Cheese,Chicken,Mushrooms,Pepperoni,Salami |
   |        4       |       103      |       1        |  2021-01-01 18:05:02.000   |         6         |       1        |Meatlovers:Bacon,BBQ Sauce,Beef,Cheese,Chicken,Mushrooms,Pepperoni,Salami |
   |        4       |       103      |       2        |  2021-01-01 18:05:02.000   |         7         |       1        |Vegetarian:Cheese,Mushrooms,Onions,Peppers,Tomatoes,Tomato Sauce          |
   |        5       |       104      |       1        |  2021-01-01 18:05:02.000   |         8         |       1        |Meatlovers:Salami,BBQ Sauce,Beef,Cheese,Chicken,Mushrooms,Pepperoni       |
   |        5       |       104      |       1        |  2021-01-01 18:05:02.000   |         8         |       2        |Meatlovers:2x Bacon                                                       |
   |        6       |       101      |       2        |  2021-01-01 18:05:02.000   |         9         |       1        |Vegetarian:Cheese,Mushrooms,Onions,Peppers,Tomatoes,Tomato Sauce          |
   |        7       |       105      |       2        |  2021-01-01 18:05:02.000   |        10         |       1        |Vegetarian:Bacon,Cheese,Mushrooms,Onions,Peppers,Tomatoes,Tomato Sauce    |
   |        8       |       102      |       1        |  2021-01-01 18:05:02.000   |        11         |       1        |Meatlovers:Bacon,BBQ Sauce,Beef,Cheese,Chicken,Mushrooms,Pepperoni,Salami |
   |        9       |       103      |       1        |  2021-01-01 18:05:02.000   |        12         |       1        |Meatlovers:Pepperoni,BBQ Sauce,Beef,Cheese,Mushrooms,Salami               |
   |        9       |       103      |       1        |  2021-01-01 18:05:02.000   |        12         |       2        |Meatlovers:2x Chicken,2x Bacon                                            |
   |       10       |       104      |       1        |  2021-01-01 18:05:02.000   |        13         |       1        |Meatlovers:BBQ Sauce,Beef,Cheese,Chicken,Mushrooms,Pepperoni,Salami,Bacon |
   |       10       |       104      |       1        |  2021-01-01 18:05:02.000   |        14         |       1        |Meatlovers:Pepperoni,BBQ Sauce,Beef,Mushrooms,Salami,Chicken              |
   |       10       |       104      |       1        |  2021-01-01 18:05:02.000   |        15         |       2        |Meatlovers:2x Cheese,2x Bacon                                             |

#### 6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?
  ```sql
  WITH cte_cleaned_customer_orders AS (
  SELECT
    order_id,
    customer_id,
    pizza_id,
    CASE
      WHEN exclusions IN ('', 'null') THEN NULL
      ELSE exclusions
    END AS exclusions,
    CASE
      WHEN extras IN ('', 'null') THEN NULL
      ELSE extras
    END AS extras,
    order_time,
    ROW_NUMBER() OVER () AS original_row_number
  FROM pizza_runner.customer_orders
  ),
  cte_regular_toppings AS (
    SELECT
    pizza_id,
    REGEXP_SPLIT_TO_TABLE(toppings, '[,\s]+')::INTEGER AS topping_id
  FROM pizza_runner.pizza_recipes
  ),
  cte_base_toppings AS (
  SELECT
    cte_cleaned_customer_orders.order_id,
    cte_cleaned_customer_orders.customer_id,
    cte_cleaned_customer_orders.pizza_id,
    cte_cleaned_customer_orders.order_time,
    cte_cleaned_customer_orders.original_row_number,
    cte_regular_toppings.topping_id
  FROM cte_cleaned_customer_orders
  LEFT JOIN cte_regular_toppings
  ON cte_cleaned_customer_orders.pizza_id = cte_regular_toppings.pizza_id
  ),
  cte_exclusions AS (
    SELECT
    order_id,
    customer_id,
    pizza_id,
    order_time,
    original_row_number,
    REGEXP_SPLIT_TO_TABLE(exclusions, '[,\s]+')::INTEGER AS topping_id
  FROM cte_cleaned_customer_orders
  WHERE exclusions IS NOT NULL
  ),
  cte_extras AS (
  SELECT
    order_id,
    customer_id,
    pizza_id,
    order_time,
    original_row_number,
    REGEXP_SPLIT_TO_TABLE(extras, '[,\s]+')::INTEGER AS topping_id
  FROM cte_cleaned_customer_orders
  WHERE extras IS NOT NULL
  ),
  cte_combined_orders AS (
    SELECT * FROM cte_base_toppings
    EXCEPT
    SELECT * FROM cte_exclusions
    UNION ALL
    SELECT * FROM cte_extras
  )
  SELECT
    t2.topping_name,
    COUNT(*) AS topping_count
  FROM cte_combined_orders AS t1
  INNER JOIN pizza_runner.pizza_toppings AS t2
  ON t1.topping_id = t2.topping_id
  GROUP BY t2.topping_name
  ORDER BY topping_count DESC;
  ```
  
#### Result
   | topping_name   | topping_count  | 
   |:--------------:|:--------------:|
   |     Bacon      |       14       |
   |    Mushrooms   |       13       |
   |     Chicken    |       11       |
   |     Cheese     |       11       |
   |    Pepperoni   |       10       |
   |     Salami     |       10       |
   |      Beef      |       10       |
   |   BBQ Sauce    |        9       |
   |  Tomato Sauce  |        4       |
   |     Onions     |        4       |
   |    Tomatoes    |        4       |
   |     Peppers    |        4       |
   
  
## D. Pricing and Ratings
#### 1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?
  ```sql
  SELECT
    SUM(
    CASE 
      WHEN pizza_id = 1 THEN 12 ELSE 10 END) AS Revenue
  FROM pizza_runner.customer_orders;
  ```
 
#### Result
   |      revenue   | 
   |:--------------:|
   |       160      |
   
#### 2. What if there was an additional $1 charge for any pizza extras? Add cheese is $1 extra
  ```sql
  DROP TABLE IF EXISTS table1;
  CREATE TEMP TABLE table1 AS 
  WITH CTE AS(
  SELECT 
    order_id,
    customer_id,
    pizza_id,
    REGEXP_SPLIT_TO_TABLE(extras, '[,\s]+') AS topping_id
  FROM pizza_runner.customer_orders
  )
  SELECT
    order_id,
    customer_id,
    pizza_id,
    topping_id
  FROM CTE;
  ```
  
  ```sql
  DROP TABLE IF EXISTS table2;
  CREATE TEMP TABLE table2 AS
  WITH cte_cleaned_customer_orders AS (
  SELECT
    order_id,
    customer_id,
    pizza_id,
    topping_id,
    CASE 
      WHEN topping_id IN ('', 'null') THEN NULL ELSE topping_id END AS extras
  FROM table1
  )
  SELECT
    order_id,
    SUM(CASE WHEN extras not IN ('null') THEN 1 ELSE 0 END)
    AS toppings
  FROM cte_cleaned_customer_orders
  GROUP BY order_id;
  ```
  
  ```sql    
  DROP TABLE IF EXISTS table3;
  CREATE TEMP TABLE table3 AS
  SELECT
    order_id,
    COUNT(*),
    SUM(
    CASE 
      WHEN pizza_id = 1 THEN 12 ELSE 10 END) AS Revenue
  FROM pizza_runner.customer_orders
  GROUP BY order_id
  ORDER BY order_id;
  ```
  
  ```sql
  SELECT
    SUM (Revenue + toppings) AS Price
  FROM table2
  INNER JOIN table3
  ON table2.order_id = table3.order_id;
  ```
  
#### Result
   |       price    | 
   |:--------------:|
   |       166      |
   
#### 3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.
  ```sql
  DROP TABLE IF EXISTS pizza_runner.ratings;
  CREATE TABLE pizza_runner.ratings (
    "order_id" INTEGER,
    "rating" INTEGER
  );
    
  INSERT INTO pizza_runner.ratings
  SELECT
    order_id,
    (RANDOM() * (1-5+1)+5)::int AS rating --May use Random/Ceiling before random
  FROM pizza_runner.runner_orders
  WHERE pickup_time IS NOT NULL;
    
  SELECT * FROM pizza_runner.ratings;
  ```
  
#### Result
   |    order_id    |     rating     | 
   |:--------------:|:--------------:|
   |       1        |        3       |
   |       2        |        4       |
   |       3        |        2       |
   |       4        |        4       |
   |       6        |        3       |
   |       7        |        2       |
   |       8        |        4       |
   |       9        |        4       |
   |      10        |        3       |
   |       2        |        3       |
   
#### 5. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
  ```sql
  WITH CTE AS(
  SELECT
    runner_orders.runner_id,
    runner_orders.order_id,
    customer_orders.customer_id,
    ratings.rating,
    customer_orders.order_time,
    runner_orders.pickup_time,
    AGE(runner_orders.pickup_time::TIMESTAMP, customer_orders.order_time)AS pickup_minutes,
    UNNEST(REGEXP_MATCH(runner_orders.distance, '(^[0-9,.]+)')) ::NUMERIC AS distance1,
    UNNEST(REGEXP_MATCH(runner_orders.duration, '(^[0-9,.]+)')) ::NUMERIC AS duration1
  FROM pizza_runner.customer_orders
  INNER JOIN pizza_runner.runner_orders
  ON customer_orders.order_id = runner_orders.order_id
  INNER JOIN pizza_runner.ratings
  ON customer_orders.order_id = ratings.order_id
  )
  SELECT
    order_id,
    runner_id,
    rating,
    order_time,
    pickup_time,
    DATE_PART('Minutes',pickup_minutes) AS diff_time,
    ROUND(distance1/(duration1/60),1) AS avg_speed,
    COUNT(order_id) AS pizza_count
  FROM CTE
  GROUP BY order_id, runner_id, rating, order_time, pickup_time, diff_time, avg_speed
  ORDER BY order_id;
  ```
 
 #### Result
   |    order_id    |    runner_id   |      rating    |        order_time       |      pickup_time    |     diff_time     |  avg_speed   |     pizza_count   | 
   |:--------------:|:--------------:|:--------------:|:-----------------------:|:-------------------:|:-----------------:|:------------:|:-----------------:|
   |        1       |       1        |        3       | 2021-01-01 18:05:02.000 | 2021-01-01 18:15:34 |         10        |     37.5     |          1        |
   |        2       |       1        |        4       | 2021-01-01 19:00:52.000 | 2021-01-01 19:10:54 |         10        |     44.4     |          1        |
   |        3       |       1        |        2       | 2021-01-02 23:51:23.000 | 2021-01-03 00:12:37 |         21        |     40.2     |          2        |
   |        4       |       2        |        4       | 2021-01-04 13:23:46.000 | 2021-01-04 13:53:03 |         29        |     35.1     |          3        |
   |        5       |       3        |        3       | 2021-01-08 21:00:29.000 | 2021-01-08 21:10:57 |         10        |     40.0     |          1        |
   |        7       |       2        |        4       | 2021-01-08 21:20:29.000 | 2021-01-08 21:30:45 |         10        |     60.0     |          1        |
   |        8       |       2        |        4       | 2021-01-09 23:54:33.000 | 2021-01-10 00:15:02 |         20        |     93.6     |          1        |
   |       10       |       1        |        3       | 2021-01-11 18:34:49.000 | 2021-01-11 18:50:20 |         15        |     60.0     |          2        |
   
 #### 5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?
  ```sql
  WITH CTE AS(
  SELECT
    runner_orders.runner_id,
    runner_orders.order_id,
    CASE 
      WHEN pizza_id = 1 THEN 12 ELSE 10 END AS Price,
    UNNEST(REGEXP_MATCH(runner_orders.distance, '(^[0-9,.]+)'))::NUMERIC AS distance1
  FROM pizza_runner.customer_orders
  INNER JOIN pizza_runner.runner_orders
  ON customer_orders.order_id = runner_orders.order_id
  INNER JOIN pizza_runner.ratings
  ON customer_orders.order_id = ratings.order_id
  WHERE pickup_time IS NOT NULL
  )
  SELECT
  SUM(price-round((distance1*0.30),2)) AS leftoverrevenue
  FROM CTE;
  ```

#### Result
   |  leftoverrevenue  | 
   |:-----------------:|
   |        73.38      |

## E. Bonus Questions
#### 1. If Danny wants to expand his range of pizzas - how would this impact the existing data design? Write an INSERT statement to demonstrate what would happen if a new Supreme pizza with all the toppings was added to the Pizza Runner menu?
  ```sql
  DROP TABLE IF EXISTS temp_pizza;
  CREATE TEMP TABLE temp_pizza AS
  SELECT * FROM pizza_runner.pizza_names;

  INSERT INTO temp_pizza 
  VALUES (3, 'Supreme');
    
  SELECT * FROM temp_pizza;
    
  DROP TABLE IF EXISTS temp_pizza_recipes;
  CREATE TEMP TABLE temp_pizza_recipes AS
  SELECT * FROM pizza_runner.pizza_recipes;

  INSERT INTO temp_pizza_recipes
  SELECT 3,
  STRING_AGG(topping_id::VARCHAR,',')
  FROM pizza_runner.pizza_toppings;
    
  SELECT * FROM temp_pizza_recipes;
  ```
 
#### Result 
   |    pizza_id    |           toppings           | 
   |:--------------:|:----------------------------:|
   |        1       |    1, 2, 3, 4, 5, 6, 8, 10   |
   |        1       |      4, 6, 7, 9, 11, 12      | 
   |        1       |  1,2,3,4,5,6,7,8,9,10,11,12  |
   
