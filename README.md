# Data Cleaning Portfolio Using SQL
#### Created by: Ee Ming Chua
#### Last Updated: September 10, 2024
#### RDBMS Used: PostgreSQL
#### Introduction: This portfolio documented my data cleaning approach and process using the raw trip data of [Divvy Bikes](https://divvybikes.com/) (a bikeshare system in Chicago) between August 2023 and July 2024.
#### Data Info & Source: Click [here](https://divvybikes.com/system-data) to read more details about the data and to access the raw data.
---
### LOAD DATASET
#### All raw data (12 individual files from August 2023 to July 2024) has been downloaded and is being loaded into a master table titled "trip_data".
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
#### The query identified 14 duplicates out of 5,715,693 rows. Since 14 is such a small number and unlikely to impact the analysis, the duplicates can be removed.
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
#### The query removed 14 duplicates, leaving the table with 5,715,679 rows.
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

### The percentage of missing end_lat and end_lng values is very low (0.14%). Dropping these rows would not significantly impact the overall dataset size. Potential causes for the missing end_lat and end_lng values include theft, tracker malfunctions, and data collection errors. However, the end_station_name, end_station_id, start_station_name, and start_station_id fields have a significant amount of missing data (approximately 16-17%). According to the company's website, users can pick up a Divvy bike that is not located at a docking station, and they do not need to return it to a docking station. This may explain the large amount of missing data.
  
### Dropping such a large portion of the dataset could potentially skew the analysis. In this case, it would be crucial to consult with stakeholders to determine the best approach to retain as much data as possible (for example, the possibility of imputing missing values using the closest station or ID based on lat/lng data). Since I am not working with Divvy Bikes directly, and for the purposes of this portfolio, the missing data will be dropped (though this is not the ideal approach). However, before dropping the missing data, it is important to determine the distribution of missing values over time. If the missing data occurs consistently each month, it can be dropped; if the majority of the missing values is inconsistent over time, further investigation with stakeholders would be needed.

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
![image](https://github.com/user-attachments/assets/535b9ee6-dd6d-448f-901b-8995ce3b1366)
![image](https://github.com/user-attachments/assets/1a09bf9a-fb54-4f12-9583-ae2a804728d7)
#### The average percentage of missing start_station_id values is about 15.86% over the 12-month period, with the distribution of missing values fairly even by month. The lowest percentage of missing values, 10.49%, occurred in February, when the weather tends to be the coldest and bikeshare usage is least popular. The highest percentage of missing values, 20.27%, occurred in June, when the weather is typically nicest and bikeshare usage is most common.

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
![image](https://github.com/user-attachments/assets/0c1cb69a-2a6d-4c72-9ab4-62358198752a)
![image](https://github.com/user-attachments/assets/a7756a43-aa4c-4d03-b978-7939cf7986d9)
#### The average percentage of missing end_station_id values is about 16.67% over the 12-month period, with the distribution of missing values fairly even by month, similar to the pattern observed for start_station_id discussed above. All missing values can be dropped.

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
#### The query removed 1,474,366 rows with missing values, leaving the table with 4,241,313 rows.
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
![image](https://github.com/user-attachments/assets/0004663f-9e15-4cbe-8201-d09c85b2ad01)
#### As the results show, the current data types are appropriate for the dataset, and no conversions are necessary.
---
### DATA CLEANING: INVALID/INCORRECT ENTRIES
#### 1. Check that if the ended_at timestamp is earlier than the started_at timestamp, which should not logically happen.
```sql
SELECT 
    COUNT(*) AS inconsistent_entries
FROM 
    trip_data
WHERE 
    ended_at < started_at;
```
#### The query returned 66 rows, and deleting them will not significantly impact the analysis.
```sql
DELETE FROM trip_data
WHERE ended_at < started_at;
```
#### 2. Identify unexpected values within value-specific columns (eg. rideable_type, member_casual)
#### Take rideable_type as an example -
```sql
SELECT DISTINCT rideable_type
FROM trip_data;
```
#### The results show that the rideable_type column contains three values: classic_bike, docked_bike, and electric_bike, all of which are valid options. Therefore, it is unlikely that the rideable_type column contains invalid entries. The same method can be applied to the member_casual column.

#### 3. Check data entries for standardized formatting (eg. Station Names are standardized)
```sql
SELECT DISTINCT start_station_name FROM trip_data;
SELECT DISTINCT end_station_name FROM trip_data;
```
#### The queries provided lists of unique station names in the dataset, and all are well-formatted and consistent. If any inconsistent entries are found, they can be updated to the standardized format using the example queries below -
```sql
-- 1. Convert to Title Case
UPDATE trip_data
SET start_station_name = INITCAP(start_station_name),
    end_station_name = INITCAP(end_station_name);

-- 2. Replace Abbreviations
UPDATE trip_data
SET start_station_name = REPLACE(start_station_name, 'St.', 'Street'),
    end_station_name = REPLACE(end_station_name, 'St.', 'Street');

-- 3. Trim Extra Spaces
UPDATE trip_data
SET start_station_name = TRIM(BOTH FROM start_station_name),
    end_station_name = TRIM(BOTH FROM end_station_name);

-- 4. Remove Duplicate Spaces
UPDATE trip_data
SET start_station_name = REGEXP_REPLACE(start_station_name, '\s+', ' '),
    end_station_name = REGEXP_REPLACE(end_station_name, '\s+', ' ');
```
#### 4. Other invalid entries may include rides that are too short or too long in duration, or rides that cover an unusually large distance. It is essential to collaborate with stakeholders to set appropriate thresholds for each scenario and validate the affected entries. Below are queries to identify potentially invalid entries -
```sql
-- Identifying Unusually Short Durations (eg. < 60 sec)
SELECT *
FROM trip_data
WHERE EXTRACT(EPOCH FROM (ended_at - started_at)) < 60;

-- Identifying Unusually Long Durations (eg. > 1 day)
SELECT *
FROM trip_data
WHERE EXTRACT(EPOCH FROM (ended_at - started_at)) > 86400;

-- Identify Unusually Long Distance Rides (eg. > 20 miles)
WITH distances AS (
    SELECT *,
           (3959 * acos(
               LEAST(1.0, GREATEST(-1.0,
                   cos(radians(start_lat)) * cos(radians(end_lat)) * 
                   cos(radians(end_lng) - radians(start_lng)) + 
                   sin(radians(start_lat)) * sin(radians(end_lat))
               ))
           )) AS distance_miles
    FROM trip_data
)
SELECT *
FROM distances
WHERE distance_miles > 20;
```
#### Since I am not working directly with Divvy, these entries will remain in the dataset for this project.
---
### This concludes the data cleaning process, and the dataset is now ready for analysis. I will be using R programming to perform an Exploratory Data Analysis (EDA) on the cleaned data. You can check out my EDA [here](https://rpubs.com/eeming/EDA_divvy_data)!
	

