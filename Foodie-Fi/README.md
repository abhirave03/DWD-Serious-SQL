# Case Study 3 - Foodie Fi
![Foodie Fi](https://8weeksqlchallenge.com/images/case-study-designs/3.png)
## Entity Relationship Diagram
<img width="389" alt="Foodie Fi - ER Diagram" src="https://user-images.githubusercontent.com/93120413/147627286-254bb9b4-b640-4ab9-ac52-40829d1344d2.png">

##  Data Sets
### Table 1: Plans
- Customers can choose which plans to join Foodie-Fi when they first sign up.
- Basic plan customers have limited access and can only stream their videos and is only available monthly at $9.90.
- Pro plan customers have no watch time limits and are able to download videos for offline viewing. Pro plans start at $19.90 a month or $199 for an annual subscription.
- Customers can sign up to an initial 7 day free trial will automatically continue with the pro monthly subscription plan unless they cancel, downgrade to basic or upgrade to an annual pro plan at any point during the trial.
- When customers cancel their Foodie-Fi service - they will have a churn plan record with a null price but their plan will continue until the end of the billing period.

<img width="117" alt="Foodie Fi - Plans" src="https://user-images.githubusercontent.com/93120413/147627288-d16c26b9-8c0c-4848-9893-ce4554afcfb8.png">

### Table 1: Subscriptions
- Customer subscriptions show the exact date where their specific plan_id starts.
- If customers downgrade from a pro plan or cancel their subscription - the higher plan will remain in place until the period is over - the start_date in the subscriptions table will reflect the date that the actual plan changes.
- When customers upgrade their account from a basic plan to a pro or annual pro plan - the higher plan will take effect straightaway.
- When customers churn - they will keep their access until the end of their current billing period but the start_date will be technically the day they decided to cancel their service.

<img width="92" alt="Foodie Fi - Subscriptions" src="https://user-images.githubusercontent.com/93120413/147627289-79839ae8-0687-4750-8b0e-5a3a5f1872e2.png">

## Case Study Question and Answer
### A. Customer Journey
#### 1. Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.
  ```sql
  SELECT * 
  FROM foodie_fi.subscriptions
  INNER JOIN foodie_fi.plans
  ON plans.plan_id = subscriptions.plan_id
  WHERE customer_id IN (1, 2, 13, 15, 16, 18, 19, 25, 39, 42)
  ORDER BY customer_id;
  ```

#### Result
   | customer_id    | plan_id        | start_date     | plan_name      | price          |
   |:--------------:|:--------------:|:--------------:|:--------------:|:--------------:|
   |      1         |       0        |   2020-08-01   |     trial      |    0.00        |
   |      1         |       0        |   2020-08-08   |  basic monthly |    9.90        |
   |      2         |       0        |   2020-09-20   |     trial      |    0.00        |
   |      2         |       3        |   2020-09-27   |  pro annual    |   199.0        |
   |     13         |       0        |   2020-12-15   |     trial      |    0.00        |
   |     13         |       1        |   2020-12-22   |  basic monthly |    9.90        |
   |     13         |       2        |   2021-03-29   |   pro monthly  |   19.90        |
   |     15         |       0        |   2020-03-17   |     trial      |    0.00        |
   |     15         |       2        |   2020-03-24   |   pro monthly  |   19.90        |
   |     15         |       4        |   2020-04-29   |     churn      |    null        |
   |     16         |       0        |   2020-05-31   |     trial      |    0.00        |
   |     16         |       1        |   2020-06-07   |  basic monthly |    9.90        |
   |     16         |       3        |   2020-10-21   |  pro annual    |   199.0        |
   |     18         |       0        |   2020-07-06   |     trial      |    0.00        |
   |     18         |       2        |   2020-07-13   |  basic monthly |    9.90        |
   |     19         |       0        |   2020-06-22   |     trial      |    0.00        |
   |     19         |       2        |   2020-06-29   |  basic monthly |    9.90        |
   |     19         |       3        |   2020-08-29   |  pro annual    |   199.0        |
   |     25         |       0        |   2020-05-10   |     trial      |    0.00        |
   |     25         |       1        |   2020-05-17   |  basic monthly |    9.90        |
   |     25         |       2        |   2020-06-16   |   pro monthly  |   19.90        |
   |     39         |       0        |   2020-05-28   |     trial      |    0.00        |
   |     39         |       1        |   2020-06-04   |  basic monthly |    9.90        |
   |     39         |       2        |   2020-08-25   |   pro monthly  |   19.90        |
   |     39         |       4        |   2020-09-10   |     churn      |    null        |
   |     42         |       0        |   2020-10-27   |     trial      |    0.00        |
   |     42         |       1        |   2020-11-03   |  basic monthly |    9.90        |
   |     42         |       2        |   2021-04-28   |   pro monthly  |   19.90        |
   

### B. Data Analysis Questions
#### 1. How many customers has Foodie-Fi ever had?
  ```sql
  SELECT
    COUNT(DISTINCT customer_id) AS customercount
  FROM foodie_fi.subscriptions;
  ```
  
#### Result
   |  customercount |
   |:--------------:|
   |      1000      |
   
#### 2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value.
  ```sql
  SELECT 
    DATE_TRUNC('Month', start_date) AS startdate,
    COUNT(plan_id) trial_customer
  FROM foodie_fi.subscriptions
  WHERE plan_id = 0
  GROUP BY startdate
  ORDER BY startdate;
  ```

#### Result
   |    startdate   | trial_customer |
   |:--------------:|:--------------:|
   |   2020-01-01   |       88       |
   |   2020-02-01   |       68       |
   |   2020-03-01   |       94       |
   |   2020-04-01   |       81       |
   |   2020-05-01   |       88       |
   |   2020-06-01   |       79       |
   |   2020-07-01   |       89       |
   |   2020-08-01   |       88       |
   |   2020-09-01   |       87       |
   |   2020-10-01   |       79       |
   |   2020-11-01   |       75       |
   |   2020-12-01   |       84       |
   
#### 3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name   
  ```sql
  WITH CTE AS(
  SELECT 
    subscriptions.customer_id,
    subscriptions.plan_id,
    plans.plan_name,
    EXTRACT('Year'FROM start_date) AS startyear
  FROM foodie_fi.subscriptions
  INNER JOIN foodie_fi.plans
  ON plans.plan_id = subscriptions.plan_id
  )
  SELECT
    plan_id,
    plan_name,
    COUNT(plan_id) AS countforplan
  FROM CTE
  WHERE startyear IN (2021)
  GROUP BY plan_id, plan_name
  ORDER BY plan_id;
  ```
  
#### Result
   |     plan_id    |    plan_name   |  countforplan  |
   |:--------------:|:--------------:|:--------------:|
   |        1       |  basic monthly |        8       |
   |        2       |   pro monthly  |       60       |
   |        3       |   pro annual   |       63       |
   |        4       |     churn      |       71       |

#### 4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
  ```sql
  SELECT
    SUM(CASE WHEN plan_id = 4 THEN 1 ELSE 0 END) AS churn_customers,
    ROUND(100 * SUM(CASE WHEN plan_id = 4 THEN 1 ELSE 0 END) / COUNT(DISTINCT customer_id),1) AS percentage
  FROM foodie_fi.subscriptions;
  ```
  
#### Result
   | churn_customers |   percentage   |
   |:---------------:|:--------------:|
   |       307       |     30.0       |

#### 5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
  ```sql
  WITH ranked_plans AS (
  SELECT
    customer_id,
    plan_id,
    start_date,
    ROW_NUMBER() OVER (
      PARTITION BY customer_id
      ORDER BY plan_id
    ) AS plan_rank
  FROM foodie_fi.subscriptions
  ORDER BY customer_id
  )
  SELECT
    SUM(CASE WHEN plan_id = 4 THEN 1 ELSE 0 END) AS churn_customers,
    ROUND(
      100 * SUM(CASE WHEN plan_id = 4 THEN 1 ELSE 0 END) /
      COUNT(*),1
    ) AS percentage 
  FROM ranked_plans
  WHERE plan_rank = 2;
  ``` 
  
#### Result
   | churn_customers |   percentage   |
   |:---------------:|:--------------:|
   |        92       |      9.0       | 
   
#### 6. What is the number and percentage of customer plans after their initial free trial?
  ```sql
  WITH CTE AS(
  SELECT
    customer_id,
    plan_id,
    start_date,
    ROW_NUMBER() OVER (
      PARTITION BY customer_id
      ORDER BY start_date
    ) AS plan_rank
  FROM foodie_fi.subscriptions
  ORDER BY customer_id
  )
  SELECT
    plans.plan_id,
    plans.plan_name,
    COUNT(*) AS customer_count,
    ROUND(100 * COUNT(*) / SUM(COUNT(*)) OVER ()) AS percentage
  FROM cte
  INNER JOIN foodie_fi.plans
  ON cte.plan_id = plans.plan_id
  WHERE plan_rank = 2
  GROUP BY plans.plan_id, plans.plan_name
  ORDER BY plans.plan_id;
  ```

#### Result
   |     plan_id     |    plan_name   | customer_count  |   percentage   |
   |:---------------:|:--------------:|:---------------:|:--------------:|
   |        1        | basic monthly  |      546        |       55       |
   |        2        |  pro monthly   |      325        |       33       |
   |        3        |   pro annual   |       37        |        4       |
   |        4        |     churn      |       92        |        9       |
   
#### 7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?
  ```sql
  DROP TABLE IF EXISTS table1;
  CREATE TEMP TABLE table1 AS
  WITH CTE AS( 
  SELECT
    customer_id, plan_id,
    MAX(start_date) AS start_date
  FROM foodie_fi.subscriptions
  WHERE start_date <= '2020-12-31'
  GROUP BY customer_id, plan_id
  ORDER BY customer_id
  )
  SELECT
    customer_id,
    plan_id,
    start_date,
    ROW_NUMBER() OVER(PARTITION BY cte.customer_id 
    ORDER BY cte.start_date desc) AS rankover
  FROM cte;
  ```
  ```sql
  WITH cte1 AS(
  SELECT * FROM table1
  WHERE rankover = '1'
  )
  SELECT
    plan_id,
    COUNT(customer_id)
  FROM cte1
  GROUP BY plan_id
  ORDER BY plan_id;
  ```

#### Result
   |     plan_id     |      count     |
   |:---------------:|:--------------:|
   |        0        |       19       |
   |        1        |      224       |
   |        2        |      326       |
   |        3        |      195       |
   |        4        |      236       |
   
#### 8. How many customers have upgraded to an annual plan in 2020?
  ```sql
  WITH CTE AS(
  SELECT
    customer_id,
    plan_id,
    start_date,
    ROW_NUMBER() OVER (
      PARTITION BY customer_id
      ORDER BY start_date
    ) AS plan_rank
  FROM foodie_fi.subscriptions
  WHERE start_date <= '2020-12-31'
  ORDER BY customer_id
  )
  SELECT
    SUM(CASE WHEN plan_id = 3 THEN 1 ELSE 0 END) AS Upgraded
  FROM CTE
  WHERE plan_rank >= 2;
  ```
  
#### Result
   |    upgraded    |
   |:--------------:|
   |      195       |
   
#### 9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?
  ```sql
  WITH cte AS(
  SELECT 
    customer_id,
    start_date
  FROM foodie_fi.subscriptions
  WHERE plan_id = 3
  ),
  cte1 AS(
  SELECT
    customer_id,
    start_date
  FROM foodie_fi.subscriptions
  WHERE plan_id = 0
  )
  SELECT
    ROUND(AVG(DATE_PART('day',cte.start_date::TIMESTAMP - cte1.start_date::TIMESTAMP))) AS averagedays
  FROM cte 
  INNER JOIN cte1
  ON cte.customer_id = cte1.customer_id;
  ```

#### Result
   |   averagedays  |
   |:--------------:|
   |      105       | 
   
#### 10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)
  ```sql
  DROP TABLE IF EXISTS table1;
  CREATE TEMP TABLE table1 AS
  WITH cte AS(
  SELECT 
    customer_id,
    start_date
  FROM foodie_fi.subscriptions
  WHERE plan_id = 3
  ),
  cte1 AS(
  SELECT
    customer_id,
    start_date
  FROM foodie_fi.subscriptions
  WHERE plan_id = 0
  )
  SELECT
    DATE_PART('day',cte.start_date::TIMESTAMP - cte1.start_date::TIMESTAMP) AS duration
  FROM cte 
  INNER JOIN cte1
  ON cte.customer_id = cte1.customer_id;
  ```
  ```sql  
  WITH cte3 AS(
  SELECT
    CASE WHEN duration>=0 and duration<=30 THEN '0 - 30 days'
      WHEN duration>=30 and duration<=60 THEN '30 - 60 days' 
      WHEN duration>=60 and duration<=90 THEN '60 - 90 days' 
      WHEN duration>=90 and duration<=120 THEN '90 - 120 days' 
      WHEN duration>=120 and duration<=150 THEN '120 - 150 days' 
      WHEN duration>=150 and duration<=180 THEN '150 - 180 days' 
      WHEN duration>=180 and duration<=210 THEN '180 - 210 days' 
      WHEN duration>=210 and duration<=240 THEN '210 - 240 days' 
      WHEN duration>=240 and duration<=270 THEN '240 - 270 days' 
      WHEN duration>=270 and duration<=300 THEN '270 - 300 days' 
      WHEN duration>=300 and duration<=330 THEN '300 - 330 days' 
      WHEN duration>=330 and duration<=360 THEN '330 - 360 days'
    END durationrange,
    count(*) AS customers
  FROM table1
  GROUP BY duration
  ORDER BY durationrange
  )
  SELECT
    durationrange,
    SUM(customers) AS customer
  FROM cte3
  GROUP BY durationrange;
  ```
  
#### Result
   |  durationrange  |    customer    |
   |:---------------:|:--------------:|
   |   0 - 30 days   |       49       |
   | 120 - 150 days  |       42       |
   | 150 - 180 days  |       36       |
   | 180 - 210 days  |       26       |
   | 210 - 240 days  |        4       |
   | 240 - 270 days  |        5       |
   | 270 - 300 days  |        1       |
   | 300 - 330 days  |        1       |
   |   30 - 60 days  |       24       |
   | 330 - 360 days  |        1       |
   |   60 - 90 days  |       34       |
   |  90 - 120 days  |       35       |

#### 11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?
  ```sql
  WITH CTE AS(
  SELECT
    customer_id,
    plan_id,
    start_date,
    ROW_NUMBER() OVER(
      PARTITION BY customer_id
      ORDER BY start_date desc
    ) AS plan_rank
  FROM foodie_fi.subscriptions
  WHERE start_date <= '2020-12-31' and plan_id NOT IN('3','4')
  ORDER BY customer_id
  )
  SELECT
    SUM(CASE WHEN plan_id = 1 THEN 1 ELSE 0 END) AS dowgradedcustomer
  FROM CTE 
  WHERE plan_rank = 2;
  ```

#### Result
   | dowgradedcustomer |
   |:-----------------:|
   |        163        | 
   
### C. Challenge Payment Question
#### 1. The Foodie-Fi team wants you to create a new payments table for the year 2020 that includes amounts paid by each customer in the subscriptions table with the following requirements: monthly payments always occur on the same day of month as the original start_date of any monthly paid plan, upgrades from basic to monthly or pro plans are reduced by the current paid amount in that month and start immediately, upgrades from pro monthly to pro annual are paid at the end of the current billing period and also starts at the end of the month period, once a customer churns they will no longer make payments.
  ```sql
  WITH lead_plans AS (
  SELECT
    customer_id,
    plan_id,
    start_date,
    LEAD(plan_id) OVER (
      PARTITION BY customer_id
      ORDER BY start_date
    ) AS lead_plan_id,
    LEAD(start_date) OVER (
      PARTITION BY customer_id
      ORDER BY start_date
    ) AS lead_start_date
  FROM foodie_fi.subscriptions
  WHERE DATE_PART('year', start_date) = 2020
  AND plan_id != 0
  ),
  case_1 AS (
  SELECT
    customer_id,
    plan_id,
    start_date,
    DATE_PART('mon', AGE('2020-12-31'::DATE, start_date))::INTEGER AS month_diff
  FROM lead_plans
  WHERE plan_id IN (1,2,3)
  ),
  case_1_payments AS (
  SELECT
    customer_id,
    plan_id,
    (start_date + GENERATE_SERIES(0, month_diff) * INTERVAL '1 month')::DATE AS start_date
  FROM case_1
  ),
  case_2 AS (
  SELECT
    customer_id,
    plan_id,
    start_date,
    DATE_PART('mon', AGE(lead_start_date - 1, start_date))::INTEGER AS month_diff
  FROM lead_plans
  WHERE lead_plan_id = 4
  ),
  case_2_payments AS (
  SELECT
    customer_id,
    plan_id,
    (start_date + GENERATE_SERIES(0, month_diff) * INTERVAL '1 month')::DATE AS start_date
  FROM case_2
  ),
  case_3 AS (
  SELECT
    customer_id,
    plan_id,
    start_date,
    DATE_PART('mon', AGE(lead_start_date - 1, start_date))::INTEGER AS month_diff
  FROM lead_plans
  WHERE plan_id = 1 AND lead_plan_id IN (2, 3)
  ),
  case_3_payments AS (
  SELECT
    customer_id,
    plan_id,
    (start_date + GENERATE_SERIES(0, month_diff) * INTERVAL '1 month')::DATE AS start_date
  FROM case_3
  ),
  case_4 AS (
  SELECT
    customer_id,
    plan_id,
    start_date,
    DATE_PART('mon', AGE(lead_start_date - 1, start_date))::INTEGER AS month_diff
  FROM lead_plans
  WHERE plan_id = 2 AND lead_plan_id = 3
  ),
  case_4_payments AS (
  SELECT
    customer_id,
    plan_id,
    (start_date + GENERATE_SERIES(0, month_diff) * INTERVAL '1 month')::DATE AS start_date
  FROM case_4
  ),
  case_5_payments AS (
  SELECT
    customer_id,
    plan_id,
    start_date
  FROM lead_plans
  WHERE plan_id = 3
  ),
  union_output AS (
  SELECT * FROM case_1_payments
  ) 
  SELECT
    customer_id AS c_id,
    plans.plan_id,
    plans.plan_name,
    start_date AS payment_date,
    CASE
      WHEN union_output.plan_id IN (2, 3) AND
      LAG(union_output.plan_id) OVER w = 1
      THEN plans.price - 9.90
      ELSE plans.price
    END AS amount,
    RANK() OVER w AS pay_order
  FROM union_output
  INNER JOIN foodie_fi.plans
  ON union_output.plan_id = plans.plan_id
  WHERE customer_id IN (1, 2, 7, 11, 13, 15, 16, 18, 19, 25, 39)
  WINDOW w AS (
    PARTITION BY union_output.plan_id
    ORDER BY start_date
  )
  ORDER BY customer_id, payment_date;
  ```
  
#### Result
| c_id | plan_id|   plan_name    |  payment_date            | amount | pay_order |
|:----:|:------:|:--------------:|:------------------------:|:------:|:---------:|
|1	   |	1     | basic monthly  | 2020-08-08T00:00:00.000Z |	9.9    |  16       |
|1		 |	1     | basic monthly  | 2020-09-08T00:00:00.000Z |	9.9    |	21       |
|1		 |	1     | basic monthly  | 2020-10-08T00:00:00.000Z |	9.9    |  26       |
|1		 |	1     | basic monthly  | 2020-11-08T00:00:00.000Z |	9.9    |	31       |
|1		 |	1     | basic monthly  | 2020-12-08T00:00:00.000Z |	9.9    |  36       |
|2		 |	3     | pro annual     | 2020-09-27T00:00:00.000Z |	199    |  2        |
|2		 |	3     | pro annual     | 2020-10-27T00:00:00.000Z |	199    |  5        |
|2		 |	3     | pro annual     | 2020-11-27T00:00:00.000Z |	199    |  8        |
|2		 |	3     | pro annual     | 2020-12-27T00:00:00.000Z |	199    |  11       |
|7		 |	1     | basic monthly  | 2020-02-12T00:00:00.000Z |	9.9    |	1        |
|7		 |	1     | basic monthly  | 2020-04-12T00:00:00.000Z |	9.9    |	3        |
|7		 |	1     |	basic monthly  | 2020-05-12T00:00:00.000Z |	9.9    |	4        |
|7		 |	2     |	pro monthly    | 2020-05-22T00:00:00.000Z |	19.9   |	3        |     
|7		 |	1     |	basic monthly  | 2020-06-12T00:00:00.000Z |	19.9   |	6        |
|7		 |	1     |	basic monthly  | 2020-07-12T00:00:00.000Z |	9.9    |	12       |
|7		 |	2     |	pro monthly    | 2020-07-22T00:00:00.000Z |	19.9   |	11       |
|7		 |	1     |	basic monthly  | 2020-08-12T00:00:00.000Z |	9.9    |	17       |
|7		 |	2     |	pro monthly    | 2020-08-22T00:00:00.000Z |	19.9   |	16       |
|7		 |	2     |	pro monthly    | 2020-09-22T00:00:00.000Z |	19.9   |	22       |
|7		 |	1     |	basic monthly  | 2020-10-12T00:00:00.000Z |	9.9    |	27       |
|7	 	 |	2     |	pro monthly    | 2020-10-22T00:00:00.000Z |	19.9   |	28       |
|7		 |	1     |	basic monthly  | 2020-11-12T00:00:00.000Z |	9.9    |	32       |
|7		 |	2     |	pro monthly    | 2020-11-22T00:00:00.000Z |	19.9   |	34       |
|7		 |	1     |	basic monthly  | 2020-12-12T00:00:00.000Z |	9.9    |	37       |
|7		 |	2     |	pro monthly    | 2020-12-22T00:00:00.000Z |	19.9   |	40       |
|13		 |	1     |	basic monthly  | 2020-12-22T00:00:00.000Z |	9.9    |	39       |
|15		 |	2     |	pro monthly    | 2020-03-24T00:00:00.000Z |	19.9   |	1        |
|15		 |	2     |	pro monthly    | 2020-04-24T00:00:00.000Z |	19.9   |	2        |
|15		 |	2     |	pro monthly    | 2020-05-24T00:00:00.000Z |	19.9   |	4        |
|15		 |	2     |	pro monthly    | 2020-06-24T00:00:00.000Z |	19.9   |	7        |
|15		 |	2     |	pro monthly    | 2020-07-24T00:00:00.000Z |	19.9   |	12       |
|15		 |	2     |	pro monthly    | 2020-08-24T00:00:00.000Z |	19.9   |	17       |
|15		 |	2     |	pro monthly    | 2020-09-24T00:00:00.000Z |	19.9   |	23       |
|15		 |	2     |	pro monthly    | 2020-10-24T00:00:00.000Z |	19.9   |	29       |
|15		 |	2     |	pro monthly    | 2020-11-24T00:00:00.000Z |	19.9   |	35       |
|15		 |	2     |	pro monthly    | 2020-12-24T00:00:00.000Z |	19.9   |	41       |
|16		 |	1     |	basic monthly  | 2020-06-07T00:00:00.000Z |	9.9    |	7        |
|16		 |	1     |	basic monthly  | 2020-07-07T00:00:00.000Z |	9.9    |	11       |
|16		 |	1     |	basic monthly  | 2020-08-07T00:00:00.000Z |	9.9    |	15       |
|16		 |	1     |	basic monthly  | 2020-09-07T00:00:00.000Z |	9.9    |	20       |
|16		 |	1     |	basic monthly  | 2020-10-07T00:00:00.000Z |	9.9    |	25       |
|16		 |	3     |	pro annual     | 2020-10-21T00:00:00.000Z |	199    |	4        |
|16		 |	1     |	basic monthly  | 2020-11-07T00:00:00.000Z |	9.9    |	30       |
|16		 |	3     |	pro annual     | 2020-11-21T00:00:00.000Z |	199    |	7        |
|16		 |	1     |	basic monthly  | 2020-12-07T00:00:00.000Z |	9.9    |	35       |
|16		 |	3     |	pro annual     | 2020-12-21T00:00:00.000Z |	199    |	10       |
|18		 |	2     |	pro monthly    | 2020-07-13T00:00:00.000Z |	19.9   |	9        |
|18		 |	2     |	pro monthly    | 2020-08-13T00:00:00.000Z |	19.9   |	14       |
|18		 |	2     |	pro monthly    | 2020-09-13T00:00:00.000Z |	19.9   |	20       |
|18		 |	2     |	pro monthly    | 2020-10-13T00:00:00.000Z |	19.9   |	26       |
|18		 |	2     |	pro monthly    | 2020-11-13T00:00:00.000Z |	19.9   |	32       |
|18		 |	2     |	pro monthly    | 2020-12-13T00:00:00.000Z |	19.9   |	38       |
|19		 |	2     |	pro monthly    | 2020-06-29T00:00:00.000Z |	19.9   |	8        |
|19		 |	2     |	pro monthly    | 2020-07-29T00:00:00.000Z |	19.9   |	13       |
|19		 |	2     |	pro monthly    | 2020-08-29T00:00:00.000Z |	19.9   |	19       |
|19		 |	3     |	pro annual     | 2020-08-29T00:00:00.000Z |	199    |	1        |
|19		 |	2     |	pro monthly    | 2020-09-29T00:00:00.000Z |	19.9   |	25       |
|19		 |	3     |	pro annual     | 2020-09-29T00:00:00.000Z |	199    |	3        |
|19		 |	3     |	pro annual     | 2020-10-29T00:00:00.000Z |	199    |	6        |
|19		 |	2     |	pro monthly    | 2020-10-29T00:00:00.000Z |	19.9   |	31       |
|19		 |	2     |	pro monthly    | 2020-11-29T00:00:00.000Z | 19.9   |	37       |
|19		 |	3     |	pro annual     | 2020-11-29T00:00:00.000Z |	199    |	9        |
|19		 |	2     |	pro monthly    | 2020-12-29T00:00:00.000Z |	19.9   |	43       |
|19		 |	3     |	pro annual     | 2020-12-29T00:00:00.000Z |	199    |	12       |
|25		 |	1     |	basic monthly  | 2020-05-17T00:00:00.000Z |	9.9    |	5        |
|25		 |	2     |	pro monthly    | 2020-06-16T00:00:00.000Z |	19.9   |	5        |
|25		 |	1     |	basic monthly  | 2020-06-17T00:00:00.000Z |	9.9    |	9        |
|25		 |	2     |	pro monthly    | 2020-07-16T00:00:00.000Z |	19.9   |	10       |
|25		 |	1     |	basic monthly  | 2020-07-17T00:00:00.000Z |	9.9    |	13       |
|25		 |	2     |	pro monthly    | 2020-08-16T00:00:00.000Z |	19.9   |	15       |
|25		 |	1     |	basic monthly  | 2020-08-17T00:00:00.000Z |	9.9    |	18       |
|25		 |	2     |	pro monthly    | 2020-09-16T00:00:00.000Z |	19.9   |	21       |
|25		 |	1     |	basic monthly  | 2020-09-17T00:00:00.000Z |	9.9    |	23       |
|25		 |	2     |	pro monthly    | 2020-10-16T00:00:00.000Z |	19.9   |	27       |
|25		 |	1     |	basic monthly  | 2020-10-17T00:00:00.000Z |	9.9    |	28       |
|25		 |	2     |	pro monthly    | 2020-11-16T00:00:00.000Z |	19.9   |	33       |
|25		 |	1     |	basic monthly  | 2020-11-17T00:00:00.000Z |	9.9    |	33       |
|25		 |	2     |	pro monthly    | 2020-12-16T00:00:00.000Z |	19.9   |	39       |
|25		 |	1     |	basic monthly  | 2020-12-17T00:00:00.000Z |	9.9    |	38       |
|39		 |	1     |	basic monthly  | 2020-07-04T00:00:00.000Z |	9.9    |	10       |
|39		 |	1     |	basic monthly  | 2020-08-04T00:00:00.000Z |	9.9    |	14       |
|39		 |	2     |	pro monthly    | 2020-08-25T00:00:00.000Z |	19.9   |	18       |
|39		 |	1     |	basic monthly  | 2020-09-04T00:00:00.000Z |	9.9    |	19       |
|39		 |	2     |	pro monthly    | 2020-09-25T00:00:00.000Z |	19.9   |	24       |
|39		 |	1     |	basic monthly  | 2020-10-04T00:00:00.000Z |	9.9    |	24       |
|39		 |	2     |	pro monthly    | 2020-10-25T00:00:00.000Z |	19.9   |	30       |
|39		 |	1     |	basic monthly  | 2020-11-04T00:00:00.000Z |	9.9    |	29       |
|39		 |	2     |	pro monthly    | 2020-11-25T00:00:00.000Z |	19.9   |	36       |
|39		 |	1     |	basic monthly  | 2020-12-04T00:00:00.000Z |	9.9    |	34       |
|39		 |	2     | pro monthly    | 2020-12-25T00:00:00.000Z |	19.9   |	42       |
