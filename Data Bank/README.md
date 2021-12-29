# Case Study 3 - Data Bank
![Data Bank](https://8weeksqlchallenge.com/images/case-study-designs/4.png)
## Entity Relationship Diagram
<img width="305" alt="Data Bank - ER Diagram" src="https://user-images.githubusercontent.com/93120413/147638392-cc4690aa-4975-49c8-8159-9ad0fa85a08b.png">

##  Data Sets
### Table 1: Regions
- Just like popular cryptocurrency platforms - Data Bank is also run off a network of nodes where both money and data is stored across the globe. In a traditional banking sense - you can think of these nodes as bank branches or stores that exist around the world.
- This regions table contains the region_id and their respective region_name values

<img width="69" alt="Data Bank - Regions" src="https://user-images.githubusercontent.com/93120413/147638397-74e4495e-b2be-494b-80bb-60c801ce48b2.png">

### Table 2: Customer Nodes
- Customers are randomly distributed across the nodes according to their region - this also specifies exactly which node contains both their cash and data.
- This random distribution changes frequently to reduce the risk of hackers getting into Data Bank’s system and stealing customer’s money and data!
- Below is a sample of the top 10 rows of the data_bank.customer_nodes

<img width="154" alt="Data Bank - Customer Nodes" src="https://user-images.githubusercontent.com/93120413/147638403-1e447a68-4bcc-4ef9-a0f3-ae6d7ea1bef9.png">

### Table 3: Customer Transactions
- This table stores all customer deposits, withdrawals and purchases made using their Data Bank debit card.

<img width="131" alt="Data Bank - Customer Transactions" src="https://user-images.githubusercontent.com/93120413/147638407-d1684d72-53bc-49fd-b9ad-009a2d441290.png">

## Case Study Question and Answer
### A. Customer Nodes Exploration
#### 1. How many unique nodes are there on the Data Bank system?
  ```sql
  WITH CTE AS(
  SELECT DISTINCT
    region_id,
    node_id
  FROM data_bank.customer_nodes
  )
  SELECT
    COUNT(*) AS unique_nodes
  FROM CTE;
  ```
  
#### Result
   |  unique_nodes  |
   |:--------------:|
   |       25       |
   
#### 2. What is the number of nodes per region?
  ```sql
  WITH CTE AS(
  SELECT DISTINCT 
    customer_nodes.region_id,
    customer_nodes.node_id,
    regions.region_name
  FROM data_bank.customer_nodes
  INNER JOIN data_bank.regions
  ON regions.region_id = customer_nodes.region_id
  )
  SELECT 
    region_name,
    COUNT(*) AS node_per_region
  FROM CTE 
  GROUP BY region_name;
  ```
  
#### Result
   |   region_name  | node_per_region |
   |:--------------:|:---------------:| 
   |     America    |       5         |
   |    Australia   |       5         |
   |     Africa     |       5         |
   |      Asia      |       5         |
   |     Europe     |       5         |

#### 3. How many customers are allocated to each region?
  ```sql
  WITH CTE AS(
  SELECT
    DISTINCT regions.region_name AS regionname,
    customer_nodes.customer_id
  FROM data_bank.customer_nodes
  INNER JOIN data_bank.regions
  ON regions.region_id = customer_nodes.region_id
  )
  SELECT 
    regionname,
    COUNT(customer_id) AS customer_alloted
  FROM CTE
  GROUP BY regionname
  ORDER BY regionname;
  ```
  
#### Result
   |   region_name  |  customer_alloted |
   |:--------------:|:-----------------:| 
   |      Africa    |        102        |
   |     America    |        105        |
   |      Asia      |        95         |
   |    Australia   |        110        |
   |     Europe     |        88         | 
   
#### 4. How many days on average are customers reallocated to a different node?
  ```sql 
  DROP TABLE IF EXISTS ranked_customer_nodes;
  CREATE TEMP TABLE ranked_customer_nodes AS
  SELECT
    customer_id,
    node_id,
    region_id,
    DATE_PART('day', AGE(end_date,start_date))::INTEGER AS duration,
    ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY start_date) AS rn
  FROM data_bank.customer_nodes;
  ```  
  ```sql  
  WITH RECURSIVE output_table AS (
  SELECT
    customer_id,
    node_id,
    duration,
    rn,
    1 AS run_id
  FROM ranked_customer_nodes
  WHERE rn = 1
      UNION ALL
  SELECT
    t1.customer_id,
    t2.node_id,
    t2.duration,
    t2.rn,
    -- update run_id if the node_id values do not match
    CASE
      WHEN t1.node_id != t2.node_id THEN t1.run_id + 1
      ELSE t1.run_id
    END AS run_id
  FROM output_table t1
  INNER JOIN ranked_customer_nodes t2
  ON t1.rn + 1 = t2.rn
  AND t1.customer_id = t2.customer_id
  And t2.rn > 1
  ),
  cte_customer_nodes AS (
  SELECT
    customer_id,
    run_id,
    SUM(duration) AS node_duration
  FROM output_table
  GROUP BY customer_id, run_id
  )
  SELECT
    ROUND(avg(node_duration)) AS average_node_duration
  FROM cte_customer_nodes;
  ```
#### Result
   | average_node_duration  |
   |:----------------------:|
   |          17            |
   
#### 5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?
  ```sql
  WITH RECURSIVE output_table AS (
  SELECT
    customer_id,
    node_id,
    duration,
    region_id,
    rn,
    1 AS run_id
  FROM ranked_customer_nodes
  WHERE rn = 1

      UNION ALL

  SELECT
    t1.customer_id,
    t2.node_id,
    t2.duration,
    t2.region_id,
    t2.rn,
    -- update run_id if the node_id values do not match
    CASE
      WHEN t1.node_id != t2.node_id THEN t1.run_id + 1
      ELSE t1.run_id
    END AS run_id
  FROM output_table t1
  INNER JOIN ranked_customer_nodes t2
  ON t1.rn + 1 = t2.rn
  AND t1.customer_id = t2.customer_id
  And t2.rn > 1
  ),
  cte_customer_nodes AS (
  SELECT
    customer_id,
    region_id,
    run_id,
    SUM((duration)) AS node_duration
  FROM output_table
  GROUP BY customer_id, run_id, region_id
  )
  SELECT
    regions.region_name,
    ROUND(PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY node_duration)) AS median_node_duration,
    ROUND(PERCENTILE_CONT(0.8) WITHIN GROUP (ORDER BY node_duration)) AS pct80_node_duration,
    ROUND(PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY node_duration)) AS pct95_node_duration
  FROM cte_customer_nodes
  INNER JOIN data_bank.regions
  ON cte_customer_nodes.region_id = regions.region_id
  GROUP BY regions.region_name, regions.region_id
  ORDER BY regions.region_id;
  ```
  
#### Result
   |   region_name  | median_node_duration | pct80_node_duration |
   |:--------------:|:--------------------:|:-------------------:|
   |    Australia   |         17           |         25          |
   |    America     |         17           |         26          |
   |    Africa      |         17           |         27          |
   |    Asia        |         17           |         26          |
   |    Europe      |         17           |         27          |
  
### B. Customer Transactions
#### 1. What is the unique count and total amount for each transaction type?
  ```sql
  SELECT
    txn_type,
    COUNT(txn_type) AS Distinctcount,
    SUM(txn_amount)
  FROM data_bank.customer_transactions
  GROUP BY txn_type;
  ```

#### Result
   |   txn_type     | distinctcount |     sum     |
   |:--------------:|:-------------:|:-----------:|
   |    purchase    |     1617      |    806537   |
   |   withdrawal   |     1580      |    793003   |
   |    deposit     |     2671      |   1359168   |
   
#### 2. What is the average total historical deposit counts and amounts for all customers?
  ```sql
  WITH CTE AS(    
  SELECT
    customer_id,
    COUNT(*) AS depositcount,
    SUM(txn_amount) AS deposit
  FROM data_bank.customer_transactions
  WHERE txn_type = 'deposit'
  GROUP BY customer_id
  )
  SELECT
    ROUND(avg(depositcount)) AS average,
    ROUND((SUM(deposit) / SUM(depositcount))) AS avg_deposit_amount
  FROM CTE;
  ```
  
#### Result
   |     average    | avg_deposit_amount | 
   |:--------------:|:------------------:|
   |        5       |        509         |
   
#### 3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
  ```sql
  WITH cte AS(
  SELECT
    customer_id,
    DATE_TRUNC('month', txn_date)::DATE AS month,
    SUM(CASE WHEN txn_type = 'deposit' THEN 1 ELSE 0 END) AS depositcount,
    SUM(CASE WHEN txn_type = 'purchase' THEN 1 ELSE 0 END) AS purchasecount,
    SUM(CASE WHEN txn_type = 'withdrawal' THEN 1 ELSE 0 END) AS withdrawalcount
  FROM data_bank.customer_transactions
  GROUP BY customer_id, month
  ORDER BY customer_id
  )
  SELECT
    month,
    COUNT(customer_id)
  FROM cte 
  WHERE depositcount > 1 and (purchasecount >= 1 or withdrawalcount >= 1)
  GROUP BY month
  ORDER BY month;
  ```
  
#### Result
   |     month      |        count       | 
   |:--------------:|:------------------:|
   |   2020-01-01   |        168         |
   |   2020-02-01   |        181         |
   |   2020-03-01   |        192         |
   |   2020-04-01   |         70         |
   
#### 4. What is the closing balance for each customer at the end of the month?
  ```sql
  WITH cte_monthly_balances AS (
  SELECT
    customer_id,
    DATE_TRUNC('mon', txn_date)::DATE AS month,
    SUM(
    CASE
      WHEN txn_type = 'deposit' THEN txn_amount
      ELSE (-txn_amount)
    END
  ) AS balance
  FROM data_bank.customer_transactions
  GROUP BY customer_id, month
  ORDER BY customer_id, month
  ),
  cte_generated_months AS (
  SELECT
    DISTINCT customer_id,
    (
    '2020-01-01'::DATE +
    GENERATE_SERIES(0, 3) * INTERVAL '1 MONTH'
    )::DATE AS month
  FROM data_bank.customer_transactions
  )
  SELECT
    cte_generated_months.customer_id,
    cte_generated_months.month,
    COALESCE(cte_monthly_balances.balance, 0) AS balance_contribution,
    SUM(cte_monthly_balances.balance) OVER (
      PARTITION BY cte_generated_months.customer_id
       ORDER BY cte_generated_months.month
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS ending_balance
  FROM cte_generated_months
  LEFT JOIN cte_monthly_balances
  ON cte_generated_months.month = cte_monthly_balances.month
  AND cte_generated_months.customer_id = cte_monthly_balances.customer_id
  WHERE cte_generated_months.customer_id BETWEEN 1 and 3;
  ```

#### Result
   |  customer_id   |       month      |  balance_contribution |   ending_balance   |
   |:--------------:|:----------------:|:---------------------:|:------------------:|
   |        1       |    2020-01-01    |          312          |        312         |
   |        1       |    2020-02-01    |           0           |        312         |
   |        1       |    2020-03-01    |         -952          |       -640         |
   |        1       |    2020-04-01    |           0           |       -640         |
   |        2       |    2020-01-01    |          549          |        549         |
   |        2       |    2020-02-01    |           0           |        549         |
   |        2       |    2020-03-01    |          61           |        610         |
   |        2       |    2020-04-01    |           0           |        610         |
   |        3       |    2020-01-01    |          144          |        144         |
   |        3       |    2020-02-01    |         -965          |       -821         |
   |        3       |    2020-03-01    |         -401          |      -1222         |
   |        3       |    2020-04-01    |          493          |       -729         |
   
#### 5. Comparing the closing balance of a customer’s first month and the closing balance from their second nth, what percentage of customers: Have a negative first month balance?, Have a positive first month balance?, Increase their opening month’s positive closing balance by more than 5% in the following month?, Reduce their opening month’s positive closing balance by more than 5% in the following month?, Move from a positive balance in the first month to a negative balance in the second month?
  ```sql
  WITH cte_monthly_balances AS (
  SELECT
    customer_id,
    DATE_TRUNC('mon', txn_date)::DATE AS month,
    SUM(
    CASE
      WHEN txn_type = 'deposit' THEN txn_amount
      ELSE (-txn_amount)
    END
    ) AS balance
  FROM data_bank.customer_transactions
  GROUP BY customer_id, month
  ORDER BY customer_id, month
  ),
  cte_generated_months AS (
  SELECT
    customer_id,
    (
    DATE_TRUNC('mon', MIN(txn_date))::DATE +
    GENERATE_SERIES(0, 1) * INTERVAL '1 MONTH'
    )::DATE AS month,
    GENERATE_SERIES(1, 2) AS month_number
  FROM data_bank.customer_transactions
  GROUP BY customer_id
  ),
  cte_monthly_transactions AS (
  SELECT
    cte_generated_months.customer_id,
    cte_generated_months.month,
    cte_generated_months.month_number,
    COALESCE(cte_monthly_balances.balance, 9001) AS transaction_amount
  FROM cte_generated_months
  LEFT JOIN cte_monthly_balances
  ON cte_generated_months.month = cte_monthly_balances.month
  AND cte_generated_months.customer_id = cte_monthly_balances.customer_id
  ),
  cte_monthly_aggregates AS (
  SELECT
    customer_id,
    month_number,
    LAG(transaction_amount) OVER (
      PARTITION BY customer_id
      ORDER BY month
    ) AS previous_month_transaction_amount,
    transaction_amount
  FROM cte_monthly_transactions
  ),
  cte_calculations AS (
  SELECT
    COUNT(DISTINCT customer_id) AS customer_count,
    SUM(CASE WHEN previous_month_transaction_amount > 0 THEN 1 ELSE 0 END) AS positive_first_month,
    SUM(CASE WHEN previous_month_transaction_amount < 0 THEN 1 ELSE 0 END) AS negative_first_month,
    SUM(CASE
      WHEN previous_month_transaction_amount > 0
      AND transaction_amount > 0
      AND transaction_amount > 0.5 * previous_month_transaction_amount
      THEN 1
      ELSE 0
    END
    ) AS increase_count,
    SUM(
    CASE
      WHEN previous_month_transaction_amount > 0
      AND transaction_amount < 0
      AND transaction_amount < -0.05 * previous_month_transaction_amount
      THEN 1
      ELSE 0
    END
    ) AS decrease_count,
    SUM(
    CASE
      WHEN previous_month_transaction_amount > 0
      AND transaction_amount < 0
      AND transaction_amount < -previous_month_transaction_amount
      THEN 1
      ELSE 0 
    END
    ) AS negative_count
  FROM cte_monthly_aggregates
  WHERE previous_month_transaction_amount IS NOT NULL
  )
  SELECT
    ROUND(100 * positive_first_month / customer_count, 2) AS positive_pc,
    ROUND(100 * negative_first_month / customer_count, 2) AS negative_pc,
    ROUND(100 * increase_count / positive_first_month, 2) AS increase_pc,
    ROUND(100 * decrease_count / positive_first_month, 2) AS decrease_pc,
    ROUND(100 * negative_count / positive_first_month, 2) AS negative_balance_pc
  FROM cte_calculations;
  ```

#### Result
   |  positive_pc   |    negative_pc   |  increase_pc |   decrease_pc  | negative_balance_pc |
   |:--------------:|:----------------:|:------------:|:--------------:|:-------------------:| 
   |      68.00     |       31.00      |    37.00     |     49.00      |        33.00        |
   
