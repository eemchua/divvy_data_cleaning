# Data Cleaning Portfolio
#### Created by: Ee Ming Chua
#### Last Updated: Aug 29, 2024
#### RDBMS Used: PostgreSQL
#### Introduction: This portfolio documented my data cleaning approach and process using the raw trip data of [Divvy Bikes](https://divvybikes.com/) (a bikeshare system in Chicago) between August 2023 and July 2024.
#### Data Info & Source: Click [here](https://divvybikes.com/system-data) to read more details about the data and to access the raw data.
---
### LOAD DATASET
#### All raw data (12 individual files from August 2023 to July 2024) is downloaded and being loaded into a master-table titled "trip_data".
```sql
CREATE TABLE trip_data (
    ride_id VARCHAR(255),
    rideable_type VARCHAR(255),
    started_at TIMESTAMP,
    ended_at TIMESTAMP,
    start_station_name VARCHAR(255),
    start_station_id VARCHAR(255),
    end_station_name VARCHAR(255),
    end_station_id VARCHAR(255),
    start_lat FLOAT,
    start_lng FLOAT,
    end_lat FLOAT,
    end_lng FLOAT,
    member_casual VARCHAR(50)
);
```
#### The query created the "trip_data" table with 5,715,693 rows.
---
### DATA CLEANING: DUPLICATES
#### 1. Identify and Count Duplicates
```sql
SELECT rideable_type, 
       started_at, 
       ended_at, 
       start_station_name, 
       start_station_id, 
       end_station_name, 
       end_station_id, 
       start_lat, 
       start_lng, 
       end_lat, 
       end_lng,
       member_casual,
       COUNT(*) AS duplicate_count
FROM trip_data
GROUP BY rideable_type, 
         started_at, 
         ended_at, 
         start_station_name, 
         start_station_id, 
         end_station_name, 
         end_station_id, 
         start_lat, 
         start_lng, 
         end_lat, 
         end_lng, 
         member_casual
HAVING COUNT(*) > 1;
```
#### The query identified 14 duplicates out of the 5,715,693 rows. Since 14 is such a small number and unlikely to impact the analysis, the 14 duplicates can ce removed.
#### 2. Remove Duplicates
```sql
WITH CTE AS (
    SELECT ctid,
           ROW_NUMBER() OVER (PARTITION BY rideable_type, started_at, ended_at, start_station_name, start_station_id, end_station_name, end_station_id, start_lat, start_lng, end_lat, end_lng, member_casual ORDER BY ctid) AS rn
    FROM trip_data
)
DELETE FROM trip_data
WHERE ctid IN (
    SELECT ctid
    FROM CTE
    WHERE rn > 1
);
```
#### The query removed 14 duplicates and the table returned with 5,715,679 rows.
---
### DATA CLEANING: NULL VALUES
#### 1. Identify and Count Missing Values
```sql
SELECT 
    'rideable_type' AS column_name, COUNT(*) AS missing_count
FROM trip_data
WHERE rideable_type IS NULL

UNION ALL

SELECT 
    'started_at' AS column_name, COUNT(*) AS missing_count
FROM trip_data
WHERE started_at IS NULL

UNION ALL

SELECT 
    'ended_at' AS column_name, COUNT(*) AS missing_count
FROM trip_data
WHERE ended_at IS NULL

UNION ALL

SELECT 
    'start_station_name' AS column_name, COUNT(*) AS missing_count
FROM trip_data
WHERE start_station_name IS NULL

UNION ALL

SELECT 
    'start_station_id' AS column_name, COUNT(*) AS missing_count
FROM trip_data
WHERE start_station_id IS NULL

UNION ALL

SELECT 
    'end_station_name' AS column_name, COUNT(*) AS missing_count
FROM trip_data
WHERE end_station_name IS NULL

UNION ALL

SELECT 
    'end_station_id' AS column_name, COUNT(*) AS missing_count
FROM trip_data
WHERE end_station_id IS NULL

UNION ALL

SELECT 
    'start_lat' AS column_name, COUNT(*) AS missing_count
FROM trip_data
WHERE start_lat IS NULL

UNION ALL

SELECT 
    'start_lng' AS column_name, COUNT(*) AS missing_count
FROM trip_data
WHERE start_lng IS NULL

UNION ALL

SELECT 
    'end_lat' AS column_name, COUNT(*) AS missing_count
FROM trip_data
WHERE end_lat IS NULL

UNION ALL

SELECT 
    'end_lng' AS column_name, COUNT(*) AS missing_count
FROM trip_data
WHERE end_lng IS NULL

UNION ALL

SELECT 
    'member_casual' AS column_name, COUNT(*) AS missing_count
FROM trip_data
WHERE member_casual IS NULL;
```
#### Query result:
![image](https://github.com/user-attachments/assets/3cded0a5-d70c-4950-b847-6577acab683f)

#### There are 3 identical groups of missing values:
1. **end_lat and end_lng**: Percentage of missing data = (7,755 / 5,715,679) * 100 ≈ 0.14%
2. **start_station_name and start_station_id**: Percentage of missing data = (947,020 / 5,715,679) * 100 ≈ 16.6%
3. **end_station_name and end_station_id**: Percentage of missing data = (989,470 / 5,715,679) * 100 ≈ 17.3%

#### The percentage of missing end_lat and end_lng values is very low (0.14%). Dropping these rows would not significantly impact the overall dataset size. Potential causes for missing end_lat and end_lng values include thefts, malfunction of trackers, and data collection errors. 

#### end_station_name, end_station_id, start_station_name, and start_station_id have a significant amount of missing data (approximately 16-17%). Dropping such a large portion of the dataset could potentially skew the analysis. In such case, it would be crucial to discuss with the stakeholders and to determine the best apporach to move forward. Since I am not working with Divvy Bikes directly, for the purpose of this portfolio, the missing data will be dropped (again, not the best approach). However, before dropping the missing data, it is crucial to determine the distribution of missing values over time. If it is an consistent occurance every month, the missing data can be dropped; if it is not, collaboration with stakeholders will be needed.    

#### For start_station_name and start_station_id, using start_station_id to run a query of missing values over time by month-
```sql
SELECT 
    DATE_TRUNC('month', started_at) AS month,
    COUNT(*) FILTER (WHERE start_station_id IS NULL) AS missing_count,
    COUNT(*) AS total_count,
    ROUND((COUNT(*) FILTER (WHERE start_station_id IS NULL)::float / COUNT(*) * 100)::numeric, 2) AS percentage_missing
FROM 
    trip_data
GROUP BY 
    month
ORDER BY 
    month;
```
#### Query result:
![image](https://github.com/user-attachments/assets/0c1cb69a-2a6d-4c72-9ab4-62358198752a)



