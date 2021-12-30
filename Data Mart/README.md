# Case Study 5 - Data Mart
![Data Mart](https://8weeksqlchallenge.com/images/case-study-designs/5.png)
## Entity Relationship Diagram
<img width="252" alt="Data Mart - ER Diagram" src="https://user-images.githubusercontent.com/93120413/147722598-7eed7bb8-9790-4a9b-8d62-7b06f0188e55.png">

##  Data Sets
### Table 1: Example Rows
10 random rows are shown in the table output below from data_mart.weekly_sales:

<img width="436" alt="Data Mart - Example Rows" src="https://user-images.githubusercontent.com/93120413/147722600-d41d2d79-b740-4b65-a7f2-2c4962c47cf3.png">

## Case Study Question and Answer
### A. Data Cleansing Steps
#### 1.In a single query, perform the following operations and generate a new table in the data_mart schema named clean_weekly_sales: 
#### - Convert the week_date to a DATE format
#### - Add a week_number as the second column for each week_date value, for example any value from the 1st of January to 7th of January will be 18th to 14th will be 2 etc
#### - Add a month_number with the calendar month for each week_date value as the 3rd column
#### - Add a calendar_year column as the 4th column containing either 2018, 2019 or 2020 values
#### - Add a new column called age_band after the original segment column using the following mapping on the number inside the segment value.

     |  segment |    age_band  |
     |:--------:|:------------:|
     |    1     | Young Adults |
     |    2     | Middle Aged  |
     |  3 or 4  | 	Retirees   |
     
#### - Add a new demographic column using the following mapping for the first letter in the segment values

     |  segment |    age_band  |
     |:--------:|:------------:|
     |    C     | Couples      |
     |    F     | Families     |   
     
#### - Ensure all null string values with an "unknown" string value in the original segment column as well as the new age_band and demographic columns
#### - Generate a new avg_transaction column as the sales value divided by transactions rounded to 2 decimal places for each record
