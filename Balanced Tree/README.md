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
  | segmentid |  segmentname  |   	      productname          |	revenue  |  percen  |
  |:---------:|:-------------:|:------------------------------:|:---------:|:--------:|
  |    3      |	    Jeans	    |Black Straight Jeans - Womens   |	121152   |  58.15   |
  |    3      |	    Jeans	    |Navy Oversized Jeans - Womens   |	50128	   |	24.06   |
  |    3      |	    Jeans	    |Cream Relaxed Jeans - Womens    |	37070  	 |	17.79   |
  |    4      |     Jacket	  |Indigo Rain Jacket - Womens     |	71383	   |	19.45   |
  |    4      |	    Jacket	  |Grey Fashion Jacket - Womens    |	209304   |	57.03   |
  |    4      |	    Jacket	  |Khaki Suit Jacket - Womens      |	86296	   |	23.51   |
  |    5      |	    Shirt	    |Blue Polo Shirt - Mens          |	217683   |	53.6    |
  |    5      |     Shirt	    |White Tee Shirt - Mens          |	152000   |	37.43   |
  |    5      |	    Shirt	    |Teal Button Up Shirt - Mens     |	36460	   |	8.98    |
  |    6      |   	Socks	    |White Striped Socks - Mens      |	62135	   |  20.18   |
  |    6      |	    Socks	    |Pink Fluro Polkadot Socks - Mens|	109330   |	35.5    |
  |    6      |	    Socks	    |Navy Solid Socks - Mens         |	136512   |	44.33   |

#### 7. What is the percentage split of revenue by segment for each category?
  ```sql
  WITH cte AS(
  SELECT
    product_details.category_id AS categoryid,
    product_details.category_name AS categoryname,
    product_details.segment_id AS segmentid,
    product_details.segment_name AS segmentname,
    SUM(sales.qty * sales.price) AS revenue
  FROM balanced_tree.product_details
  INNER JOIN balanced_tree.sales
  ON product_details.product_id = sales.prod_id
  GROUP BY categoryname, categoryid, segmentid, segmentname
  )
  SELECT
    categoryid,
    categoryname,
    segmentid, 
    segmentname, 
    revenue,
    ROUND(100 * revenue / SUM(revenue) OVER(PARTITION BY categoryname),2) AS percentage
  FROM cte 
  ORDER BY categoryid, percentage DESC;
  ```
#### Result
  | categoryid |  categoryname |   	segmentid    |	segmentname |  revenue  |  percentage  |
  |:----------:|:-------------:|:---------------:|:------------:|:---------:|:------------:|
  |      1     |    Womens     |        4        |   Jacket     |   366983  |     63.79    |
  |      1     |    Womens     |        3        |   Jeans      |   366983  |     63.79    |
  |      2     |    Mens       |        5        |   Shirt      |   366983  |     63.79    |
  |      2     |    Men s      |        6        |   Socks      |   366983  |     63.79    |

#### 8. What is the percentage split of total revenue by category?
  ```sql
  WITH cte AS(
  SELECT
    product_details.category_id AS categoryid,
    product_details.category_name AS categoryname,
    SUM(sales.qty * sales.price) AS revenue
  FROM balanced_tree.product_details
  INNER JOIN balanced_tree.sales
  ON product_details.product_id = sales.prod_id
  GROUP BY categoryname, categoryid
  )
  SELECT
    categoryid,
    categoryname,
    revenue,
    ROUND(100 * revenue / SUM(revenue) OVER(),2) AS percentage
  FROM cte 
  ORDER BY categoryid, percentage DESC;
  ```
#### Result
  | categoryid |  categoryname |  revenue  |  percentage  |
  |:----------:|:-------------:|:---------:|:------------:|
  |      1     |    Womens     |  575333   |    44.62     |
  |      2     |    Mens       |  714120   |    55.38     |

#### 9. What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions)
  ```sql
  WITH CTE AS(
  SELECT
    product_details.product_id AS productid,
    product_details.product_name AS productname,
    COUNT(DISTINCT sales.txn_id) AS producttotal
  FROM balanced_tree.product_details
  INNER JOIN balanced_tree.sales
  ON product_details.product_id = sales.prod_id
  GROUP BY product_details.product_id,
  product_details.product_name 
  ),
  cte1 AS(
  SELECT
    COUNT(DISTINCT txn_id) AS totall
  FROM balanced_tree.sales
  )
  SELECT
    productid,
    productname,
    ROUND((100 * producttotal::NUMERIC/totall),2) AS penetration
  FROM cte 
  CROSS JOIN cte1
  ORDER BY penetration DESC;
  ```
#### Result
  |productid|	      productname	              |  penetration  |
  |:-------:|:-------------------------------:|:-------------:|
  |f084eb   |Navy Solid Socks - Mens	        |   51.24       |
  |9ec847   |Grey Fashion Jacket - Womens     |   51          |
  |c4a632   |Navy Oversized Jeans - Womens    |   50.96       |
  |2a2353   |Blue Polo Shirt - Mens	          |   50.72       |
  |5d267b   |White Tee Shirt - Mens	          |   50.72       |
  |2feb6b   |Pink Fluro Polkadot Socks - Mens |   50.32       |
  |72f5d4   |Indigo Rain Jacket - Womens      |   50          |
  |d5e9a6   |Khaki Suit Jacket - Womens       |   49.88       |
  |e83aa3   |Black Straight Jeans - Womens    |   49.84       |
  |e31d39   |Cream Relaxed Jeans - Womens     |   49.72       |
  |b9a74d   |White Striped Socks - Mens       |   49.72       |
  |c8d436   |Teal Button Up Shirt - Mens      |   49.68       |

#### 10. What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?
  ```sql
  DROP TABLE IF EXISTS temp_product_combos;
  CREATE TEMP TABLE temp_product_combos AS
  WITH RECURSIVE input(product) AS (
  SELECT product_id::TEXT FROM balanced_tree.product_details
  ),
  output_table AS (
  SELECT 
    ARRAY[product] AS combo,
    product,
    1 AS product_counter
  FROM input
  UNION ALL  -- important to remove duplicates!
  SELECT
    ARRAY_APPEND(output_table.combo, input.product),
    input.product,
    product_counter + 1
  FROM output_table
  INNER JOIN input ON input.product > output_table.product
  WHERE output_table.product_counter <= 2
  )
  SELECT * from output_table
  WHERE product_counter = 2;
  WITH cte_transaction_products AS (
  SELECT
    txn_id,
    ARRAY_AGG(prod_id::TEXT ORDER BY prod_id) AS products
  FROM balanced_tree.sales
  GROUP BY txn_id
  ),
  cte_combo_transactions AS (
  SELECT
    txn_id,
    combo,
    products
  FROM cte_transaction_products
  CROSS JOIN temp_product_combos  -- previously created temp table above!
  WHERE combo < products  -- combo is contained in products
  ),
  cte_ranked_combos AS (
  SELECT
    combo,
    COUNT(DISTINCT txn_id) AS transaction_count,
    RANK() OVER (ORDER BY COUNT(DISTINCT txn_id)) AS combo_rank,
    ROW_NUMBER() OVER (ORDER BY COUNT(DISTINCT txn_id)) AS combo_id
  FROM cte_combo_transactions
  GROUP BY combo
  ),
  cte_most_common_combo_product_transactions AS (
  SELECT
    cte_combo_transactions.txn_id,
    cte_ranked_combos.combo_id,
    UNNEST(cte_ranked_combos.combo) AS prod_id
  FROM cte_combo_transactions
  INNER JOIN cte_ranked_combos
  ON cte_combo_transactions.combo = cte_ranked_combos.combo
  )
  SELECT
    product_details.product_id,
    product_details.product_name,
    COUNT(DISTINCT top_combo.txn_id) AS combo_transaction_count,
    SUM(sales.qty) AS quantity,
    SUM(sales.qty * sales.price) AS revenue,
    ROUND(
    SUM(sales.qty * sales.price * sales.discount / 100),
    2
    ) AS discount,
    ROUND(
    SUM(sales.qty * sales.price * (1 - sales.discount / 100)),
    2
    ) AS net_revenue
  FROM balanced_tree.sales
  INNER JOIN cte_most_common_combo_product_transactions AS top_combo
  ON sales.txn_id = top_combo.txn_id
  AND sales.prod_id = top_combo.prod_id
  INNER JOIN balanced_tree.product_details
  ON sales.prod_id = product_details.product_id
  GROUP BY product_details.product_id, product_details.product_name;
  ```
#### Result
  |product_id|	    product_name              |combo_transaction_count| quantity |  revenue  |	discount  |net_revenue|
  |:--------:|:------------------------------:|:---------------------:|:--------:|:---------:|:----------:|:---------:|
  |2a2353    |Blue Polo Shirt - Mens          |	1268		              |	7585     |   432345  |	52231     |	432345    |
  |2feb6b    |Pink Fluro Polkadot Socks - Mens|	1258		              |	7491     |   217239  |	24241     |	217239    |
  |5d267b    |White Tee Shirt - Mens          |	962		                |	6442     |   257680  |	29854     |	257680    |
  |72f5d4    |Indigo Rain Jacket - Womens     |	759		                |	5354     |   101726  |	11282     |	101726    |
  |9ec847    |Grey Fashion Jacket - Womens    |	685		                |	4738     |   255852  |	30479     |	255852    |
  |b9a74d    |White Striped Socks - Mens      |	651		                |	4090     |   69530   |	7568      |	69530     |
  |c4a632    |Navy Oversized Jeans - Womens   |	674		                |	4266     |   55458   |	5932      |	55458     |
  |c8d436    |Teal Button Up Shirt - Mens     |	604		                |	3578     |   35780   |	3723      |	35780     |
  |d5e9a6    |Khaki Suit Jacket - Womens      |	631		                |	3837     |   88251   |	10014     |	88251     |
  |e31d39    |Cream Relaxed Jeans - Womens    |	584		                |	3481     |   34810   |	3635      |	34810     |
  |e83aa3    |Black Straight Jeans - Womens   |	635		                |	3710     |   118720  |	13615     |	118720    |
  |f084eb    |Navy Solid Socks - Mens         |	645		                |	3754     |   135144  |	15134     |	135144    |
  
###  D. Bonus Challenge
#### 1. Use a single SQL query to transform the product_hierarchy and product_prices datasets to the product_details table.
  ```sql
  DROP TABLE IF EXISTS temp_product_details;
  CREATE TEMP TABLE temp_product_details AS
  WITH RECURSIVE output_table
    (id, category_id, segment_id, style_id, category_name, segment_name, style_name)
  AS (

  SELECT
    id,
    id AS category_id,
    NULL::INTEGER AS segment_id,
    NULL::INTEGER AS style_id,
    level_text AS category_name,
    NULL AS segment_name,
    NULL AS style_name
  FROM balanced_tree.product_hierarchy
  WHERE parent_id IS NULL

  UNION ALL

  SELECT
    product_hierarchy.id,
    output_table.category_id,
    CASE
      WHEN output_table.category_id != product_hierarchy.parent_id
      THEN product_hierarchy.id
      ELSE output_table.segment_id
    END AS category_id,
    CASE
      WHEN output_table.segment_id != product_hierarchy.parent_id
      THEN product_hierarchy.id
      ELSE output_table.style_id
    END AS style_id,
    output_table.category_name,
    CASE
      WHEN output_table.category_id != product_hierarchy.parent_id
      THEN product_hierarchy.level_text
      ELSE output_table.segment_name
    END AS segment_name,
    CASE
      WHEN output_table.segment_id != product_hierarchy.parent_id
      THEN product_hierarchy.level_text
      ELSE output_table.style_name
    END AS style_name
  FROM output_table
  INNER JOIN balanced_tree.product_hierarchy
  ON output_table.id = product_hierarchy.parent_id
  AND product_hierarchy.parent_id IS NOT NULL
  )
  SELECT
    product_prices.product_id,
    product_prices.price,
    CONCAT_WS(segment_name, '-', style_name, '-', category_name) AS product_name,
    category_id,
    segment_id,
    style_id,
    category_name,
    segment_name,
    style_name
  FROM output_table
  INNER JOIN balanced_tree.product_prices
  ON output_table.id = product_prices.id;
  ```  
