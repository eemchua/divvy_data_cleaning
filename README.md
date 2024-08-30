# Data Cleaning Portfolio
#### Created by: Ee Ming Chua
#### Last Updated: Aug 29, 2024
#### RDBMS Used: PostgreSQL with minor usage of Excel
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
>![image](https://github.com/user-attachments/assets/3cded0a5-d70c-4950-b847-6577acab683f)

#### There are 3 identical groups of missing values:
1. **end_lat and end_lng**: Percentage of missing data = (7,755 / 5,715,679) * 100 ≈ 0.14%
2. **start_station_name and start_station_id**: Percentage of missing data = (947,020 / 5,715,679) * 100 ≈ 16.6%
3. **end_station_name and end_station_id**: Percentage of missing data = (989,470 / 5,715,679) * 100 ≈ 17.3%

#### The percentage of missing end_lat and end_lng values is very low (0.14%). Dropping these rows would not significantly impact the overall dataset size. Potential causes for missing end_lat and end_lng values include thefts, malfunction of trackers, and data collection errors. The end_station_name, end_station_id, start_station_name, and start_station_id have a significant amount of missing data (approximately 16-17%). According to the company's website, users can pick up a divvy avaialble that is not at a docking station, and they also do not need to return to a docking station. This may explain the reason of the large missing data.

#### Dropping such a large portion of the dataset could potentially skew the analysis. In such case, it would be crucial to discuss with the stakeholders and to determine the best apporach to retain as much data as possible (for example, the posibility of imputing the missing values using the closest station or ID based on their lat/lng data). Since I am not working with Divvy Bikes directly, and for the purpose of this portfolio, the missing data will be dropped (again, not the best approach). However, before dropping the missing data, it is crucial to determine the distribution of missing values over time. If it is a consistent occurance every month, the missing data can be dropped; if majority of the missing values is inconsistent over time, futher investigation with stakeholders is needed.    

#### For start_station_name and start_station_id, using start_station_id to run a query of missing values over time by month -
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
>![image](https://github.com/user-attachments/assets/535b9ee6-dd6d-448f-901b-8995ce3b1366)
>![image](https://github.com/user-attachments/assets/1a09bf9a-fb54-4f12-9583-ae2a804728d7)
#### The average percentage of mising start_station_id is about 15.86% over the 12-month period, and the distribution of missing values by month is fairly even. The lowest percentage of mising of 10.49% occurred during February, when the weather tends to be the coldest and bikeshare usage is the least popular; the percentage of mising of 20.27% occurred during June, when the weather tends to be the nicest and bikeshare usage is most common.

#### Similarly, for end_station_name and end_station_id, using end_station_id to run a query of missing values over time by month -
```sql
SELECT 
    DATE_TRUNC('month', started_at) AS month,
    COUNT(*) FILTER (WHERE end_station_id IS NULL) AS missing_count,
    COUNT(*) AS total_count,
    ROUND((COUNT(*) FILTER (WHERE end_station_id IS NULL)::float / COUNT(*) * 100)::numeric, 2) AS percentage_missing
FROM 
    trip_data
GROUP BY 
    month
ORDER BY 
    month;
```

#### Query result:
>![image](https://github.com/user-attachments/assets/0c1cb69a-2a6d-4c72-9ab4-62358198752a)
>![image](https://github.com/user-attachments/assets/a7756a43-aa4c-4d03-b978-7939cf7986d9)
#### The average percentage of mising end_station_id is about 16.67% over the 12-month period, and the distribution of missing values by month is again fairly even, mirroring what we see for the start_station_id discussed right above. All missing values can be dropped.

#### 2. Remove Missing Values
```sql
DELETE FROM trip_data
WHERE 
    start_station_id IS NULL 
    OR end_station_id IS NULL 
    OR start_station_name IS NULL
    OR end_station_name IS NULL
    OR start_lat IS NULL
    OR start_lng IS NULL
    OR end_lat IS NULL
    OR end_lng IS NULL
    OR rideable_type IS NULL
    OR member_casual IS NULL
    OR started_at IS NULL
    OR ended_at IS NULL;
```
#### The query removed 1,474,366 missing values and the table returned with 4,241,313 rows.
---
### DATA CLEANING: DATA TYPES
#### 1. Identify Data Types of All Columns
```sql
SELECT 
    column_name, 
    data_type
FROM 
    information_schema.columns
WHERE 
    table_name = 'trip_data';
```
#### Query result:
>![image](https://github.com/user-attachments/assets/0004663f-9e15-4cbe-8201-d09c85b2ad01)
#### As results showns, the current data types are appropriate for the dataset, and no conversions are needed.
---
### DATA CLEANING: INVALID ENTRIES
#### 1. Check if ended_at is before started_at (which should not logically happen)
```sql
SELECT 
    COUNT(*) AS inconsistent_entries
FROM 
    trip_data
WHERE 
    ended_at < started_at;
```
#### The query returned with 66 rows, and deleting them will not have significant impact the analysis -
```sql
DELETE FROM trip_data
WHERE ended_at < started_at;
```

	

