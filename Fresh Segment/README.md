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
    |_month|_year|		month_year          	|interest_id|composition|index_value|ranking|percentile_ranking|interest_name	              		|interest_summary	                        			      |created_at		             |	last_modified	 |
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

#### 7. 
