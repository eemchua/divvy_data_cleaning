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
###### Query result:
>![image](https://github.com/user-attachments/assets/3cded0a5-d70c-4950-b847-6577acab683f)



