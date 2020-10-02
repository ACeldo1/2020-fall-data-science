
# SQL:  Structured Query Language  Exercise

### Getting Started
1. Go to BigQuery UI https://console.cloud.google.com/bigquery
2. Add in the public data sets. 
	3. Click the Add Data icon
	4. Add any dataset
	5. `bigquery-public-data` should become visible and populate in the BigQuery UI. 
3. Add your queries where it says [YOUR QUERY HERE].
4. Make sure you add your query in between the triple tick marks. 
---

For this section of the exercise we will be using the `bigquery-public-data.austin_311.311_service_requests`  table. 

5. Write a query that tells us how many rows are in the table. 
	```
    SELECT
      COUNT(*) AS total_rows
    FROM
      `bigquery-public-data.austin_311.311_service_requests`
	```

7. Write a query that tells us how many _distinct_ values there are in the complaint_description column.
	``` 
    SELECT
      COUNT(DISTINCT complaint_description) AS total_distinct_complaint_values
    FROM
      `bigquery-public-data.austin_311.311_service_requests`
    
	```
  
8. Write a query that counts how many times each owning_department appears in the table and orders them from highest to lowest. 
	``` 
    SELECT
      DISTINCT owning_department, COUNT (owning_department) AS frequency
    FROM
      `bigquery-public-data.austin_311.311_service_requests`
    GROUP BY
      owning_department
    ORDER BY
      frequency DESC
	```

9. Write a query that lists the top 5 complaint_description that appear most and the amount of times they appear in this table. (hint... limit)
	```
    SELECT
      complaint_description,
      COUNT(complaint_description) AS total_complaints
    FROM
      `bigquery-public-data.austin_311.311_service_requests`
    GROUP BY
      complaint_description
    ORDER BY
      total_complaints DESC
    LIMIT
      5;
	  ```
10. Write a query that lists and counts all the complaint_description, just for the where the owning_department is 'Animal Services Office'.
	```
    SELECT
      complaint_description,
      COUNT(complaint_description) AS total_complaints
    FROM
      `bigquery-public-data.austin_311.311_service_requests`
    WHERE
      owning_department = 'Animal Services Office'
    GROUP BY
      complaint_description;
	```

11. Write a query to check if there are any duplicate values in the unique_key column (hint.. There are two was to do this, one is to use a temporary table for the groupby, then filter for values that have more than one count, or, using just one table but including the  `having` function). 
	```
	With a temporary table:
    
    WITH
      T AS (
          SELECT
            unique_key AS uniqueT,
            COUNT(unique_key) AS total_appearances
          FROM
            `bigquery-public-data.austin_311.311_service_requests`
          GROUP BY
            unique_key 
       )

    SELECT
      DISTINCT unique_key
    FROM
      `bigquery-public-data.austin_311.311_service_requests`
    JOIN
      T
    ON
      unique_key = T.uniqueT
    WHERE
      T.total_appearances > 1
      
      
     
     With the HAVING keyword:
     
     SELECT
      DISTINCT unique_key,
      COUNT(unique_key) AS total_appearances
    FROM
      `bigquery-public-data.austin_311.311_service_requests`
    GROUP BY
      unique_key
    HAVING
      unique_key = unique_key
      AND total_appearances > 1
    ORDER BY
      total_appearances DESC
      
      To find what isn't duplicated, we would change the HAVING condition's '=' to '!=' and for the other, change the WHERE condition to find when the value of T.total_appearances is only 1
      
    ```


### For the next question, use the `census_bureau_usa` tables.

1. Write a query that returns each zipcode and their population for 2000 and 2010. 
	```
    WITH
      T2 AS (
      SELECT
        T1.zipcode AS Zip,
        SUM(T1.population) AS Population_2000
      FROM
        `bigquery-public-data.census_bureau_usa.population_by_zip_2000`  AS T1
      GROUP BY
        T1.zipcode )
    SELECT
      T3.zipcode,
      T2.Population_2000,
      SUM(T3.population) AS Population_2010
    FROM
      `bigquery-public-data.census_bureau_usa.population_by_zip_2010` AS T3
    JOIN
      T2
    ON
      T2.Zip = T3.zipcode
    GROUP BY
      T3.zipcode,
      T2.Population_2000
    ORDER BY
      T3.zipcode ASC
	```

### For the next section, use the  `bigquery-public-data.google_political_ads.advertiser_weekly_spend` table.
1. Using the `advertiser_weekly_spend` table, write a query that finds the advertiser_name that spent the most in usd. 
	```
    SELECT
      advertiser_name,
      SUM(spend_usd) AS US_dollars_spent
    FROM
      `bigquery-public-data.google_political_ads.advertiser_weekly_spend`
    GROUP BY
      advertiser_name
    ORDER BY
      US_dollars_spent DESC
    LIMIT
      1
	```
2. Who was the 6th highest spender? (No need to insert query here, just type in the answer.)
	```
    TOM STEYER 2020 is the 6th highest spender.
	```

3. What week_start_date had the highest spend? (No need to insert query here, just type in the answer.)
	```
    2020-02-23 had the highest spent in USD by MIKE BLOOMBERG 2020 INC.
	```

4. Using the `advertiser_weekly_spend` table, write a query that returns the sum of spend by week (using week_start_date) in usd for the month of August only. 
	```
	SELECT
      CAST(week_start_date AS STRING) AS weekStartDate,
      SUM(spend_usd) as US_dollars_spent
    FROM
      `bigquery-public-data.google_political_ads.advertiser_weekly_spend`
    GROUP BY
      weekStartDate
    HAVING
      weekStartDate LIKE '%-08-%' 
    ORDER BY
      weekStartDate DESC
	```
6.  How many ads did the 'TOM STEYER 2020' campaign run? (No need to insert query here, just type in the answer.)
	```
	Based on the table, 'TOM STEYER 2020' had 50 rows/tuples of data, meaning that the advertiser spent money for around 50 weeks. The 'TOM STEYER 2020' campaign also had two advertiser_id values, meaning that they had run 2 ads.
	```
7. Write a query that has, in the US region only, the total spend in usd for each advertiser_name and how many ads they ran. (Hint, you're going to have to join tables for this one). 
	```
	WITH T1 AS (
      SELECT advertiser_name AS advertiserName, SUM(spend_usd) AS US_dollars_spent FROM `bigquery-public-data.google_political_ads.advertiser_weekly_spend` GROUP BY advertiserName
      )
    SELECT
      advertiser_name,
      T1.US_dollars_spent,
      COUNT(DISTINCT advertiser_id) as ads_run
    FROM
      `bigquery-public-data.google_political_ads.advertiser_weekly_spend`
    JOIN 
      T1 ON advertiser_name = T1.advertiserName
    GROUP BY
      advertiser_name, T1.US_dollars_spent
    ORDER BY
      ads_run DESC
      
     I added the ORDER BY keyword and condition so I could see if there any campaigns/groups that had more than 1-2 ads. There weren't many.
	```
8. For each advertiser_name, find the average spend per ad. 
	```
	WITH
      T1 AS (
          SELECT
            advertiser_id AS advertiserID,
            AVG(spend_usd) AS average_usd_spent
          FROM
            `bigquery-public-data.google_political_ads.advertiser_weekly_spend`
          GROUP BY
            advertiserID )
    SELECT
      DISTINCT advertiser_id,
      advertiser_name,
      T1.average_usd_spent
    FROM
      `bigquery-public-data.google_political_ads.advertiser_weekly_spend`
    JOIN
      T1
    ON
      advertiser_id = T1.advertiserID
    ORDER BY
      T1.average_usd_spent DESC
      
      I added the ORDER BY just to give some more organization to the results
	```
10. Which advertiser_name had the lowest average spend per ad that was at least above 0. 
	``` 
	WITH
      T1 AS (
      SELECT
        DISTINCT advertiser_id AS advertiserID,
        AVG(spend_usd) AS average_usd_spent
      FROM
        `bigquery-public-data.google_political_ads.advertiser_weekly_spend`
      GROUP BY
        advertiserID)
    SELECT
      advertiser_name,
      T1.average_usd_spent
    FROM
      `bigquery-public-data.google_political_ads.advertiser_weekly_spend`
    JOIN
      T1
    ON
      advertiser_id = T1.advertiserID
    WHERE
      T1.average_usd_spent > 0
    GROUP BY
      advertiser_name,
      T1.average_usd_spent
    ORDER BY
      T1.average_usd_spent ASC
      
    The advertiser_name that had the lowst non-zero average amount of US dollars spent per ad was BRANDY K. CHAMBERS with ~1.78
	```
## For this next section, use the `new_york_citibike` datasets.

1. Who went on more bike trips, Males or Females?
	```
	SELECT
      gender AS Genders,
      COUNT(gender) AS total_trips
    FROM
      `bigquery-public-data.new_york_citibike.citibike_trips`
    GROUP BY
      Genders
    ORDER BY
      total_trips DESC
    LIMIT
      1
    
    Limit 1 is used for seeing who went on the most trips
	```
2. What was the average, shortest, and longest bike trip taken in minutes?
	```
	SELECT
      (AVG(tripduration)/60) AS AverageTripTime,
      (MIN(tripduration)/60) AS ShortestTrip,
      (MAX(tripduration)/60) AS LongestTrip
    FROM
      `bigquery-public-data.new_york_citibike.citibike_trips`
	```

3. Write a query that, for every station_name, has the amount of trips that started there and the amount of trips that ended there. (Hint, use two temporary tables, one that counts the amount of starts, the other that counts the number of ends, and then join the two.) 
	```
	WITH
      T1 AS (
      SELECT
        name AS startStation,
        COUNT(T1A.starttime) AS total_trips_start
      FROM
        `bigquery-public-data.new_york_citibike.citibike_stations`
      LEFT JOIN
        `bigquery-public-data.new_york_citibike.citibike_trips` AS T1A
      ON
        name = T1A.start_station_name
      GROUP BY
        startStation
      ORDER BY
        total_trips_start ),
      T2 AS (
      SELECT
        name AS endStation,
        COUNT(T2A.stoptime) AS total_trips_end
      FROM
        `bigquery-public-data.new_york_citibike.citibike_stations`
      LEFT JOIN
        `bigquery-public-data.new_york_citibike.citibike_trips` AS T2A
      ON
        name = T2A.end_station_name
      GROUP BY
        endStation
      ORDER BY
        total_trips_end )
    SELECT
      T1.startStation,
      T1.total_trips_start,
      T2.total_trips_end
    FROM
      T1
    JOIN
      T2
    ON
      T1.startStation = T2.endStation
    ORDER BY
      T1.total_trips_start ASC
	```
# The next section is the Google Colab section.  
1. Open up this [this Colab notebook](https://colab.research.google.com/drive/1kHdTtuHTPEaMH32GotVum41YVdeyzQ74?usp=sharing).
2. Save a copy of it in your drive. 
3. Rename your saved version with your initials. 
4. Click the 'Share' button on the top right.  
5. Change the permissions so anyone with link can view. 
6. Copy the link and paste it right below this line. 
	* YOUR LINK:  ________________________________
9. Complete the two questions in the colab notebook file. 
