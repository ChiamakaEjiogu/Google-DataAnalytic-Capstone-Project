# Google Data Analytic Capstone Project (A cyclistic bike-share analysis case study)


## Table of Content

- [Business Task](#business-task)
- [Data Preparation](#data-preparation)
- [Data Processing](#data-processing)
- [Data Transformation and Analysis](#data-transformation-and-analysis)
- [Findings](#findings)
- [Recommendation](#recommendation)


## Business Task
The goal of this project is to study how annual subscribers and customers differ. The project will identify how annual subscribers and customers use cyclistic bikes differently and identify what influence member to choose annual subscription. Thereafter, provide recommendations on how customers can be converted to subscribers.


## Data Preparation
#### Data Source
The dataset is from Divvy bikes.com made available by Motivate International Inc, it contains user’s gender, date of birth, station name, type of user, trip duration of riders. Furthermore, plans and pricing information were utilized in the analysis of our datasets.
###### Orginial Data Source
City of Chicago’s (“City”) system data owned made available by Lyft Bikes and Scooters
#### Data Credibility
In assessing the dataset for the data analysis report, it is determined that while the source is reliable and the originality of the data is validated, it lacks comprehensiveness due to a substantial amount of missing information (gender and birth year) crucial for addressing our inquiry. Additionally, its relevance is compromised as the dataset pertains to the year 2019, potentially limiting its applicability for analysis in 2024.


## Data Processing
The dataset was stored in CSV files for all the quarters in 2019. I decided to use all the dataset for 2019 to ensure I am able to accurately identify patterns and behaviors of users. 
#### Tools Used
1. SQL Server- Cleaning and analzing data
2. Excel- Data visualation
#### Cleaning Processs
The following cleaning steps were performed: 
1. Change the column headers of quarter 2 dataset while importing to ensure consistency in column headers.
2. Check for duplicate using trip_id column
  
```` SQL
SELECT 
		trip_id,
		COUNT(*) AS occurrence_count
FROM  ( 
			SELECT *
			FROM Divvy_Trips_2019_Q1
			UNION ALL
			SELECT *
			FROM Divvy_Trips_2019_Q2
			UNION ALL
			SELECT *
			FROM Divvy_Trips_2019_Q3
			UNION ALL
			SELECT *
			FROM Divvy_Trips_2019_Q4
	) AS Total_Trips
GROUP BY 
    trip_id
HAVING 
    COUNT(*) > 1;
``````````
3. Check that length of the trip_id and birthyear are consistent- No inconsistency found
```sql
SELECT trip_id,
       birthyear,
       gender
FROM  (
        SELECT *
        FROM Divvy_Trips_2019_Q1
        UNION ALL
        SELECT *
        FROM Divvy_Trips_2019_Q2
        UNION ALL
        SELECT *
        FROM Divvy_Trips_2019_Q3
        UNION ALL
        SELECT *
        FROM Divvy_Trips_2019_Q4
      ) AS Total_Trips
WHERE LEN(birthyear) > 4 OR LEN(trip_id) > 8;
```
   
4. Check that the gender column only has female or male and the usertype is only subscriber and customer- No inconsistency found.

````` SQL
SELECT
	usertype,
	gender	
FROM  (
		SELECT *
		FROM Divvy_Trips_2019_Q1
		UNION ALL
		SELECT *
		FROM Divvy_Trips_2019_Q2
		UNION ALL
		SELECT *
		FROM Divvy_Trips_2019_Q3
		UNION ALL
		SELECT *
		FROM Divvy_Trips_2019_Q4
		) AS Total_Trips
WHERE usertype NOT IN ('Customer', 'Subscriber')
       OR
      gender NOT IN ('Male', 'Female');
```````````
5. Check for null in trip duration-- null found
  ```sql
SELECT
	tripduration	
FROM  (
		SELECT *
		FROM Divvy_Trips_2019_Q1
		UNION ALL
		SELECT *
		FROM Divvy_Trips_2019_Q2
		UNION ALL
		SELECT *
		FROM Divvy_Trips_2019_Q3
		UNION ALL
		SELECT *
		FROM Divvy_Trips_2019_Q4
		) AS Total_Trips
WHERE tripduration IS NULL;
```   
6. check if the start time is higher than the end time- Yes, some start time and end time were wrongly captured.
```sql
SELECT
	start_time,
	end_time
FROM  (
		SELECT *
		FROM Divvy_Trips_2019_Q1
		UNION ALL
		SELECT *
		FROM Divvy_Trips_2019_Q2
		UNION ALL
		SELECT *
		FROM Divvy_Trips_2019_Q3
		UNION ALL
		SELECT *
		FROM Divvy_Trips_2019_Q4
		) AS Total_Trips
WHERE start_time > end_time;
```


## Data Transformation and Analysis
To answer the business questions, I will explore the dataset to address the following inquiries:

1. Identify the day of the week with the highest trip count.
2. Analyze the trip count per seasonal period.
3. Perform statistical analysis of trip duration.
4. Determine the top 10 stations for origin and destination by each user group.
5. Classify users based on generation and age group.
6. Group users based on gender.
7. Determine the pass type used by customers.
#### SQL Codes and Results
```sql
--- Calculating the day of the week with the highest trip count by each usertype
WITH Trips_2019_Q1_transformed AS (
    SELECT 
        trip_id,
        DATENAME(dw, end_time) AS Daysoftheweek,
        usertype
    FROM (
		SELECT *
		FROM Divvy_Trips_2019_Q1
		UNION ALL
		SELECT *
		FROM Divvy_Trips_2019_Q2
		UNION ALL
		SELECT *
		FROM Divvy_Trips_2019_Q3
		UNION ALL
		SELECT *
		FROM Divvy_Trips_2019_Q4
		) AS Total_Trips -----Combination of all the trips made in 2019, to allow for comprehensive analysis. Using Union  for vertical join.
)
SELECT 
   COALESCE(Daysoftheweek, 'Grand total') AS Days_of_week,--- To replace null with grand total
    COUNT(*) AS No_trip,
    SUM(CASE WHEN usertype = 'Customer' THEN 1 ELSE 0 END) AS No_trip_byCustomer,
	DENSE_RANK() OVER (ORDER BY SUM(CASE WHEN usertype = 'Customer' THEN 1 ELSE 0 END) DESC) AS Customer_trip_ranking,
    SUM(CASE WHEN usertype = 'Subscriber' THEN 1 ELSE 0 END) AS No_trip_bySubscriber,
	DENSE_RANK() OVER (ORDER BY SUM(CASE WHEN usertype = 'Subscriber' THEN 1 ELSE 0 END) DESC) AS Subscriber_trip_ranking
FROM 
    Trips_2019_Q1_transformed
GROUP BY 
    GROUPING SETS
				(
				Daysoftheweek, 
				usertype
				) 
ORDER BY 
    CASE Daysoftheweek
        WHEN 'Monday' THEN 1
        WHEN 'Tuesday' THEN 2
        WHEN 'Wednesday' THEN 3
        WHEN 'Thursday' THEN 4
        WHEN 'Friday' THEN 5
        WHEN 'Saturday' THEN 6
		WHEN 'Sunday' THEN 7
        ELSE 8
    END;
```
| Days_of_week | No_trip | No_trip_byCustomer | Customer_trip_ranking | No_trip_bySubscriber | Subscriber_trip_ranking |
|--------------|---------|--------------------|-----------------------|-----------------------|-------------------------|
| Monday       | 560717  | 102027             | 5                     | 458690                | 5                       |
| Tuesday      | 585456  | 88559              | 8                     | 496897                | 2                       |
| Wednesday    | 584193  | 89855              | 7                     | 494338                | 3                       |
| Thursday     | 587826  | 101133             | 6                     | 486693                | 4                       |
| Friday       | 577286  | 120553             | 4                     | 456733                | 6                       |
| Saturday     | 494486  | 207404             | 2                     | 287082                | 7                       |
| Sunday       | 428040  | 171106             | 3                     | 256934                | 8                       |
| Grand total  | 880637  | 880637             | 1                     | 0                     | 9                       |
| Grand total  | 2937367 | 0                  | 9                     | 2937367               | 1                       |

```sql
---- Determination bike usage  during seasonal period by customer and subscriber
WITH Divvy_Trips_total_2019 AS (
    SELECT *
    FROM Divvy_Trips_2019_Q1
    UNION ALL
    SELECT *
    FROM Divvy_Trips_2019_Q2
    UNION ALL
    SELECT *
    FROM Divvy_Trips_2019_Q3
    UNION ALL
    SELECT *
    FROM Divvy_Trips_2019_Q4
), Season AS (SELECT 
        CASE 
            WHEN DATENAME(MONTH, end_time) IN ('March', 'April', 'May') THEN 'Spring'
            WHEN DATENAME(MONTH, end_time) IN ('June', 'July', 'August') THEN 'Summer'
            WHEN DATENAME(MONTH, end_time) IN ('September', 'October', 'November') THEN 'Autumn'
            WHEN DATENAME(MONTH, end_time) IN ('December', 'January', 'February') THEN 'Winter'
            ELSE 'Unknown'
        END AS Season,
        usertype
    FROM 
        Divvy_Trips_total_2019 
		)
		SELECT 
		 Season,
		 COUNT(*) as Total_trip_per_Season,
    SUM(CASE WHEN usertype = 'Customer' THEN 1 ELSE 0 END) AS No_trip_byCustomer,
    SUM(CASE WHEN usertype = 'Subscriber' THEN 1 ELSE 0 END) AS No_trip_bySubscriber
	FROM Season
	GROUP BY   
	Season;
```

| Season | Total_trip_per_Season | No_trip_byCustomer | No_trip_bySubscriber |
|--------|------------------------|--------------------|----------------------|
| Winter | 354571                 | 23684              | 330887               |
| Summer | 1622900                | 492717             | 1130183              |
| Spring | 798179                 | 145139             | 653040               |
| Autumn | 1042354                | 219097             | 823257               |


```sql
----- calculation of Max, Min, Avg trip duration by each user cateory 
WITH Total_Trip_2019 AS (
    SELECT *
    FROM Divvy_Trips_2019_Q1
    UNION ALL
    SELECT *
    FROM Divvy_Trips_2019_Q2
    UNION ALL
    SELECT *
    FROM Divvy_Trips_2019_Q3
    UNION ALL
    SELECT *
    FROM Divvy_Trips_2019_Q4
), Trip_duration_Minute AS (
    SELECT
        CAST(ABS(DATEDIFF(SECOND, start_time, end_time)) AS BIGINT) AS New_tripduration,---- Using ABS to correct the dataset where start time is hgher than end time.
        usertype
    FROM 
        Total_Trip_2019
)
SELECT
    usertype,
     CAST(MAX(New_tripduration) / 60 AS DECIMAL(18, 0)) AS Maximum_trip_in_minutes,
    CAST(MIN(New_tripduration) / 60 AS DECIMAL(18, 0)) AS Minimum_trip_in_minutes,
    CAST(AVG(New_tripduration) / 60 AS DECIMAL(18, 0)) AS Average_trip_in_minuteses
FROM 
    Trip_duration_Minute
GROUP BY 
    usertype;
```
| usertype   | Maximum_trip_in_minutes | Minimum_trip_in_minutes | Average_trip_in_minuteses |
|------------|--------------------------|-------------------------|---------------------------|
| Customer   | 177200                   | 1                       | 57                        |
| Subscriber | 150943                   | 1                       | 14                        |


```sql
---- Top 10 From-stations based on the number of trips made by customers and subscribers
WITH Total_Trip_2019 AS (
    SELECT *
    FROM (
        SELECT *
        FROM Divvy_Trips_2019_Q1
        UNION ALL
        SELECT *
        FROM Divvy_Trips_2019_Q2
        UNION ALL
        SELECT *
        FROM Divvy_Trips_2019_Q3
        UNION ALL
        SELECT *
        FROM Divvy_Trips_2019_Q4
    ) AS All_Trips
), Station_Counts AS (
    SELECT
        from_station_id,
        MAX(from_station_name) AS station_name,----Dummy aggregate
        SUM(CASE WHEN usertype = 'Customer' THEN 1 ELSE 0 END) AS Customer_trip_count,
        DENSE_RANK() OVER (ORDER BY SUM(CASE WHEN usertype = 'Customer' THEN 1 ELSE 0 END) DESC) AS Customer_trip_ranking,
        SUM(CASE WHEN usertype = 'Subscriber' THEN 1 ELSE 0 END) AS Subscriber_trip_count,
        DENSE_RANK() OVER (ORDER BY SUM(CASE WHEN usertype = 'Subscriber' THEN 1 ELSE 0 END) DESC) AS Subscriber_trip_ranking
    FROM Total_Trip_2019
    GROUP BY from_station_id---- Using station ID instead of station name to eliminate likely duplication and error in station name.
)
SELECT *
FROM Station_Counts
WHERE Customer_trip_ranking <= 10 OR Subscriber_trip_ranking <= 10;----To filter result to top 10 station for each user type.
```
| from_station_id | station_name                 | Customer_trip_count | Customer_trip_ranking | Subscriber_trip_count | Subscriber_trip_ranking |
|-----------------|------------------------------|----------------------|-----------------------|------------------------|-------------------------|
| 192             | Canal St & Adams St          | 3814                 | 45                    | 50575                  | 1                       |
| 77              | Clinton St & Madison St      | 3918                 | 43                    | 45990                  | 2                       |
| 91              | Clinton St & Washington Blvd | 2775                 | 77                    | 45378                  | 3                       |
| 195             | Columbus Dr & Randolph St    | 7822                 | 14                    | 31370                  | 4                       |
| 287             | Franklin St & Monroe St      | 3465                 | 54                    | 30832                  | 5                       |
| 133             | Kingsbury St & Kinzie St     | 2448                 | 91                    | 30654                  | 6                       |
| 81              | Daley Center Plaza           | 3452                 | 55                    | 30423                  | 7                       |
| 174             | Canal St & Madison St        | 2693                 | 79                    | 27138                  | 8                       |
| 43              | Michigan Ave & Washington St | 12228                | 9                     | 25468                  | 9                       |
| 283             | LaSalle St & Jackson Blvd    | 2357                 | 96                    | 23021                  | 10                      |
| 177             | Theater on the Lake          | 15027                | 7                     | 16974                  | 26                      |
| 268             | Lake Shore Dr & North Blvd   | 18952                | 6                     | 15520                  | 33                      |
| 35              | Streeter Dr & Grand Ave      | 53104                | 1                     | 14879                  | 41                      |
| 85              | Michigan Ave & Oak St        | 21388                | 4                     | 14061                  | 45                      |
| 90              | Millennium Park              | 21749                | 3                     | 12357                  | 67                      |
| 76              | Lake Shore Dr & Monroe St    | 39238                | 2                     | 10566                  | 78                      |
| 3               | Shedd Aquarium               | 20617                | 5                     | 5815                   | 186                     |
| 341             | Adler Planetarium            | 11928                | 10                    | 4807                   | 219                     |
| 6               | Dusable Harbor               | 12546                | 8                     | 4607                   | 225                     |


```sql
---- Top 10 to-stations based on the number of trips made by customers and subscribers
WITH Total_Trip_2019 AS (
    SELECT *
    FROM (
        SELECT *
        FROM Divvy_Trips_2019_Q1
        UNION ALL
        SELECT *
        FROM Divvy_Trips_2019_Q2
        UNION ALL
        SELECT *
        FROM Divvy_Trips_2019_Q3
        UNION ALL
        SELECT *
        FROM Divvy_Trips_2019_Q4
    ) AS All_Trips
), Station_Counts AS (
    SELECT
        to_station_id,
        MAX(to_station_name) AS station_name,----Dummy aggregate
        SUM(CASE WHEN usertype = 'Customer' THEN 1 ELSE 0 END) AS Customer_trip_count,
        DENSE_RANK() OVER (ORDER BY SUM(CASE WHEN usertype = 'Customer' THEN 1 ELSE 0 END) DESC) AS Customer_trip_ranking,
        SUM(CASE WHEN usertype = 'Subscriber' THEN 1 ELSE 0 END) AS Subscriber_trip_count,
        DENSE_RANK() OVER (ORDER BY SUM(CASE WHEN usertype = 'Subscriber' THEN 1 ELSE 0 END) DESC) AS Subscriber_trip_ranking
    FROM Total_Trip_2019
    GROUP BY to_station_id---- Using station ID instead of station name to eliminate likely duplication and error in station name.
)
SELECT *
FROM Station_Counts
WHERE Customer_trip_ranking <= 10 OR Subscriber_trip_ranking <= 10; ----To filter result to top 10 station for each user type.
```
| to_station_id | station_name                 | Customer_trip_count | Customer_trip_ranking | Subscriber_trip_count | Subscriber_trip_ranking |
|---------------|------------------------------|----------------------|-----------------------|------------------------|-------------------------|
| 91            | Clinton St & Washington Blvd | 2493                 | 87                    | 48193                  | 1                       |
| 192           | Canal St & Adams St          | 2797                 | 70                    | 47330                  | 2                       |
| 77            | Clinton St & Madison St      | 3004                 | 61                    | 44307                  | 3                       |
| 81            | Daley Center Plaza           | 2503                 | 86                    | 30631                  | 4                       |
| 133           | Kingsbury St & Kinzie St     | 2290                 | 92                    | 30212                  | 5                       |
| 43            | Michigan Ave & Washington St | 12397                | 8                     | 27934                  | 6                       |
| 287           | Franklin St & Monroe St      | 2797                 | 70                    | 26763                  | 7                       |
| 174           | Canal St & Madison St        | 2246                 | 94                    | 26339                  | 8                       |
| 176           | Clark St & Elm St            | 4332                 | 35                    | 22720                  | 9                       |
| 283           | LaSalle St & Jackson Blvd    | 1564                 | 142                   | 22053                  | 10                      |
| 268           | Lake Shore Dr & North Blvd   | 23278                | 5                     | 19181                  | 18                      |
| 177           | Theater on the Lake          | 18803                | 6                     | 17136                  | 24                      |
| 85            | Michigan Ave & Oak St        | 23691                | 4                     | 14158                  | 43                      |
| 35            | Streeter Dr & Grand Ave      | 67585                | 1                     | 14138                  | 44                      |
| 90            | Millennium Park              | 25215                | 3                     | 12209                  | 65                      |
| 76            | Lake Shore Dr & Monroe St    | 30673                | 2                     | 9960                   | 89                      |
| 3             | Shedd Aquarium               | 16475                | 7                     | 5709                   | 191                     |
| 341           | Adler Planetarium            | 10598                | 9                     | 4838                   | 218                     |
| 6             | Dusable Harbor               | 9488                 | 10                    | 4808                   | 219                     |

```sql
---- Grouping of Users  generation and Age group.
WITH Total_Trip_2019 AS (
    SELECT *
    FROM (
        SELECT *
        FROM Divvy_Trips_2019_Q1
        UNION ALL
        SELECT *
        FROM Divvy_Trips_2019_Q2
        UNION ALL
        SELECT *
        FROM Divvy_Trips_2019_Q3
        UNION ALL
        SELECT *
        FROM Divvy_Trips_2019_Q4
    ) AS All_Trips
),  Generational_group AS ( SELECT *,
CASE
WHEN birthyear BETWEEN 1997 AND  2012 THEN 'GEN Z'
WHEN birthyear BETWEEN 1981 AND  1996 THEN 'Millennials'
WHEN birthyear BETWEEN 1965 AND  1980 THEN 'GEN X'
								WHEN birthyear IS NULL THEN 'Missing birth year'--- To omit missing data in the grouping
								Else 'Other'
								END AS Generation,
							CASE
								WHEN birthyear BETWEEN 1997 AND  2012 THEN 'Children and Teenager'
								WHEN birthyear BETWEEN 1981 AND  1996 THEN 'Young Group'
								WHEN birthyear BETWEEN 1965 AND  1980 THEN 'Middle Age Group'
								WHEN birthyear IS NULL THEN 'Missing birth year'--- To omit missing data in the grouping
								Else 'Elderly'
								END AS Age_Group
							FROM Total_Trip_2019
							)------ Grouping birthyear into generation and age group
		SELECT 
			   Max(Generation) AS Generation,---- Dummy agregrate
			   MAx(Age_Group) AS Age_Group,-----Dummy Agregrate
			   SUM(CASE WHEN usertype = 'Customer' THEN 1 Else 0 END	) AS Customer,
			   SUM(CASE WHEN usertype = 'Subscriber' THEN 1 Else 0 END	) AS Subscriber
		FROM Generational_group
		GROUP BY 
			GROUPING SETS (
					(Generation, Age_Group)
					);
```
| Generation       | Age_Group            | Customer | Subscriber |
|------------------|----------------------|----------|------------|
| GEN X            | Middle Age Group     | 47772    | 610655     |
| Missing birth year | Missing birth year | 532420   | 6331       |
| Millennials      | Young Group          | 243795   | 1992242    |
| GEN Z            | Children and Teenager| 43795    | 83389      |
| Other            | Elderly              | 12855    | 244750     |
```sql								 
---- Classification of  gender of users .
WITH Total_Trip_2019 AS (
    SELECT *
    FROM (
        SELECT *
        FROM Divvy_Trips_2019_Q1
        UNION ALL
        SELECT *
        FROM Divvy_Trips_2019_Q2
        UNION ALL
        SELECT *
        FROM Divvy_Trips_2019_Q3
        UNION ALL
        SELECT *
        FROM Divvy_Trips_2019_Q4
    ) AS All_Trips
)   SELECT 
			   MAX(gender)	AS Gender,----Dummy aggregrate
			   SUM(CASE WHEN usertype = 'Customer' THEN 1 Else 0 END	) AS Customer,
			   SUM(CASE WHEN usertype = 'Subscriber' THEN 1 Else 0 END	) AS Subscriber
	FROM Total_Trip_2019
	Group BY gender;
```
| Gender | Customer | Subscriber |
|--------|----------|------------|
| NULL   | 536455   | 22751      |
| Female | 131439   | 726539     |
| Male   | 212743   | 2188077    |
```sql			
/* Determining the most used pass by customer
Assumptions
1) Each trip involves only one bike lock.
2) For determining the pass type, we assume that a trip duration of up to 3 hours (180 minutes) falls under the single ride category
if the total cost (including the $1 unlocking fee and the per-minute charge) is less than the cost of a day pass ($18.10). 
Otherwise, it is considered a day pass.
Source of pricing : https://divvybikes.com/pricing*/

 WITH Total_Trip_2019 AS (
    SELECT *
    FROM Divvy_Trips_2019_Q1
    UNION ALL
    SELECT *
    FROM Divvy_Trips_2019_Q2
    UNION ALL
    SELECT *
    FROM Divvy_Trips_2019_Q3
    UNION ALL
    SELECT *
    FROM Divvy_Trips_2019_Q4
), Customer_trips_Passes AS (
    SELECT
        ABS(DATEDIFF(SECOND, start_time, end_time)) / 60.0 AS trip_duration_minutes, -- Convert seconds to minutes
        CASE
            WHEN ABS(DATEDIFF(SECOND, start_time, end_time)) / 60.0 <= 180 -- Less than or equal to 180 minutes (3 hours)
                AND (ABS(DATEDIFF(SECOND, start_time, end_time)) / 60.0 * 0.18 + 1) < 18.10 THEN 'Single_Ride'
            ELSE 'Day_Pass'
        END AS Pass_Type
    FROM 
        Total_Trip_2019
    WHERE 
        usertype = 'Customer'
)
SELECT
    Pass_Type,
    COUNT(*) AS Total_trip_Per_Pass
FROM 
    Customer_trips_Passes
GROUP BY 
    Pass_Type;
````
| Pass_Type   | Total_trip_Per_Pass |
|-------------|---------------------|
| Day_Pass    | 65081               |
| Single_Ride | 815556              |
```sql
WITH Total_Trip_2019 AS (
    SELECT *
    FROM Divvy_Trips_2019_Q1
    UNION ALL
    SELECT *
    FROM Divvy_Trips_2019_Q2
    UNION ALL
    SELECT *
    FROM Divvy_Trips_2019_Q3
    UNION ALL
    SELECT *
    FROM Divvy_Trips_2019_Q4
), Total_Trips_Hourly AS (
	SELECT 
    trip_id,
    usertype,
    start_time,
    DATEPART(HOUR, start_time) AS start_hour
	FROM Total_Trip_2019)
	SELECT 
			start_hour,
			SUM(CASE WHEN usertype = 'Customer' THEN 1 Else 0 END	) AS Customer,
			   SUM(CASE WHEN usertype = 'Subscriber' THEN 1 Else 0 END	) AS Subscriber

	FROM Total_Trips_Hourly
	GROUP BY start_hour
	ORDER BY 
    start_hour;
```
| Start Hour | Customer | Subscriber |
|------------|----------|------------|
| 0          | 8204     | 15874      |
| 1          | 5384     | 9033       |
| 2          | 3335     | 5329       |
| 3          | 1925     | 3686       |
| 4          | 1175     | 6614       |
| 5          | 2614     | 33156      |
| 6          | 6091     | 102141     |
| 7          | 12866    | 224842     |
| 8          | 21798    | 283982     |
| 9          | 28702    | 135131     |
| 10         | 44764    | 100700     |
| 11         | 60036    | 119930     |
| 12         | 69591    | 136676     |
| 13         | 75179    | 131946     |
| 14         | 78471    | 127914     |
| 15         | 80094    | 163346     |
| 16         | 82877    | 292901     |
| 17         | 84487    | 390667     |
| 18         | 68019    | 247234     |
| 19         | 50375    | 158385     |
| 20         | 34403    | 99757      |
| 21         | 24925    | 71268      |
| 22         | 21166    | 48535      |
| 23         | 14156    | 28320      |


## Findings
**Day of the Week Analysis:**
Subscribers take more trips during weekdays, while customers take more trips during weekends.

**Seasonal Trip Count Analysis:**
Summer has the highest total trip count, followed by Autumn, Spring, and Winter. However, the majority of customer trips occur during summer.
Subscribers generally contribute more trips than customers across all seasons.

**Trip Duration Analysis by User Type:**
Customers tend to have longer maximum trip durations compared to subscribers, suggesting occasional users might use the service for longer periods.
Subscribers have a much lower average trip duration, likely due to their frequent usage patterns.

**Top Origin and Destination Stations by User Group:**
The top stations for both customers and subscribers include popular tourist attractions and downtown areas.
There are differences in preferences between the two user groups, with subscribers favoring centrally located stations.

**User Classification by Generation and Age Group:**
Millennials make up the largest user group, followed by Generation X. However, there's a significant number of users with missing birth year information, affecting accurate age group classification.

**User Grouping by Gender:**
Males constitute the majority of both customers and subscribers, followed by females. However, a significant number of records have missing gender information.

**Pass Type Usage Analysis:**
Single ride passes are much more commonly used compared to day passes, indicating a preference for one-off trips rather than day-long usage.

**Hourly Trip Count Analysis by User Type:**
Subscribers tend to have higher usage during peak commuting hours, particularly in the morning (7-9 AM) and evening (4-6 PM) hours, suggesting usage for work-related travel.
Customers exhibit more varied usage patterns throughout the day, with peaks occurring during midday hours (10 AM - 3 PM) and lower usage during early morning and late night hours, indicating possible leisure or recreational use.


## Recommendation
Introduce seasonal membership options specifically designed for Summer and Autumn, offering flexible pricing plans and benefits tailored to the needs of customers.

Provide discounted rates or limited-time promotions for these seasonal memberships to incentivize customer uptake.

Develop weekend packages targeting young adults.

Launch a youth-focused marketing campaign leveraging social media platforms, influencer partnerships, and targeted advertising to reach young adults.




