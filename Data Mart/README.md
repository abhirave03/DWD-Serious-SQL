# Case Study 5 - Data Mart
![Data Mart](https://8weeksqlchallenge.com/images/case-study-designs/5.png)
## Entity Relationship Diagram
<img width="252" alt="Data Mart - ER Diagram" src="https://user-images.githubusercontent.com/93120413/147722598-7eed7bb8-9790-4a9b-8d62-7b06f0188e55.png">

##  Data Sets
### Table 1: Example Rows
10 random rows are shown in the table output below from data_mart.weekly_sales:

<img width="436" alt="Data Mart - Example Rows" src="https://user-images.githubusercontent.com/93120413/147722600-d41d2d79-b740-4b65-a7f2-2c4962c47cf3.png">

## Case Study Question and Answer
### A. Customer Nodes Exploration
#### 1. In a single query, generate a new table in the data_mart schema named clean_weekly_sales:
- Convert the week_date to a DATE format
- Add a week_number as the second column for each week_date value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc
- Add a month_number with the calendar month for each week_date value as the 3rd column
- Add a calendar_year column as the 4th column containing either 2018, 2019 or 2020 values
- Add a new column called age_band after the original segment column using the following mapping on the number inside the segment value
     |  segment  |   age_band   |
     |:---------:|:------------:|
     |    1      | Young Adults |
     |    2      |  Middle Aged |
     |   3 or 4  |   Retirees   |
     
- Add a new demographic column using the following mapping for the first letter in the segment values:
     |  segment  |   age_band   |
     |:---------:|:------------:|
     |    C      |    Couples   |
     |    F      |    Families  |
- Ensure all null string values with an "unknown" string value in the original segment column as well as the new age_band and demographic columns
- Generate a new avg_transaction column as the sales value divided by transactions rounded to 2 decimal places for each record
  ```sql
  DROP TABLE IF EXISTS data_mart;
  CREATE TEMP TABLE data_mart AS
  SELECT
    to_char(to_date(week_date, 'DD/MM/YY'), 'DD-MM-YY') AS week_date,
    DATE_PART('week', TO_DATE(week_date, 'DD/MM/YY')) AS week_number,
    DATE_PART('month', TO_DATE(week_date, 'DD/MM/YY')) AS month_number,
    DATE_PART('year', TO_DATE(week_date, 'DD/MM/YY')) AS calendar_year,
    region,
    platform,
    segment,
    CASE 
      WHEN segment = 'F1' or segment = 'C1' THEN 'Young Adults'
      WHEN segment = 'F2' or segment = 'C2' THEN 'Middle Aged'
      WHEN segment = 'F3' or segment = 'C3' or segment = 'F4' or segment = 'C4' THEN 'Retirees' ELSE 'Unknown' END
    AS segment1,
    CASE 
      WHEN LEFT(segment,1) = 'C' THEN 'Couples' 
      WHEN LEFT(segment,1) = 'F' THEN 'Families' ELSE 'Unknown' END AS Demographic,
    customer_type,
    transactions,
    sales,
    (sales / transactions) AS avg_transaction
  FROM data_mart.weekly_sales
  ORDER BY week_date;
  ```
### B. Data Exploratio
#### 1. What day of the week is used for each week_date value?
  ```sql
  WITH CTE AS(
  SELECT
    DISTINCT
    EXTRACT('DOW' FROM week_date::DATE) AS day
  FROM data_mart
  )
  SELECT
    CASE 
      WHEN day = '0' THEN 'Sunday'
      WHEN day = '1' THEN 'Monday' 
      WHEN day = '2' THEN 'Tueday' 
      WHEN day = '3' THEN 'Wednesday' 
      WHEN day = '4' THEN 'Thurday' 
      WHEN day = '5' THEN 'Friday' 
      WHEN day = '6' THEN 'Saturday' END AS dayoftheweek
  FROM CTE;
  ```

#### Result
  |   weekday  |
  |:----------:|
  |   Monday   |
     
#### 2. What range of week numbers are missing from the dataset?
  ```sql
  WITH all_week_numbers AS (
  SELECT GENERATE_SERIES(1, 52) AS week_number
  )
  SELECT
    week_number
  FROM all_week_numbers AS t1
  WHERE NOT EXISTS (
    SELECT 1
  FROM data_mart AS t2
  WHERE t1.week_number = t2.week_number
  );
  ```

#### Result
  | week_number |
  |:-----------:|
  |      1      |
  |      2      |
  |      3      |
  |      4      |
  |      5      |
  |      6      |
  |      7      |
  |      8      |
  |      9      |
  |      10     |
  |      11     |
  |      12     |
  |      37     |
  |      38     |
  |      39     |
  |      40     |
  |      41     |
  |      42     |
  |      43     |
  |      44     |
  |      45     |
  |      46     |
  |      47     |
  |      48     |
  |      49     |
  |      50     |
  |      51     |
  |      52     |

#### 3. How many total transactions were there for each year in the dataset?
  ```sql
  SELECT
    calendar_year,
    SUM(transactions) AS totaltransactions
  FROM data_mart
  GROUP BY calendar_year
  ORDER BY calendar_year;
  ``` 
  
#### Result
  |   calendar_year   |   totaltransactions   |
  |:-----------------:|:---------------------:|
  |       2018        |       346406460       |
  |       2019        |       365639285       |
  |       2020        |       375813651       |

#### 4. What is the total sales for each region for each month?
  ```sql
  SELECT
    region,
    month_number,
    SUM(sales) AS totalsales
  FROM data_mart
  GROUP BY region, month_number
  ORDER BY month_number, region
  LIMIT 7;
  ```

#### Result
  |      region       |       totalsales      |
  |:-----------------:|:---------------------:|
  |      AFRICA       |       567767480       |
  |       ASIA        |       529770793       |
  |      CANADA       |       144634329       |
  |      EUROPE       |       35337093        |
  |      OCEANIA      |       783282888       |
  |   SOUTH AMERICA   |       71023109        |
  |        USA        |       225353043       |
     
#### 5. What is the total count of transactions for each platform
  ```SQL
  SELECT
    platform,
    SUM(transactions) AS totalcount
  FROM data_mart
  GROUP BY platform;
  ```
  
#### Result
  |      platform     |       totacount     |
  |:-----------------:|:-------------------:|
  |     Shopify       |       5925169       |   
  |     Retail        |      1081934227     | 
     
#### 6. What is the percentage of sales for Retail vs Shopify for each month?
  ```sql
  WITH cte_monthly_platform_sales AS (
  SELECT
    DATE_TRUNC('MONTH', week_date::DATE) AS month,
    platform,
    SUM(sales) AS monthly_sales
  FROM data_mart
  GROUP BY  month, platform
  )
  SELECT
    month,
    ROUND (100 * SUM(CASE WHEN platform = 'Retail' THEN monthly_sales ELSE NULL END) / SUM(monthly_sales)::BIGINT, 2) AS retail_percentage,
    ROUND (100 * SUM(CASE WHEN platform = 'Shopify' THEN monthly_sales ELSE NULL END) / SUM(monthly_sales)::BIGINT, 2) AS shopify_percentage
  FROM cte_monthly_platform_sales
  GROUP BY  _month
  ORDER BY _month;
  ```
  
#### Result
  | month    |retail_percentage|shopify_percentage|
  |:--------:|:---------------:|:----------------:|
  |2018-03-01|	97.92	      |	  2.08         |
  |2018-04-01|	97.93	      |       2.07       |
  |2018-05-01|	97.73	      |       2.27       |
  |2018-06-01|	97.76	      |       2.24       |
  |2018-07-01|	97.75	      |       2.25       |
  |2018-08-01|	97.71	      |       2.29       |
  |2018-09-01|	97.68	      |       2.32       |
  |2019-03-01|	97.71	      |       2.29       |
  |2019-04-01|	97.80	      |       2.20       |
  |2019-05-01|	97.52	      |       2.48       |
  |2019-06-01|	97.42	      |       2.58       |
  |2019-07-01|	97.35	      |       2.65       |
  |2019-08-01|	97.21	      |       2.79       |
  |2019-09-01|	97.09	      |       2.91       |
  |2020-03-01|	97.30	      |       2.70       |
  |2020-04-01|	96.96	      |       3.04       |
  |2020-05-01|	96.71	      |       3.29       |
  |2020-06-01|	96.80	      |       3.20       |
  |2020-07-01|	96.67	      |       3.33       |
  |2020-08-01|	96.51	      |       3.49       |
    
#### 7. What is the amount and percentage of sales by demographic for each year in the dataset?
  ```sql
  SELECT
    calendar_year,
    demographic,
    SUM(sales) AS saless,
    ROUND((100* sum(sales)/sum(sum(sales))OVER (PARTITION BY demographic)),2) AS percentage
  FROM data_mart
  GROUP BY calendar_year, demographic
  ORDER BY calendar_year;
  ```

#### Result
  | calendar_year |   demographic   |      sales       |   percentage    |
  |:-------------:|:---------------:|:----------------:|:---------------:|
  |    2018       |    Unknown      |    5369434106    |      32.86      |
  |    2018       |    Couples      |    3402388688    |      30.38      |
  |    2018       |    Families     |    4125558033    |      31.25      |
  |    2019       |    Families     |    4463918344    |      33.81      |
  |    2019       |    Unknown      |    5532862221    |      33.86      |
  |    2019       |    Couples      |    3749251935    |      33.47      |
  |    2020       |    Families     |    4614338065    |      34.95      |
  |    2020       |    Couples      |    4049566928    |      36.15      |
  |    2020       |    Unknown      |    5436315907    |      33.27      |
     
 #### 8. Which age_band and demographic values contribute the most to Retail sales?
   ```sql
   SELECT
    segment1,
    demographic,
    SUM(sales) AS totalsales,
    ROUND(100* sum(sales)/sum(sum(sales))OVER ()) AS percentage
  FROM data_mart
  WHERE platform = 'Retail'
  GROUP BY segment1, demographic
  ORDER BY totalsales DESC;
  ```
  
#### Result
  |   segment1    |   demographic   |   totalsales     |    percentage   |
  |:-------------:|:---------------:|:----------------:|:---------------:|
  |   Unknown     |    Unknown      |    16067285533   |        41       |
  |   Retirees    |    Families     |    6634686916    |        17       |
  |   Retirees    |    Couples      |    6370580014    |        16       |
  |  Middle Aged  |    Families     |    4354091554    |        11       |
  |  Young Adults |    Couples      |    2602922797    |         7       |
  |  Middle Aged  |    Couples      |    1854160330    |         5       |
  |  Young Adults |    Families     |    1770889293    |         4       |
     
#### 9. Can we use the avg_transaction column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?
  ```sql
  SELECT
    platform,
    calendar_year,
    ROUND(AVG(avg_transaction),2)
  FROM data_mart
  GROUP BY platform, calendar_year
  ORDER BY calendar_year;
  ```

#### Result
  | platform |  calendar_year  |       round      |
  |:--------:|:---------------:|:----------------:|
  |  Retail  |	  2018	    |	  42.41      |
  | Shopify  |	  2018	    |	 187.80      |
  |  Retail  |	  2019	    |	  41.47      |
  | Shopify  |	  2019	    |	 177.07      |
  | Shopify  |	  2020	    |	 174.40      |
  |  Retail  |	  2020	    |	  40.14      |

### C. Before & After Analysis
#### 1. What is the total sales for the 4 weeks before & after 2020-06-15? What is the growth/reduction rate in actual value & percentage of sales?
  ```sql
  WITH CTE AS(
  SELECT
    CASE 
      WHEN week_number BETWEEN 21 AND 24 THEN '2. After'
      WHEN week_number BETWEEN 25 AND 28 THEN '1. Before' END AS periodicdata,
    SUM(sales) AS totalsales,
    SUM(transactions) AS totaltransactions,
    SUM(sales)/SUM(transactions) AS divide
  FROM data_mart
  WHERE week_number BETWEEN 21 and 28
  AND calendar_year = '2020'
  GROUP BY periodicdata
  ORDER BY periodicdata
  ),
  cte1 AS(
  SELECT
    totalsales - LEAD(totalsales) over() AS salesdiff,
    ROUND(100 * (totalsales::NUMERIC / LEAD(totalsales) OVER ()-1),2) AS perchange
  FROM CTE
  )
  SELECT
    salesdiff,
    perchange
  FROM cte1
  WHERE salesdiff IS NOT NULL and perchange IS NOT NULL;
  ```

#### Result
  |     sales_diff    |      percentage     |
  |:-----------------:|:-------------------:|
  |     -26884188     |        -1.15        | 
     
#### 2. What about the entire 12 weeks before and after?
  ```sql
  WITH CTE AS(
  SELECT
    CASE 
      WHEN week_number BETWEEN 13 AND 24 THEN '1.Before'
      WHEN week_number BETWEEN 24 AND 36 THEN '2.After' END AS periodicdata,
    SUM(sales) AS totalsales,
    SUM(transactions) AS totaltransactions,
    SUM(sales)/sum(transactions) AS divide
  FROM data_mart
  WHERE week_number BETWEEN 13 and 36
  AND calendar_year = '2020'
  GROUP BY periodicdata
  ORDER BY periodicdata
  ),
  cte1 AS(
  SELECT
    totalsales - LAG(totalsales) over() AS salesdiff,
    ROUND(100*((totalsales::NUMERIC / LAG(totalsales) OVER ())-1),2) AS perchange
  FROM CTE
  )
  SELECT
    salesdiff,
    perchange
  FROM cte1
  WHERE salesdiff IS NOT NULL and perchange IS NOT NULL;
  ```
  
#### Result
  |     sales_diff    |      percentage     |
  |:-----------------:|:-------------------:|
  |     -152325394     |        -2.14       |  
     
#### 3. How do the sale metrics for these 2 periods before and after compare with the previous years in 2018 and 2019?
  ```sql
  WITH CTE AS(
  SELECT
    calendar_year,
    CASE 
      WHEN week_number BETWEEN 13 AND 24 THEN '1.Before'
      WHEN week_number BETWEEN 24 AND 36 THEN '2.After' END AS periodicdata,
    SUM(sales) AS totalsales,
    SUM(transactions) AS totaltransactions,
    SUM(sales)/sum(transactions) AS divide
  FROM data_mart
  WHERE week_number BETWEEN 13 and 36
  GROUP BY calendar_year, periodicdata
  ORDER BY calendar_year
  ),
  cte1 AS(
  SELECT
    calendar_year,
    periodicdata,
    totalsales - LEAD(totalsales) OVER (PARTITION BY calendar_year ORDER BY periodicdata) AS salesdiff,
    ROUND(100 * ((totalsales::NUMERIC / LEAD(totalsales) OVER (PARTITION BY calendar_year ORDER BY periodicdata)) - 1),2) AS perchange
  FROM CTE
  )
  SELECT
    calendar_year,
    salesdiff,
    perchange
  FROM cte1
  WHERE salesdiff IS NOT NULL and perchange IS NOT NULL;
  ```

#### Result
  | calendar_year |     sales_diff    |    percentage   |
  |:--------------|:-----------------:|:---------------:|
  |     2018      |     -104256193    |      -1.60      | 
  |     2019      |     20740294      |       0.30      | 
  |     2020      |     152325394     |       2.18      |

### D. Bonus Question
#### 1. Which areas of the business have the highest negative impact in sales metrics performance in 2020 for the 12 week before and after period?
  ```sql
  WITH CTE AS(
  SELECT
    region,
    CASE 
      WHEN week_number BETWEEN 13 AND 24 THEN '1. Before'
      WHEN week_number BETWEEN 25 AND 36 THEN '2. After' END AS periodicdata,
    SUM(sales) AS totalsales,
    SUM(transactions) AS totaltransactions,
    SUM(sales)/sum(transactions) AS divide
  FROM data_mart
  WHERE week_number BETWEEN 13 and 36
  AND calendar_year = '2020'
  GROUP BY periodicdata, region
  ORDER BY periodicdata
  ),
  cte1 AS(
  SELECT
    region,
    totalsales,
    totalsales - LAG(totalsales) OVER (PARTITION BY region) AS salesdiff,
    ROUND(100 * ((totalsales::NUMERIC / LAG(totalsales) OVER (PARTITION BY region)) - 1),2) AS perchange
  FROM CTE
  )
  SELECT
    *
  FROM cte1
  WHERE salesdiff IS NOT NULL
  ORDER BY region;
  ```
  
#### Result
  |      region       |       totalsales      |       salesdiff     |       perchange     |
  |:-----------------:|:---------------------:|:-------------------:|:-------------------:|
  |      AFRICA       |       1700390294      |       -9146811      |         -0.54       |
  |       ASIA        |       1583807621      |      -53436845      |         -3.26       |
  |      CANADA       |       418264441       |       -8174013      |         -1.92       |
  |      EUROPE       |       114038959       |        5152392      |          4.73       |
  |      OCEANIA      |       2282795690      |      -71321100      |         -0.54       |
  |   SOUTH AMERICA   |       208452033       |       -4584174      |         -3.03       |
  |        USA        |       666198715       |      -10814843      |         -1.60       | 
     
    
