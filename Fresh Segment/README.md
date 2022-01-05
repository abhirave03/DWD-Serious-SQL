# Case Study 8 - Fresh Segment
![Fresh Segment](https://8weeksqlchallenge.com/images/case-study-designs/8.png)

##  Data Sets
### Table 1: Interest Metrics
This table contains information about aggregated interest metrics for a specific major client of Fresh Segments which makes up a large proportion of their customer base. Each record in this table represents the performance of a specific interest_id based on the client’s customer base interest measured through clicks and interactions with specific targeted advertising content.

![Fresh Segment - Interest Metrics](https://user-images.githubusercontent.com/93120413/148192016-a5d70290-72d8-4399-ac35-688dedd220da.jpg)

### Table 2: Interest Map
This mapping table links the interest_id with their relevant interest information. You will need to join this table onto the previous interest_details table to obtain the interest_name as well as any details about the summary information.

![Fresh Segment - Interest Maps](https://user-images.githubusercontent.com/93120413/148192021-b13d1d38-1e28-42a6-87b1-78eb71c23a3b.jpg)

# Case Study Question and Answer
### A. Data Exploration and Cleansing
#### 1. Update the fresh_segments.interest_metrics table by modifying the month_year column to be a date data type with the start of the month
  ```sql
  UPDATE fresh_segments.interest_metrics
  SET month_year = TO_DATE(month_year, 'MM-YYYY');

  ALTER TABLE fresh_segments.interest_metrics
  ALTER month_year TYPE DATE USING month_year::DATE;
  ```
  
#### 2. What is count of records in the fresh_segments.interest_metrics for each month_year value sorted in chronological order (earliest to latest) with the null values appearing first?
  ```
  SELECT
    month_year,
    COUNT(month_year) AS records
  FROM fresh_segments.interest_metrics
  GROUP BY month_year
  ORDER BY month_year ASC NULLS FIRST;
  ```
#### Result
  |      month_year        |records|
  |:----------------------:|:-----:|
  |			   null            |0      |
  |2018-07-01T00:00:00.000Z|729    |
  |2018-08-01T00:00:00.000Z|767    |
  |2018-09-01T00:00:00.000Z|780    |
  |2018-10-01T00:00:00.000Z|857    |
  |2018-11-01T00:00:00.000Z|928    |
  |2018-12-01T00:00:00.000Z|995    |
  |2019-01-01T00:00:00.000Z|973    |
  |2019-02-01T00:00:00.000Z|1121   |
  |2019-03-01T00:00:00.000Z|1136   |
  |2019-04-01T00:00:00.000Z|1099   |
  |2019-05-01T00:00:00.000Z|857    |
  |2019-06-01T00:00:00.000Z|824    |
  |2019-07-01T00:00:00.000Z|864    |
  |2019-08-01T00:00:00.000Z|1149   |
  
#### 4. How many interest_id values exist in the fresh_segments.interest_metrics table but not in the fresh_segments.interest_map table? What about the other way around?
  ```sql
  SELECT 
    COUNT(DISTINCT interest_id) AS metric_id,
    COUNT(DISTINCT id) AS map_id,
    (COUNT(DISTINCT id) - COUNT(DISTINCT interest_id)) AS not_in_metric
  FROM fresh_segments.interest_metrics
  CROSS JOIN fresh_segments.interest_map;
  ```
#### Result
  |   metric_id   |   map_id    |   not_in_metric   |
  |:-------------:|:-----------:|:-----------------:|
  |     1202      |   1209      |         7         |

#### 5. Summarize the id values in the fresh_segments.interest_map by its total record count in this table
  ```sql
  WITH cte AS(
  SELECT
    id,
    COUNT(id) AS record_count
  FROM fresh_segments.interest_map
  GROUP BY id
  )
  SELECT
    record_count,
    COUNT(id) AS total
  FROM cte 
  GROUP BY record_count;
  ```  
#### Result
  |   record_count   |    total    |  
  |:----------------:|:-----------:|
  |       1          |    1209     |
  
#### 6. What sort of table join should we perform for our analysis and why? Check your logic by checking the rows where interest_id = 21246 in your joined output and include all columns from fresh_segments.interest_metrics and all columns from fresh_segments.interest_map except from the id column.
  ```sql
  SELECT
    interest_metrics._month,
    interest_metrics._year,
    interest_metrics.month_year,
    interest_metrics.interest_id,
    interest_metrics.composition,
    interest_metrics.index_value,
    interest_metrics.ranking,
    interest_metrics.percentile_ranking,
    interest_map.interest_name,
    interest_map.interest_summary,
    interest_map.created_at,
    interest_map.last_modified
  FROM fresh_segments.interest_metrics
  INNER JOIN fresh_segments.interest_map
  ON interest_metrics.interest_id = interest_map.id
  WHERE _month IS NOT NULL and interest_metrics.interest_id = '21246';
  ```
  #### Result
    |_month|_year|		month_year          	|interest_id|composition|index_value|ranking|percentile_ranking|interest_name	              		|interest_summary	                        			      |created_at		            |	last_modified        	 |
    |:----:|:---:|:------------------------:|:---------:|:---------:|:---------:|:-----:|:----------------:|:------------------------------:|:---------------------------------------------------:|:-----------------------:|:----------------------:|
    |7     |2018 |2018-07-01T00:00:00.000Z	|  21246    |	2.26      |0.65       |722    |0.96              |Readers of El Salvadoran Content|People reading news from El Salvadoran media sources.|2018-06-11T17:50:04.000Z	|2018-06-11T17:50:04.000Z|
    |8     |2018 |2018-08-01T00:00:00.000Z	|  21246    |	2.13      |0.59       |765    |0.26              |Readers of El Salvadoran Content|People reading news from El Salvadoran media sources.|2018-06-11T17:50:04.000Z	|2018-06-11T17:50:04.000Z|
    |9     |2018 |2018-09-01T00:00:00.000Z	|  21246    |	2.06      |0.61       |774    |0.77              |Readers of El Salvadoran Content|People reading news from El Salvadoran media sources.|2018-06-11T17:50:04.000Z	|2018-06-11T17:50:04.000Z|
    |10    |2018 |2018-10-01T00:00:00.000Z	|  21246    |	1.74      |0.58       |855    |0.23              |Readers of El Salvadoran Content|People reading news from El Salvadoran media sources.|2018-06-11T17:50:04.000Z	|2018-06-11T17:50:04.000Z|
    |11    |2018 |2018-11-01T00:00:00.000Z	|  21246    |	2.25      |0.78       |908    |2.16              |Readers of El Salvadoran Content|People reading news from El Salvadoran media sources.|2018-06-11T17:50:04.000Z	|2018-06-11T17:50:04.000Z|
    |12    |2018 |2018-12-01T00:00:00.000Z	|  21246    |	1.97      |0.7        |983    |1.21              |Readers of El Salvadoran Content|People reading news from El Salvadoran media sources.|2018-06-11T17:50:04.000Z	|2018-06-11T17:50:04.000Z|
    |1     |2019 |2019-01-01T00:00:00.000Z	|  21246    |	2.05      |0.76       |954    |1.95              |Readers of El Salvadoran Content|People reading news from El Salvadoran media sources.|2018-06-11T17:50:04.000Z	|2018-06-11T17:50:04.000Z|
    |2     |2019 |2019-02-01T00:00:00.000Z	|  21246    |	1.84      |0.68       |1109   |1.07              |Readers of El Salvadoran Content|People reading news from El Salvadoran media sources.|2018-06-11T17:50:04.000Z	|2018-06-11T17:50:04.000Z|
    |3     |2019 |2019-03-01T00:00:00.000Z	|  21246    |	1.75      |0.67       |1123   |1.14              |Readers of El Salvadoran Content|People reading news from El Salvadoran media sources.|2018-06-11T17:50:04.000Z	|2018-06-11T17:50:04.000Z|
    |4     |2019 |2019-04-01T00:00:00.000Z	|  21246    |	1.58	    |0.63       |1092   |0.64              |Readers of El Salvadoran Content|People reading news from El Salvadoran media sources.|2018-06-11T17:50:04.000Z	|2018-06-11T17:50:04.000Z|

#### 7. Are there any records in your joined table where the month_year value is before the created_at value from the fresh_segments.interest_map table? Do you think these values are valid and why?
  ```sql
  WITH cte AS(
  SELECT 
    interest_metrics.*,
    interest_map.interest_name,
    interest_map.interest_summary,
    interest_map.created_at ::DATE,
    interest_map.last_modified,
    (CASE WHEN interest_metrics.month_year < interest_map.created_at THEN 1 ELSE 0 END) AS casee
  FROM fresh_segments.interest_metrics
  INNER JOIN fresh_segments.interest_map
  ON interest_metrics.interest_id = interest_map.id
  WHERE interest_metrics._month IS NOT NULL
  )
  SELECT
    COUNT(*)
  FROM cte
  WHERE casee=1;
  ```
 #### Result
   |   record_count   |
   |:----------------:|
   |      188         |

### B. Interest Analysis
#### 1. Which interests have been present in all month_year dates in our dataset?
  ```sql
  WITH cte AS(
  SELECT
    interest_id,
    COUNT(month_year) AS monthyear
  FROM fresh_segments.interest_metrics
  WHERE interest_id IS NOT NULL
  GROUP BY interest_id
  )
  SELECT
    monthyear,
    COUNT(interest_id) AS interests 
  FROM cte 
  GROUP BY monthyear 
  ORDER BY monthyear DESC;
  ```
#### Result
  |    monthyear   |    interests   |
  |:--------------:|:--------------:|
  |       14       |       480      |
  |       13       |       82       |
  |       12       |       65       |
  |       11       |       94       |
  |       10       |       86       |
  |       9        |       95       |
  |       8        |       67       |
  |       7        |       90       |
  |       6        |       33       |
  |       5        |       38       |
  |       4        |       32       |
  |       3        |       15       |
  |       2        |       12       |
  |       1        |       13       |
  
#### 2. Using this same total_months measure - calculate the cumulative percentage of all records starting at 14 months - which total_months value passes the 90% cumulative percentage value?
  ```sql
  WITH cte AS(
  SELECT
    interest_id,
    COUNT(month_year) AS monthyear
  FROM fresh_segments.interest_metrics
  WHERE interest_id IS NOT NULL
  GROUP BY interest_id
  ),
  cte1 AS(
  SELECT
    monthyear,
    COUNT(interest_id) AS interests 
  FROM cte 
  GROUP BY monthyear 
  )
  SELECT
    monthyear,
    interests,
    ROUND(100 * sum(interests ) OVER (ORDER BY monthyear DESC) / sum(interests ) OVER (),2) AS percentage
  FROM cte1
  GROUP BY monthyear, interests
  ORDER BY monthyear DESC;
  ```
#### Result
  |    monthyear   |    interests   |   percentage    |
  |:--------------:|:--------------:|:---------------:|
  |       14       |       480      |     39.93       |
  |       13       |       82       |     46.76       |
  |       12       |       65       |     52.16       |
  |       11       |       94       |     59.98       |
  |       10       |       86       |     67.14       |
  |       9        |       95       |     75.04       |
  |       8        |       67       |     80.62       |
  |       7        |       90       |     88.10       |
  |       6        |       33       |     90.85       |
  |       5        |       38       |     94.01       |
  |       4        |       32       |     96.67       |
  |       3        |       15       |     97.92       |
  |       2        |       12       |     98.92       |
  |       1        |       13       |    100.00       |

#### 3. If we were to remove all interest_id values which are lower than the total_months value we found in the previous question - how many total data points would we be removing? 
  ```sql
  WITH cte_removed_interests AS (
  SELECT
    interest_id
  FROM fresh_segments.interest_metrics
  WHERE interest_id IS NOT NULL
  GROUP BY interest_id
  HAVING COUNT(DISTINCT month_year) < 6
  )
  SELECT
    COUNT(interest_id) AS removed_rows
  FROM fresh_segments.interest_metrics 
  WHERE EXISTS (
  SELECT 1
  FROM cte_removed_interests
  WHERE interest_metrics.interest_id = cte_removed_interests.interest_id);
  ```
#### Result
   |   removed_rows   |
   |:----------------:|
   |        400       | 
   
#### 4. Does this decision make sense to remove these data points from a business perspective? Use an example where there are all 14 months present to a removed interest example for your arguments - think about what it means to have less months present from a segment perspective. > 5. If we include all of our interests regardless of their counts - how many unique interests are there for each month?
  ```sql
  WITH CTE AS(
  SELECT
    interest_metrics.month_year,
    interest_map.interest_name,
    interest_metrics.composition,
    RANK() OVER(PARTITION BY interest_map.interest_name ORDER BY interest_metrics.composition DESC) AS ranked
  FROM fresh_segments.interest_metrics
  INNER JOIN fresh_segments.interest_map
  ON interest_metrics.interest_id = interest_map.id
  ORDER BY interest_metrics.composition DESC
  ),
  cte1 AS(
  SELECT
    month_year,
    interest_name,
    composition
  FROM CTE 
  WHERE ranked = 1
  ORDER BY composition DESC
  ),
  cte2 AS (
    SELECT
    month_year,
    interest_name,
    composition
  FROM cte
  WHERE ranked = 1
  ORDER BY composition
  LIMIT 10
  ),
  final_output AS (
  SELECT * FROM cte1
    UNION
  SELECT * FROM cte2
  )
  SELECT * FROM final_output
  ORDER BY composition DESC;
  ```

### C. Segment Analysis
#### 1. Using the complete dataset - which are the top 10 and bottom 10 interests which have the largest composition values in any month_year? Only use the maximum composition value for each interest but you must keep the corresponding month_year
  ```sql
  WITH CTE AS(
  SELECT
    interest_metrics.month_year,
    interest_map.interest_name,
    interest_metrics.composition,
    RANK() OVER(PARTITION BY interest_map.interest_name ORDER BY interest_metrics.composition DESC) AS ranked
  FROM fresh_segments.interest_metrics
  INNER JOIN fresh_segments.interest_map
  ON interest_metrics.interest_id = interest_map.id
  ORDER BY interest_metrics.composition DESC
  ),
  cte1 AS(
    SELECT
    month_year,
    interest_name,
    composition
  FROM CTE 
  WHERE ranked = 1
  ORDER BY composition DESC
  ),
  cte2 AS (
    SELECT
    month_year,
    interest_name,
    composition
  FROM cte
  WHERE ranked = 1
  ORDER BY composition
  LIMIT 10
  ),
  final_output AS (
  SELECT * FROM cte1
    UNION
  SELECT * FROM cte2
  )
  SELECT * FROM final_output
  ORDER BY composition DESC;
  ```

#### 2. Which 5 interests had the lowest average ranking value?
  ```sql
  SELECT
    interest_map.interest_name,
    ROUND(AVG(interest_metrics.ranking),1) AS averageinterest
  FROM fresh_segments.interest_metrics
  INNER JOIN fresh_segments.interest_map
  ON interest_metrics.interest_id = interest_map.id
  WHERE interest_metrics.month_year IS NOT NULL
  GROUP BY interest_map.interest_name
  ORDER BY averageinterest asc
  LIMIT 5;
  ```
#### Result
  |         interest_name        |   averageinterest   |
  |:----------------------------:|:-------------------:|
  |Winter Apparel Shoppers       |         1.0         |
  |Fitness Activity Tracker Users|         4.1         |
  |Men's Shoe Shoppers           |         5.9         |
  |Elite Cycling Gear Shoppers   |         7.8         |
  |Shoe Shoppers                 |         9.4         |
  
#### 3. Which 5 interests had the largest standard deviation in their percentile_ranking value?
  ```sql
  DROP TABLE IF EXISTS table1;
  CREATE TEMP TABLE table1 AS
  WITH cte AS(
  SELECT
    interest_metrics.interest_id,
    interest_map.interest_name,
    ROUND(STDDEV(interest_metrics.percentile_ranking::NUMERIC),1) AS stddeviation,
    MAX(interest_metrics.percentile_ranking) AS max_rank,
    MIN(interest_metrics.percentile_ranking) AS min_rank,
    COUNT(*) AS record_count
  FROM fresh_segments.interest_metrics
  INNER JOIN fresh_segments.interest_map
  ON interest_metrics.interest_id = interest_map.id
  WHERE interest_metrics.month_year IS NOT NULL
  GROUP BY interest_map.interest_name, interest_metrics.interest_id
  )
  SELECT * FROM cte 
  WHERE stddeviation IS NOT NULL
  ORDER BY stddeviation DESC
  LIMIT 5;
  ```
#### Result
  |interest_id|		  interest_name	                   |stddeviation|max_rank|min_rank|record_count|
  |:---------:|:------------------------------------:|:----------:|:------:|:------:|:----------:|
  |6260	      |Blockbuster Movie Fans		             |41.3        |60.63   |2.26	  |2           |
  |131	      |Android Fans          		             |30.7        |75.03   |4.84	  |5           |
  |150	      |TV Junkies          		               |30.4        |93.28   |10.01	  |5           |
  |23	        |Techies	          		               |30.2        |86.69   |7.92	  |6           |
  |20764	    |Entertainment Industry Decision Makers|29	        |86.15   |11.23	  |6           |

#### 4. For the 5 interests found in the previous question - what was minimum and maximum percentile_ranking values for each interest and its corresponding year_month value? Can you describe what is happening for these 5 interests?   
  ```sql
  SELECT
    table1.interest_name,
    interest_metrics.month_year,
    interest_metrics.ranking,
    interest_metrics.percentile_ranking,
    interest_metrics.composition,
    interest_map.interest_name
  FROM table1
  INNER JOIN fresh_segments.interest_map
  ON table1.interest_name = interest_map.interest_name
  INNER JOIN fresh_segments.interest_metrics
  ON interest_map.id = interest_metrics.interest_id
  ORDER BY table1.interest_name, interest_metrics.percentile_ranking DESC
  ```
#### Result
  |interest_name				                  |month_year		           |ranking|percentile_ranking|composition|
  |:-------------------------------------:|:----------------------:|:-----:|:----------------:|:---------:|
  |Android Fans 				                  |2018-07-01T00:00:00.000Z|182    |75.03		          |5.09       |
  |Android Fans 				                  |2018-08-01T00:00:00.000Z|684    |10.82		          |1.77       |
  |Android Fans 				                  |2019-02-01T00:00:00.000Z|1058   |5.62		          |1.85       |
  |Android Fans 				                  |2019-08-01T00:00:00.000Z|1092   |4.96		          |1.91       |
  |Android Fans 				                  |2019-03-01T00:00:00.000Z|1081   |4.84		          |1.72       |
  |Blockbuster Movie Fans	             		|2018-07-01T00:00:00.000Z|287    |60.63			        |5.27       |
  |Blockbuster Movie Fans		             	|2019-08-01T00:00:00.000Z|1123   |2.26       		    |1.83       |
  |Entertainment Industry Decision Makers	|2018-07-01T00:00:00.000Z|101    |86.15		          |5.85       |
  |Entertainment Industry Decision Makers	|2019-02-01T00:00:00.000Z|873    |22.12     		    |2.11       |
  |Entertainment Industry Decision Makers	|2018-10-01T00:00:00.000Z|697    |18.67     		    |2.01       |
  |Entertainment Industry Decision Makers	|2018-08-01T00:00:00.000Z|644    |16.04      		    |1.78       |
  |Entertainment Industry Decision Makers	|2019-03-01T00:00:00.000Z|1005   |11.53     		    |1.97       |
  |Entertainment Industry Decision Makers	|2019-08-01T00:00:00.000Z|1020   |11.23	      	    |1.91       |
  |Techies                        				|2018-07-01T00:00:00.000Z|97     |86.69     		    |5.41       |
  |Techies                        				|2018-08-01T00:00:00.000Z|530    |30.9      		    |1.9        |
  |Techies                        				|2018-09-01T00:00:00.000Z|594    |23.85     		    |1.6        |
  |Techies                        				|2019-03-01T00:00:00.000Z|1026   |9.68      		    |1.91       |
  |Techies                        				|2019-02-01T00:00:00.000Z|1015   |9.46      		    |1.89       |
  |Techies                        				|2019-08-01T00:00:00.000Z|1058   |7.92      		    |1.9        |
  |TV Junkies                             |2018-07-01T00:00:00.000Z|49     |93.28	      	    |5.3        |
  |TV Junkies                             |2018-10-01T00:00:00.000Z|430    |49.82	      	    |2.34       |
  |TV Junkies                             |2018-12-01T00:00:00.000Z|619    |37.79	      	    |1.72       |
  |TV Junkies                             |2018-08-01T00:00:00.000Z|481    |37.29		          |1.7        |
  |TV Junkies                             |2019-08-01T00:00:00.000Z|1034   |10.01     		    |1.94       |
  
### D. Index Analysis
#### 1. What is the top 10 interests by the average composition for each month?
  ```sql
  WITH cte AS(
  SELECT
    interest_metrics.month_year,
    interest_map.interest_name,
    ROUND((interest_metrics.composition::NUMERIC / interest_metrics.index_value::NUMERIC),2) AS averagecomposition,
    RANK() OVER(PARTITION BY interest_metrics.month_year ORDER BY interest_metrics.composition / interest_metrics.index_value DESC) AS ranked
  FROM fresh_segments.interest_metrics
  INNER JOIN fresh_segments.interest_map
  ON interest_metrics.interest_id = interest_map.id
  WHERE interest_metrics.month_year IS NOT NULL 
  ORDER BY averagecomposition DESC
  )
  SELECT * FROM cte 
  WHERE ranked<=10
  ORDER BY month_year, ranked
  LIMIT 10;
  ```
#### Result
    |	month_year		          	|	interest_name			                    |averagecomposition|	  ranked	|
    |:-------------------------:|:-------------------------------------:|:----------------:|:----------:|
    |	2018-07-01T00:00:00.000Z	|	Las Vegas Trip Planners		            |	7.36	           |	    1	    |
    |	2018-07-01T00:00:00.000Z	|	Gym Equipment Owners		              |	6.94	           |	    2	    |
    |	2018-07-01T00:00:00.000Z	|	Cosmetics and Beauty Shoppers	        |	6.78	           |	    3	    |
    |	2018-07-01T00:00:00.000Z	|	Luxury Retail Shoppers		            |	6.61	           |	    4	    |
    |	2018-07-01T00:00:00.000Z	|	Furniture Shoppers		                |	6.51	           |	    5	    |
    |	2018-07-01T00:00:00.000Z	|	Asian Food Enthusiasts		            |	6.1	             |	    6	    |
    |	2018-07-01T00:00:00.000Z	|	Recently Retired Individuals	        |	5.72	           |	    7	    |
    |	2018-07-01T00:00:00.000Z	|	Family Adventures Travelers	          |	4.85	           |	    8	    |
    |	2018-07-01T00:00:00.000Z	|	Work Comes First Travelers	          |	4.8	             |	    9	    |
    |	2018-07-01T00:00:00.000Z	|	HDTV Researchers		                  |	4.71	           |	    10	   |
 
 #### 2. For all of these top 10 interests - which interest appears the most often?
  ```sql
  WITH cte AS(
  SELECT
    interest_metrics.month_year,
    interest_map.interest_name,
    ROUND((interest_metrics.composition::NUMERIC / interest_metrics.index_value::NUMERIC),2) AS averagecomposition,
    RANK() OVER(PARTITION BY interest_metrics.month_year ORDER BY interest_metrics.composition / interest_metrics.index_value DESC) AS ranked
  FROM fresh_segments.interest_metrics
  INNER JOIN fresh_segments.interest_map
  ON interest_metrics.interest_id = interest_map.id
  WHERE interest_metrics.month_year IS NOT NULL 
  )
  SELECT 
    interest_name,
    COUNT(*) AS record
  FROM cte 
  WHERE ranked <=10
  GROUP BY interest_name
  ORDER BY record DESC
  LIMIT 3;
  ```
### Result
 |      interest_name     | record |
 |:----------------------:|:------:|
 |Luxury Bedding Shoppers |  10    |
 |Solar Energy Researchers|  10    |
 |Alabama Trip Planners   |  10    |

#### 3. What is the average of the average composition for the top 10 interests for each month?
  ```sql
  WITH cte AS(
  SELECT
    interest_metrics.month_year,
    interest_map.interest_name,
    ROUND((interest_metrics.composition::NUMERIC / interest_metrics.index_value::NUMERIC),2) AS averagecomposition,
    RANK() OVER(PARTITION BY interest_metrics.month_year 
    ORDER BY interest_metrics.composition / interest_metrics.index_value DESC) AS ranked
  FROM fresh_segments.interest_metrics
  INNER JOIN fresh_segments.interest_map
  ON interest_metrics.interest_id = interest_map.id
  WHERE interest_metrics.month_year IS NOT NULL 
  )
  SELECT
    month_year,
    ROUND(AVG(averagecomposition),2) AS average
  FROM cte
  WHERE ranked <=10
  GROUP BY month_year
  ORDER BY month_year;
  ```
#### Result
  |	month_year	              |average|
  |:-------------------------:|:-----:|
  |	2018-07-01T00:00:00.000Z	|	6.04	|
  |	2018-08-01T00:00:00.000Z	|	5.95	|
  |	2018-09-01T00:00:00.000Z	|	6.9	  |
  |	2018-10-01T00:00:00.000Z	|	7.07	|
  |	2018-11-01T00:00:00.000Z	|	6.62	|
  |	2018-12-01T00:00:00.000Z	|	6.65	|
  |	2019-01-01T00:00:00.000Z	|	6.4	  |
  |	2019-02-01T00:00:00.000Z	|	6.58	|
  |	2019-03-01T00:00:00.000Z	|	6.17	|
  |	2019-04-01T00:00:00.000Z	|	5.75	|
  |	2019-05-01T00:00:00.000Z	|	3.54	|
  |	2019-06-01T00:00:00.000Z	|	2.43	|
  |	2019-07-01T00:00:00.000Z	|	2.77	|
  |	2019-08-01T00:00:00.000Z	|	2.63	|

#### 4. What is the 3 month rolling average of the max average composition value from September 2018 to August 2019 and include the previous top ranking interests in the same output shown below.
  ```sql
  WITH cte AS(
  SELECT
    interest_metrics.month_year,
    interest_map.interest_name,
    ROUND((interest_metrics.composition::NUMERIC / interest_metrics.index_value::NUMERIC),2) AS index_composition,
    RANK() OVER(PARTITION BY interest_metrics.month_year 
    ORDER BY interest_metrics.composition / interest_metrics.index_value DESC) AS index_rank
  FROM fresh_segments.interest_metrics
  INNER JOIN fresh_segments.interest_map
  ON interest_metrics.interest_id = interest_map.id
  WHERE interest_metrics.month_year IS NOT NULL
  ),
  final_output AS (
  SELECT
    month_year,
    interest_name,
    index_composition AS max_index_composition,
    LAG(interest_name || ': ' || index_composition) OVER (ORDER BY month_year) AS "1_month_ago",
    LAG((interest_name || ': ' || index_composition), 2) OVER (ORDER BY month_year) AS "2_months_ago"
  FROM cte
  WHERE index_rank = 1 
  )
  SELECT * FROM final_output
  WHERE month_year >= '2018-09-01' and month_year <= '2019-08-01'
  ORDER BY month_year ;
  ```
#### Result
|	        month_year	     |	      interest_name       	|	max_index_composition	|	      1_month_ago	              |	        2_months_ago              |
|:------------------------:|:----------------------------:|:---------------------:|:-------------------------------:|:---------------------------------:|
|	2018-09-01T00:00:00.000Z |	Work Comes First Travelers	|	8.26	                |	Las Vegas Trip Planners: 7.21	  |	Las Vegas Trip Planners: 7.36	    |
|	2018-10-01T00:00:00.000Z |	Work Comes First Travelers	|	9.14	                |	Work Comes First Travelers: 8.26|	Las Vegas Trip Planners: 7.21	    |
|	2018-11-01T00:00:00.000Z |	Work Comes First Travelers	|	8.28	                |	Work Comes First Travelers: 9.14|	Work Comes First Travelers: 8.26	|
|	2018-12-01T00:00:00.000Z |	Work Comes First Travelers	|	8.31	                |	Work Comes First Travelers: 8.28|	Work Comes First Travelers: 9.14	|
|	2019-01-01T00:00:00.000Z |	Work Comes First Travelers	|	7.66	                |	Work Comes First Travelers: 8.31|	Work Comes First Travelers: 8.28	|
|	2019-02-01T00:00:00.000Z |	Work Comes First Travelers	|	7.66	                |	Work Comes First Travelers: 7.66|	Work Comes First Travelers: 8.31	|
|	2019-03-01T00:00:00.000Z |	Alabama Trip Planners	      |	6.54	                |	Work Comes First Travelers: 7.66|	Work Comes First Travelers: 7.66	|
|	2019-04-01T00:00:00.000Z |	Solar Energy Researchers	  |	6.28	                |	Alabama Trip Planners: 6.54   	|	Work Comes First Travelers: 7.66	|
|	2019-05-01T00:00:00.000Z |	Readers of Honduran Content	|	4.41	                |	Solar Energy Researchers: 6.28	|	Alabama Trip Planners: 6.54	      |
|	2019-06-01T00:00:00.000Z |	Las Vegas Trip Planners	    |	2.77	                |Readers of Honduran Content: 4.41|	Solar Energy Researchers: 6.28	  |
|	2019-07-01T00:00:00.000Z |	Las Vegas Trip Planners	    |	2.82	                |	Las Vegas Trip Planners: 2.77	  |	Readers of Honduran Content: 4.41	|
|	2019-08-01T00:00:00.000Z |Cosmetics and Beauty Shoppers	|	2.73	                |	Las Vegas Trip Planners: 2.82	  |	Las Vegas Trip Planners: 2.77   	|

