# Case Study 6 - Clique Bait
![Clique Bait](https://8weeksqlchallenge.com/images/case-study-designs/6.png)

##  Data Sets
### Table 1: Users
Customers who visit the Clique Bait website are tagged via their cookie_id.

![Clique Bait - Users](https://user-images.githubusercontent.com/93120413/147914457-c5cb920f-9511-43f0-90e8-385994702628.jpg)

### Table 2: Events
Customer visits are logged in this events table at a cookie_id level and the event_type and page_id values can be used to join onto relevant satellite tables to obtain further information about each event. The sequence_number is used to order the events within each visit.

![Clique Bait - Events](https://user-images.githubusercontent.com/93120413/147914473-75dca9ad-dc01-4d13-85ad-a9a1e5f51ccf.jpg)


### Table 3: Event Identifier
The event_identifier table shows the types of events which are captured by Clique Bait’s digital data systems.

![Clique Bait - Event Identifier](https://user-images.githubusercontent.com/93120413/147914481-2e641ea1-6997-427e-b1ef-ba8af3c5d4a2.jpg)

### Table 4: Campaign Identifier
This table shows information for the 3 campaigns that Clique Bait has ran on their website so far in 2020.

![Clique Bait - Campaign Identifier](https://user-images.githubusercontent.com/93120413/147914483-033802c6-5ee9-406b-a890-477c4afd5129.jpg)

### Table 5: Page Hierarchy
This table lists all of the pages on the Clique Bait website which are tagged and have data passing through from user interaction events.

![Clique Bait - Page Hierarchy](https://user-images.githubusercontent.com/93120413/147914484-dac1bf91-7f44-4a1c-9170-48818e67b600.jpg)

# Case Study Question & Answer
### A. Enterprise Relationship Diagram
#### ER Diagram

<img width="958" alt="Clique Bait A" src="https://user-images.githubusercontent.com/93120413/147914490-25f01700-0d51-423a-8b22-cdfc2d5bdf4d.png">

### B. Digital Analysis
#### 1. How many users are there?
  ```sql
  SELECT
    COUNT(DISTINCT user_id) AS users
  FROM clique_bait.users;
  ``` 
#### Result
  |   users   |
  |:---------:|
  |    500    |
  
  
#### 2. How many cookies does each user have on average?
  ```sql
  WITH CTE AS(
  SELECT
    DISTINCT user_id,
    COUNT(cookie_id) AS total
  FROM clique_bait.users
  GROUP BY user_id
  ORDER BY user_id
  )
  SELECT
    ROUND(AVG(total),2) AS averagecookies
  FROM CTE;
  ```  
#### Result
  |   averagecookies  |
  |:-----------------:|
  |       3.56        |
 
 
#### 3. What is the unique number of visits by all users per month?
  ```sql
  WITH CTE AS(
  SELECT
    DISTINCT visit_id,
    DATE_TRUNC('Month', event_time) AS month_start
  FROM clique_bait.events
  )
  SELECT
    month_start,
    COUNT(visit_id) AS uniquevisits
  FROM CTE 
  GROUP BY month_start
  ORDER BY month_start;
  ```
#### Result
  |   month_start  |  uniquevisits  |
  |:--------------:|:--------------:|
  |   2020-01-01   |      876       |
  |   2020-02-01   |     1488       |
  |   2020-03-01   |      916       |
  |   2020-04-01   |      248       |
  |   2020-05-01   |       36       |


#### 4. What is the number of events for each event type?
  ```sql
  SELECT
    event_identifier.event_type,
    event_identifier.event_name,
    COUNT(events.event_type)
  FROM clique_bait.event_identifier
  INNER JOIN clique_bait.events
  ON event_identifier.event_type = events.event_type
  GROUP BY event_identifier.event_name, event_identifier.event_type
  ORDER BY event_identifier.event_type;
  ```  
#### Result
  |   event_type   |   event_name   |      count    |
  |:--------------:|:--------------:|:-------------:|
  |       1        |    Page View   |      20928    |
  |       2        |   Add to Cart  |       8451    |
  |       3        |    Purchase    |       1777    |
  |       4        |  Ad Impression |        876    |
  |       5        |    Ad Click    |        702    |


#### 5. What is the percentage of visits which have a purchase event?
  ```sql
  WITH cte AS (
  SELECT
    COUNT(DISTINCT visit_id) AS totalvisits,
    SUM(CASE WHEN event_type = 3 THEN 1 ELSE 0 END) AS purchase
  FROM clique_bait.events;
  )
  SELECT
    ROUND((100 * purchase / totalvisits),2) AS purchase_percentage
  FROM cte;
  ```
#### Result
  |  purchase_percentage  |
  |:---------------------:|
  |         49.00         |
 
 
#### 6. What is the percentage of visits which view the checkout page but do not have a purchase event?
  ```sql
  WITH cte_visits_with_checkout_and_purchase_flags AS (
  SELECT
    COUNT(visit_id) AS total,
    SUM(CASE WHEN page_id = 12 THEN 1 ELSE 0 END) AS checkout_flag,
    MAX(CASE WHEN event_type = 3 THEN 0 ELSE 1 END) AS purchase_flag
  FROM clique_bait.events
  )
  SELECT
    ROUND(100 * checkout_flag / sum(total), 2) AS checkout_without_purchase_percentage
  FROM cte_visits_with_checkout_and_purchase_flags
  GROUP BY checkout_flag;
  ```
#### Result
  |  checkout_without_purchase_percentage  |
  |:--------------------------------------:|
  |                    6.42                |


#### 7. What are the top 3 pages by number of views?
  ```sql
  SELECT
  page_hierarchy.page_name,
    COUNT(events.visit_id) AS pageview
  FROM clique_bait.page_hierarchy
  INNER JOIN clique_bait.events
  ON page_hierarchy.page_id = events.page_id
  WHERE events.event_type = 1
  GROUP BY page_hierarchy.page_name
  ORDER BY pageview DESC
  LIMIT 3;
  ```
#### Result 
  |    page_name   |   page_view    |
  |:--------------:|:--------------:|
  |  All Products  |      3174      |
  |    Checkout    |      2103      |
  |    Home Page   |      1782      |


#### 8. What is the number of views and cart adds for each product category?
  ```sql
  SELECT
    page_hierarchy.product_category,
    SUM(CASE WHEN events.event_type = 1 THEN 1 ELSE 0 END) AS pageview,
    SUM(CASE WHEN events.event_type = 2 THEN 1 ELSE 0 END) AS cartadds
  FROM clique_bait.page_hierarchy
  INNER JOIN clique_bait.events
  ON page_hierarchy.page_id = events.page_id
  WHERE page_hierarchy.product_category IS NOT NULL
  GROUP BY page_hierarchy.product_category 
  ORDER BY pageview DESC;
  ```
#### Result
  |    category    |    pageview    |    cartadds   |
  |:--------------:|:--------------:|:-------------:|
  |    Shellfish   |      6204      |     3792      |
  |       Fish     |      4633      |     2789      |
  |     Luxury     |      3032      |     1870      |


#### 9. What are the top 3 products by purchases?
   ```sql
  WITH cte_purchase_visits AS (
  SELECT
    visit_id
  FROM clique_bait.events
  WHERE event_type = 3
  )
  SELECT
    page_hierarchy.product_id,
    page_hierarchy.page_name AS product_name,
    SUM(CASE WHEN event_type = 2 THEN 1 ELSE 0 END) AS purchases
  FROM clique_bait.events
  INNER JOIN clique_bait.page_hierarchy
  ON events.page_id = page_hierarchy.page_id
  WHERE EXISTS (
  SELECT *
  FROM cte_purchase_visits
  WHERE events.visit_id = cte_purchase_visits.visit_id
  )
  AND page_hierarchy.product_id IS NOT NULL
  GROUP BY page_hierarchy.product_id, product_name
  ORDER BY page_hierarchy.product_id;
  ```
#### Result
  |   product_id   |  product_name  |    puchases   |
  |:--------------:|:--------------:|:-------------:|
  |       1        |     Salmon     |      711      |
  |       2        |    Kingfish    |      707      |
  |       3        |      Tuna      |      697      |
  |       4        | Russian Caviar |      697      |
  |       5        |  Black Truffle |      707      |
  |       6        |     Abalone    |      699      |
  |       7        |     Lobster    |      754      |
  |       8        |      Crab      |      719      |
  |       9        |     Oyster     |      726      |
  
### C. Product Funnel Analysis
#### 1. Create a new output table which has the following details: How many times was each product viewed?, How many times was each product added to cart? ,How many times was each product added to a cart but not purchased (abandoned)?, How many times was each product purchased?, Additionally, create another table which further aggregates the data for the above points but this time for each product category instead of individual products.
   ```sql
  DROP TABLE IF EXISTS t1;
  CREATE TEMP TABLE t1 AS
  WITH CTE AS(
  SELECT
    events.visit_id,
    page_hierarchy.product_id,
    page_hierarchy.page_name,
    page_hierarchy.product_category,
    SUM(CASE WHEN event_type = 1 THEN 1 ELSE 0 END) AS productviewed,
    SUM(CASE WHEN event_type = 2 THEN 1 ELSE 0 END) AS addedtocart
  FROM clique_bait.events
  INNER JOIN clique_bait.page_hierarchy
  ON events.page_id = page_hierarchy.page_id
  WHERE page_hierarchy.product_category IS NOT NULL
  GROUP BY page_hierarchy.product_id, page_hierarchy.product_category, page_hierarchy.page_name, events.visit_id
  ORDER BY page_hierarchy.product_id
  ),
  cte1 AS (
  SELECT DISTINCT
    visit_id
  FROM clique_bait.events
  WHERE event_type = 3 
  ),
  cte2 AS (
  SELECT
    t1.visit_id,
    t1.product_id,
    t1.page_name,
    t1.product_category,
    t1.productviewed,
    t1.addedtocart,
    CASE WHEN t2.visit_id IS NOT NULL THEN 1 ELSE 0 END as purchase
  FROM cte AS t1
  LEFT JOIN cte1 AS t2
  ON t1.visit_id = t2.visit_id
  )
  SELECT
    product_id,
    page_name AS product,
    product_category,
    SUM(productviewed) AS page_views,
    SUM(addedtocart) AS cart_adds,
    SUM(CASE WHEN addedtocart = 1 AND purchase = 0 THEN 1 ELSE 0 END) AS abandoned,
    SUM(CASE WHEN addedtocart = 1 AND purchase = 1 THEN 1 ELSE 0 END) AS purchases
  FROM cte2
  GROUP BY product_id, product_category, product
  ORDER BY product_id;


  DROP TABLE IF EXISTS t2;
  CREATE TEMP TABLE t2 AS
  SELECT
    product_category,
    SUM(page_views) AS page_views,
    SUM(cart_adds) AS cart_adds,
    SUM(abandoned) AS abandoned,
    SUM(purchases) AS purchases
  FROM t1
  GROUP BY product_category
  ORDER BY product_category;
  ```

#### 1. Which product had the most views, cart adds and purchases?
  ```sql
  SELECT
  * FROM t1 
  ORDER BY page_views DESC;
    
  SELECT
  * FROM t1 
  ORDER BY cart_adds DESC;
    
  SELECT
  * FROM t1 
  ORDER BY purchases DESC;
  ```
  
#### 2. Which product was most likely to be abandoned?
  ```sql
  SELECT 
    product,
    ROUND(100 * abandoned/cart_adds) AS percentage
  FROM t1 
  ORDER BY percentage DESC
  LIMIT 1;
  ```
#### Result
  |      product     |    percentage  |
  |:----------------:|:--------------:|
  |  Russian Caviar  |      26        |

#### 3. Which product had the highest view to purchase percentage?
  ```sql
  SELECT
    product,
    ROUND(100 * purchases/page_views,2) AS percentage
  FROM t1 
  ORDER BY percentage DESC
  LIMIT 1;
  ```
#### Result
  |      product     |    percentage  |
  |:----------------:|:--------------:|
  |      Lobster     |      48.74     |
  
#### 4. What is the average conversion rate from view to cart add?
  ```sql
  SELECT
    AVG(100*cart_adds/page_views) AS avgconversion
  FROM t1;
  ```
#### Result
  |    averageconversion    |
  |:-----------------------:|
  |         60.95           |

#### 5. What is the average conversion rate from cart add to purchase?
  ```sql
  SELECT
    AVG(100*purchases/cart_adds) AS avgconversion
  FROM t1;
  ```
#### Result
  |    averageconversion    |
  |:-----------------------:|
  |         75.92           |
  
### Part D. Campaigns Analysis
#### 1. 
  ```sql
  SELECT
    users.user_id,
    events.visit_id, 
    MIN(events.event_time) AS visit_start_time,
    SUM(CASE WHEN events.event_type = 1 THEN 1 ELSE 0 END) AS page_views,
    SUM(CASE WHEN events.event_type = 2 THEN 1 ELSE 0 END) AS cart_adds,
    SUM(CASE WHEN events.event_type = 3 THEN 1 ELSE 0 END) AS purchases,
    campaign_identifier.campaign_name,
    SUM(CASE WHEN events.event_type = 4 THEN 1 ELSE 0 END) AS impression,
    SUM(CASE WHEN events.event_type = 5 THEN 1 ELSE 0 END) AS click,
    STRING_AGG(
    CASE
      WHEN page_hierarchy.product_id IS NOT NULL AND event_type = 2
      THEN page_hierarchy.page_name
      ELSE NULL END,
      ', ' ORDER BY events.sequence_number
    ) AS cart_products
  FROM clique_bait.events
  INNER JOIN clique_bait.users
  ON events.cookie_id = users.cookie_id
  INNER JOIN clique_bait.page_hierarchy
  ON events.page_id = page_hierarchy.page_id
  LEFT JOIN clique_bait.campaign_identifier
  ON events.event_time BETWEEN campaign_identifier.start_date AND campaign_identifier.end_date
  WHERE users.user_id = 1
  GROUP BY users.user_id, events.visit_id, campaign_identifier.campaign_name
  ORDER BY users.user_id;
  ```
#### Result
  | user_id |  visit_id  |    visit_start_time    |page_views|cart_adds|purchases|          	campaign_name	    	 |impression| click |	                           cart_products				                        	   |
  |:-------:|-----------:|-----------------------:|---------:|--------:|--------:|--------------------------------:|---------:|------:|-----------------------------------------------------------------------------:|
  |    1	  |    02a5d5  |2020-02-26T16:57:26.261Z|    4	   |	  0    | 	  0    |Half Off - Treat Your Shellf(ish)|     0    |	  0   |	    		                 	NULL                                               |
  |    1	  |    0826dc  |2020-02-26T05:58:37.919Z|    1     |	  0    |	  0    |Half Off - Treat Your Shellf(ish)|     0    | 	0   |	    		                	NULL                                               |
  |    1	  |    0fc437  |2020-02-04T17:49:49.603Z|    10    |	  6    |	  1    |Half Off - Treat Your Shellf(ish)|     1    | 	1   |	Tuna, Russian Caviar, Black Truffle, Abalone, Crab, Oyster                   |
  |    1	  |    30b94d  |2020-03-15T13:12:54.024Z|    9     |  	7    |	  1    |Half Off - Treat Your Shellf(ish)|     1    | 	1   |	Salmon, Kingfish, Tuna, Russian Caviar, Abalone, Lobster, Crab               |
  |    1	  |    41355d  |2020-03-25T00:11:17.861Z|    6     |  	1    |	  0    |Half Off - Treat Your Shellf(ish)|     0    | 	0   |	Lobster                                                                      |
  |    1	  |    ccf365  |2020-02-04T19:16:09.183Z|    7     |	  3    |	  1    |Half Off - Treat Your Shellf(ish)|     0    | 	0   |	Lobster, Crab, Oyster                                                        |
  |    1	  |    eaffde  |2020-03-25T20:06:32.343Z|    10    |  	8    |	  1    |Half Off - Treat Your Shellf(ish)|     1    |	  1   |	Salmon, Tuna, Russian Caviar, Black Truffle, Abalone, Lobster, Crab, Oyster  |
  |    1	  |    f7c798  |2020-03-15T02:23:26.313Z|    9	   |    3    |	  1    |Half Off - Treat Your Shellf(ish)|     0    |	  0   |	Russian Caviar, Crab, Oyster                                                 |
