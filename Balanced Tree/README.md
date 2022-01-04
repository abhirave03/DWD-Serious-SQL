# Case Study 7 - Balanced Tree
![Balanced Tree](https://8weeksqlchallenge.com/images/case-study-designs/7.png)

##  Data Sets
### Table 1: Product Details
balanced_tree.product_details includes all information about the entire range that Balanced Clothing sells in their store.

![Balanced Tree - Product Details](https://user-images.githubusercontent.com/93120413/148049077-2e507530-1b6b-4cf9-9b12-dc8a4868d693.jpg)

### Table 2: Product Sales
balanced_tree.sales contains product level information for all the transactions made for Balanced Tree including quantity, price, percentage discount, member status, a transaction ID and also the transaction timestamp.

![Balanced Tree - Product Sales](https://user-images.githubusercontent.com/93120413/148049114-6cfd3214-a0bb-4edc-a58d-31f9810d2af2.jpg)

### Table 3: Product Hierarcy & Product Price
These tables are used only for the bonus question where we will use them to recreate the balanced_tree.product_details table.

![Balanced Tree - Product Hierarcy](https://user-images.githubusercontent.com/93120413/148049121-27be826b-9374-412a-be46-6d1178c93bdb.jpg)


![Balanced Tree - Product Price](https://user-images.githubusercontent.com/93120413/148049128-aa94cfb6-6b4e-4923-9d42-3a399c43e821.jpg)

# Case Study Question and Answer
### A. High Level Sales Analysis
#### 1. What was the total quantity sold for all products?
  ```sql
  SELECT
    product_details.product_name,
    SUM(qty) AS sold
  FROM balanced_tree.sales
  INNER JOIN balanced_tree.product_details
  ON sales.prod_id = product_details.product_id
  GROUP BY product_details.product_name;
  ```  
#### Result
  |	           product_name	       |      sold     |
  |:------------------------------:|:-------------:|
  |Grey Fashion Jacket - Womens    |    	3876     |
  |Navy Oversized Jeans - Womens	 |	    3856     |
  |Pink Fluro Polkadot Socks - Mens|	    3770     |
  |Khaki Suit Jacket - Womens	     |	    3752     |
  |Black Straight Jeans - Womens	 |	    3786     |
  |White Striped Socks - Mens	     |	    3655     |
  |Blue Polo Shirt - Mens		       |	    3819     |
  |Indigo Rain Jacket - Womens	   |	    3757     |
  |Cream Relaxed Jeans - Womens	   |	    3707     |
  |Teal Button Up Shirt - Mens	   |	    3646     |

#### 2. What is the total generated revenue for all products before discounts?
  ```sql
  SELECT
    SUM(qty*price) AS revenuebeforediscount
  FROM balanced_tree.sales;
  ```
#### Result
  |	    revenuebeforediscount      |   
  |:------------------------------:|
  |           1289453              |
  
#### 3. What was the total discount amount for all products?
  ```sql
  SELECT
    ROUND (
      SUM((price * qty) - discount), 2
    ) AS total_discount
  FROM balanced_tree.sales;
  ```
#### Result
  |	    total_discount      |   
  |:-----------------------:|
  |      1106753.00         |
  
### B. Transaction Analysis
#### 1. How many unique transactions were there?
  ```sql
  SELECT
    COUNT(DISTINCT txn_id) AS uniquetransaction
  FROM balanced_tree.sales;
  ```
#### Result
  |	   uniquetransaction    |   
  |:-----------------------:|
  |          2500           | 

#### 2. What is the average unique products purchased in each transaction?
  ```sql
  WITH CTE AS(
  SELECT
    COUNT(*) AS total,
    txn_id
  FROM balanced_tree.sales
  GROUP BY txn_id
  )
  SELECT
    ROUND(AVG(total)) AS avguniqueprod
  FROM CTE;
  ```
#### Result
  |	   avguniqueprod    |   
  |:-------------------:|
  |         6           |
  
#### 3. What are the 25th, 50th and 75th percentile values for the revenue per transaction?
  ```sql
  WITH cte AS (
  SELECT
    txn_id,
    SUM(qty * price) AS revenue
  FROM balanced_tree.sales
  GROUP BY txn_id
  )
  SELECT
    PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY revenue) AS per25,
    PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY revenue) AS per50,
    PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY revenue) AS per75
  FROM cte;
  ```
#### Result
  |	   per25   |    per25   |    per25   |    
  |:----------:|:----------:|:----------:|
  |   375.75   |    509.5   |    647     |
  
#### 4. What is the average discount value per transaction?
  ```sql
  WITH CTE AS(
  SELECT
    txn_id,
    SUM(discount)  AS disc 
  FROM balanced_tree.sales
  GROUP BY txn_id
  )
  SELECT
    AVG(disc) AS avgdiscount
  FROM CTE;
  ```
#### Result
  |	 avgdiscount  |   
  |:-------------:|
  |    73.0800    |
  
#### 5. What is the percentage split of all transactions for members vs non-members?
  ```sql
  WITH CTE AS(
  SELECT
    member,
    txn_id,
    COUNT (*) AS trans
  FROM balanced_tree.sales
  GROUP BY member,txn_id
  ),
  cte1 AS(
  SELECT
    member,
    SUM(trans) AS transaction
  FROM CTE
  GROUP BY member
  )
  SELECT 
    member, 
    transaction,
  ROUND(100 * transaction / sum(transaction) over()) AS percentage
  FROM cte1
  GROUP BY member, transaction;
  ```
#### Result
  |	   member  |   transaction  |  percentage |    
  |:----------:|:--------------:|:-----------:|
  |    false   |      6034      |      40     |
  |    true    |      9061      |      60     |

#### 6. What is the average revenue for member transactions and non-member transactions?
  ```sql
  WITH CTE AS(
  SELECT
    txn_id,
    SUM(CASE WHEN member = 'true' THEN (qty*price) END) AS memberrevenue,
    SUM(CASE WHEN member = 'false' THEN (qty*price) END) AS non_memberrevenue
  FROM balanced_tree.sales
  GROUP BY txn_id
  )
  SELECT
    ROUND(AVG(memberrevenue),2) AS memberrev,
    ROUND(AVG(non_memberrevenue),2) AS non_memberrev
  FROM CTE;
  ```
#### Result
  |	 memberrev  |  non_memberrev |
  |:-----------:|:--------------:|
  |    516.27   |     515.04     | 
  
### C. Product Analysis
#### 1. What are the top 3 products by total revenue before discount?
  ```sql
  SELECT
    product_details.product_id,
    product_details.product_name,
    SUM(sales.qty * sales.price) AS revenue
  FROM balanced_tree.sales
  INNER JOIN balanced_tree.product_details
  ON sales.prod_id = product_details.product_id
  GROUP BY product_details.product_id, product_details.product_name
  ORDER BY revenue DESC
  LIMIT 3;
  ```
#### Result
  |	 product_id |         product_name         |     revenue    |
  |:-----------:|:----------------------------:|:--------------:|
  |   2a2353    |    Blue Polo Shirt - Mens    |      217683    |
  |   9ec847    | Grey Fashion Jacket - Womens |      209304    |
  |   5d267b    |    White Tee Shirt - Mens    |      152000    |

#### 2. What is the total quantity, revenue and discount for each segment?
  ```sql
  SELECT
    product_details.segment_id,
    product_details.segment_name,
    SUM(sales.qty) AS totalqty,
    SUM(sales.qty * sales.price) AS revenue,
    ROUND(SUM((sales.qty * sales.price * sales.discount))/100,2) AS total_discount
  FROM balanced_tree.sales
  INNER JOIN balanced_tree.product_details
  ON sales.prod_id = product_details.product_id
  GROUP BY product_details.segment_id, product_details.segment_name
  ORDER BY revenue DESC;
  ```
#### Result
  |	 segment_id | segment_name |     totalqty   |     revenue    |  total_discount |
  |:-----------:|:------------:|:--------------:|:--------------:|:---------------:|
  |     5       |    Shirt     |      11265     |     406143     |    49594.00     |
  |     4       |    Jacket    |      11385     |     366983     |    44277.00     |
  |     6       |    Socks     |      11217     |     307977     |    37013.00     |
  |     3       |    Jeans     |      11349     |     208350     |    25343.00     |

#### 3. What is the top selling product for each segment?
  ```sql
  WITH CTE AS(
  SELECT
    product_details.segment_id AS segmentid,
    product_details.segment_name AS segmentname,
    product_details.product_name AS productname,
    SUM(sales.qty) AS productqtysales,
    DENSE_RANK() OVER(PARTITION BY product_details.segment_name ORDER BY SUM(sales.qty) DESC) AS ranked
  FROM balanced_tree.product_details
  INNER JOIN balanced_tree.sales
  ON product_details.product_id = sales.prod_id
  GROUP BY segmentid, segmentname, productname
  ORDER BY product_details.segment_name, productqtysales desc
  )
  SELECT
    segmentid,
    segmentname,
    productname,
    productqtysales
  FROM cte
  WHERE ranked = 1;
  ```
#### Result
  |	 segment_id | segment_name |          product_name           |  productqtysales   |
  |:-----------:|:------------:|:-------------------------------:|:------------------:|
  |      4      |   Jacket     |  Grey Fashion Jacket - Womens   |       3876         |
  |      3      |   Jean       |  Navy Oversized Jeans - Womens  |       3856         |
  |      5      |   Shirt      |  Blue Polo Shirt - Mens         |       3819         |
  |      6      |   Socks      |  Navy Solid Socks - Mens        |       3792         |

#### 4. What is the total quantity, revenue and discount for each category?
  ```sql
  SELECT
    product_details.category_id AS categoryid,
    product_details.category_name AS categoryname,
    SUM(sales.qty) AS totalqty,
    SUM(sales.qty*sales.price) AS totalrevenue,
    SUM(sales.qty*sales.price*sales.discount) AS totaldiscount
  FROM balanced_tree.product_details
  INNER JOIN balanced_tree.sales
  ON product_details.product_id = sales.prod_id
  GROUP BY categoryid, categoryname;
  ```
#### Result
  |	 categoryid | categoryname |     totalqty   |  totalrevenue  |  totaldiscount  |
  |:-----------:|:------------:|:--------------:|:--------------:|:---------------:|
  |       2     |     Mens     |      22482     |     714120     |     8660771     |
  |       2     |   Womens     |      22734     |     575333     |     6962143     |
  
#### 5. What is the top selling product for each category?
  ```sql
  WITH cte AS(
  SELECT
    product_details.category_id AS categoryid,
    product_details.category_name AS categoryname,
    product_details.product_name AS productname,
    SUM(sales.qty) AS totalqty,
    DENSE_RANK() OVER(PARTITION BY product_details.category_name ORDER BY SUM(sales.qty) DESC) AS ranked
  FROM balanced_tree.product_details
  INNER JOIN balanced_tree.sales
  ON product_details.product_id = sales.prod_id
  GROUP BY categoryid, categoryname, productname
  )
  SELECT
    categoryid,
    categoryname,
    productname,
    totalqty
  FROM cte 
  WHERE ranked = 1
  ORDER BY categoryid;
  ```  
#### Result
  |	 categoryid |  categoryname |          product_name           |    totalqty     |
  |:-----------:|:-------------:|:-------------------------------:|:---------------:|
  |      1      |    Women      |  Grey Fashion Jacket - Womens   |      3876       |
  |      2      |     Men       |      Blue Polo Shirt - Mens     |      3819       |  
  
#### 6. What is the percentage split of revenue by product for each segment?
  ```sql
  WITH cte AS(
  SELECT
    product_details.segment_id AS segmentid,
    product_details.segment_name AS segmentname,
    product_details.product_name AS productname,
    SUM(sales.qty * sales.price) AS revenue
  FROM balanced_tree.product_details
  INNER JOIN balanced_tree.sales
  ON product_details.product_id = sales.prod_id
  GROUP BY segmentid, segmentname, productname
  )
  SELECT
    segmentid, 
    segmentname, 
    productname,
    revenue,
    ROUND(100 * revenue / SUM(revenue) OVER(PARTITION BY segmentname),2) AS percentage
  FROM cte 
  ORDER BY segmentid;
  ```  
#### Result

  
