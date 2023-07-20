# Strava Running Data Process Walkthough

## Aims
Strava is an app that is used to record exercise, predominatly running and cycling using GPS data. I have used Strava solely for running over the past few years. The free versoin of Strava has limited functionality and whilst the GPS data for all runs is recorded it is dificult to see any trends or compare activites.
For this project I therefore intended to:
- Identify long term trends in distance and speed
- Easily view different runs with split times, elevation change and map
- Easily identify Personal Bests at various distances
- Look for other interesting trends


## Tools
- **Excel** for initial data cleaning and format changes
   - XML import wizard
- **SQL** *(PostgreSQL)* for more in depth data cleaning and calculating additional information
     - Temp tables
     - CTEs
     - Window functions
     - String manipulation
     - Type casting
     - Lag functions
     - Others
- **Tableau** for final visulaisations

## Process

### Data Collection
Strava allows for the bulk downlaod of all data held for any account. Once requested I have access to all the data that Strava has associated with my account. The relevant information for this project is a .csv file with the metadata for all activities and .gpx files for each individual activity. For Strava each run counts as a single 'activity'

*It should be noted that the Strava app appears to try and take a GPS reading every second, therefore for nearly all cases a new datapoint is created every second*

### Data Cleaning/Preparing
The .csv file with the activity metadata requires very little cleaning. The majority of columns are irrelevant and are deleted leaving only a few columns with relevant information: date, distance, time etc.


However The Strava download with a .gpx file for each activity require more extensive preparation. .gpx files contain GPS data in .xml data format. 

.gpx or .xml files cannot be imported dircetly into PostgreSQL. However using the XML import wizard in Excel all the .gpx files can be combined and converted to a .csv file. Once imported into Excel a large number of irrelevant rows and columns are removed. This leaves only columns for Latitude, Longitude, Elevation and Timestamp.

*This method will have to be altered once the Excel row limit is reached, most likely be limiting the XML import to only include activities since the previous xml import. A further option is that, given that the granularity of the data is one data point every second, every other data point could be deleted without a significant impact on the outputs*


CREATE TABLE FOR CSV PGX INPUT
In order to import the activity data to PostgreSQL a table has to be created with appropriate column headings and data types matching that contained in the .csv file.


INPUT DATA
The input for the Merged_activity_gpx table is updated by referencing the updated .csv that now includes the new GPS data for new activites.

WALKTHROUGH SQL CODE

<Details>

<Summary>PostrgreSQL Code</Summary>

```pgsql
-- Full code to Tableau ready for Strava GPX data

--Creating claculated columns for later use
-----START-----
DROP TABLE IF EXISTS gpxstep;
CREATE TEMP TABLE gpxstep AS
WITH cte_gpx AS
(SELECT 
latitude,
longitude,
elevation,
run_time_stamp,
run_time_stamp::date AS run_date,
(run_time_stamp::timestamp)::time AS run_timestamp,

LAG (latitude,1) 
 OVER (PARTITION BY run_time_stamp::date 
	   ORDER BY run_time_stamp) AS prev_lat, 
LAG (longitude,1) 
 OVER (PARTITION BY run_time_stamp::date 
	   ORDER BY run_time_stamp) AS prev_long,
LAG ((run_time_stamp::timestamp)::time,1) 
 OVER (PARTITION BY run_time_stamp::date 
	   ORDER BY run_time_stamp) AS prev_run_timestamp,
LAG (elevation,1) 
OVER (PARTITION BY run_time_stamp::date 
	  ORDER BY run_time_stamp) AS prev_elevation
FROM "Merged_activity_gpx"
ORDER BY run_time_stamp ASC)

SELECT *,
earth_distance (
	ll_to_earth(latitude, longitude),
	ll_to_earth(prev_lat, prev_long))
	AS d_distance, 
--This calculates d_distance 
run_timestamp - prev_run_timestamp AS d_time,
elevation - prev_elevation AS d_elevation

FROM cte_gpx;



DROP TABLE IF EXISTS gpxstep2;
CREATE TABLE gpxstep2 AS(

WITH gpxstep2 AS
(SELECT *,
(d_time)/NULLIF((d_distance/1000),0) AS "point_speed_min/km", 
 -- NULLIF removes div by zero error
SUM (d_distance) OVER (PARTITION BY run_time_stamp::date
					   ROWS
					   BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS sum_distance,
-- ROWS BETWEEN not RANGE BETWEEN to account for nulls and keep running total per run
SUM (d_time) OVER (PARTITION BY run_time_stamp::date
				  ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS sum_time,
SUM (d_elevation) OVER (PARTITION BY run_time_stamp::date
						ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS sum_d_elevation
 FROM gpxstep) 

SELECT *,
(sum_time)/NULLIF((sum_distance/1000),0) AS "cumulative_speed_min/km"
FROM gpxstep2
ORDER BY run_time_stamp asc);
-----END-----


-- USING gpxstep2 this generates split times per 100m for each run, for use in longer split times
-----START-----
DROP TABLE IF EXISTS splits_100m;
CREATE TABLE splits_100m AS(

WITH histo_data AS
(SELECT
run_time_stamp,
run_date,
latitude,
longitude,
elevation,
DENSE_RANK () OVER (ORDER BY (run_date))  AS run_no,
-- gives an id number for each run(date) allowing for each unique split ID to be created
sum_distance,
d_time,
WIDTH_BUCKET (sum_distance::numeric, 
			  0.0,
			  MAX((CEIL(sum_distance / 100))/10 )::numeric,
			  (MAX(CEIL(sum_distance / 100) ))::int) AS splits 
-- MAX((CEIL(sum_distance / 100))/10 ) converts m to km, force rounds up to nearest 100m, result in km

FROM gpxstep2
GROUP BY run_time_stamp, run_date, sum_distance, d_time, latitude, longitude, elevation
ORDER BY run_time_stamp)


SELECT 
*,
run_no + (splits-1)/1000::double precision AS split_dist,
--split_dist returns: run_number.run_distance_in_100m_intervals the unique split ID
SUM (d_time) OVER (PARTITION BY (run_no + (splits-1)/1000::double precision)) AS split_time

FROM histo_data
ORDER BY run_time_stamp);


DROP TABLE IF EXISTS splits_100m_final;
CREATE TABLE splits_100m_final AS(

WITH splits_100m_final AS(
SELECT 
DISTINCT -- This keeps only one record per split, the remaining are not required
split_dist::decimal (10,3),--Type cast to force 3 decimal places to ease future calcs
split_time,
run_date,
MAX(split_dist) OVER (PARTITION BY run_date) AS incomplete_split 
--This creates a reference in the same format as the unique split ID in order to 
--remove the final incomplete split of a run

FROM splits_100m
ORDER BY split_dist)

SELECT * 
FROM
splits_100m_final
ORDER BY split_dist);

DELETE FROM splits_100m_final
WHERE split_dist = incomplete_split;
--This removes the incomplete splits

ALTER TABLE splits_100m_final
DROP COLUMN incomplete_split;
--This removes the unecessary column

DROP TABLE IF EXISTS splits_100m;
-- Removes intermediate table
-----END-----

-- USING gpxstep2 this generates split times per 1000m for each run, for use only for individual runs
-----START-----
DROP TABLE IF EXISTS splits_1000m;
CREATE TABLE splits_1000m AS(

WITH histo_data AS
(SELECT
run_time_stamp,
run_date,
DENSE_RANK () OVER (ORDER BY (run_date))  AS run_no,
-- gives an id number for each run(date) allowing for each unique split ID to be created
sum_distance,
d_elevation,
d_time,
latitude,
longitude,
elevation,
run_timestamp,
WIDTH_BUCKET (sum_distance::numeric, 
			  0.0,
			  MAX((CEIL(sum_distance / 1000))/10 )::numeric,
			  (MAX(CEIL(sum_distance / 1000) ))::int) AS splits 
-- MAX((CEIL(sum_distance / 1000))/10 ) converts m to km, force rounds up to nearest 1000m, result in km

FROM gpxstep2
GROUP BY run_time_stamp, run_date, sum_distance, d_time, d_elevation, latitude, longitude, elevation, run_timestamp
ORDER BY run_time_stamp)


SELECT 
*,
run_no + (splits-1)/100::double precision AS split_dist,
--split_dist returns: run_number.run_distance_in_1000m_intervals the unique split ID
SUM (d_time) OVER (PARTITION BY (run_no + (splits-1)/100::double precision)) AS split_time

FROM histo_data
ORDER BY run_time_stamp);


-- This maps the lat long datapoints to the split no.
DROP TABLE IF EXISTS latlongsplit_final;
CREATE TABLE latlongsplit_final AS(

WITH latlongsplit_final AS(
SELECT 
split_dist,
split_time,
run_date,
latitude,
longitude,
elevation,
run_timestamp,
SUM(d_elevation) OVER (PARTITION BY split_dist) AS sum_d_elevation,
MAX(split_dist) OVER (PARTITION BY run_date) AS incomplete_split 
--This creates a reference in the same format as the unique split ID in order to 
--remove the final incomplete split of a run

FROM splits_1000m
ORDER BY split_dist)

SELECT * 
FROM
latlongsplit_final
ORDER BY split_dist);

DELETE FROM latlongsplit_final
WHERE split_dist = incomplete_split;
--This removes the incomplete splits

ALTER TABLE latlongsplit_final
DROP COLUMN incomplete_split;
--This removes the unecessary column

-- END OF LAT LONG SPLIT CREATION


DROP TABLE IF EXISTS splits_1000m_final;
CREATE TABLE splits_1000m_final AS(

WITH splits_1000m_final AS(
SELECT 
DISTINCT -- This keeps only one record per split, the remaining are not required
split_dist,
split_time,
run_date,
SUM(d_elevation) OVER (PARTITION BY split_dist) AS sum_d_elevation,
MAX(split_dist) OVER (PARTITION BY run_date) AS incomplete_split 
--This creates a reference in the same format as the unique split ID in order to 
--remove the final incomplete split of a run

FROM splits_1000m
ORDER BY split_dist)

SELECT * 
FROM
splits_1000m_final
ORDER BY split_dist);

DELETE FROM splits_1000m_final
WHERE split_dist = incomplete_split;
--This removes the incomplete splits

ALTER TABLE splits_1000m_final
DROP COLUMN incomplete_split;
--This removes the unecessary column

DROP TABLE IF EXISTS splits_1000m;
-- Removes intermediate table
-----END-----

-- Creates rolling split times
-----START-----
--Fastest 100m

-- Due to inacuracies in the gps readings and small distance, results not to be trusted
SELECT
split_dist,
split_time,
run_date
FROM splits_100m_final
ORDER BY split_time ASC;

--Rolling Fastest Splits
DROP TABLE IF EXISTS rolling_splits_final;
CREATE TABLE rolling_splits_final AS(	
SELECT
SUM(split_time) OVER(PARTITION BY run_date 
					 ORDER BY split_dist 
					 ROWS BETWEEN 9 PRECEDING AND CURRENT ROW ) AS prev_km_split,
-- Adds together previous 10 100m splits
SUM(split_time) OVER(PARTITION BY run_date 
					 ORDER BY split_dist 
					 ROWS BETWEEN 49 PRECEDING AND CURRENT ROW ) AS prev_5km_split,
-- Adds together previous 50 100m splits
SUM(split_time) OVER(PARTITION BY run_date 
					 ORDER BY split_dist 
					 ROWS BETWEEN 99 PRECEDING AND CURRENT ROW ) AS prev_10km_split,
-- Adds together previous 100 100m splits
SUM(split_time) OVER(PARTITION BY run_date 
					 ORDER BY split_dist 
					 ROWS BETWEEN 210 PRECEDING AND CURRENT ROW ) AS prev_half_marathon_split,
-- Adds together previous 211 100m splits
	
	
	
split_dist,
split_time,
run_date
FROM splits_100m_final
ORDER BY split_dist asc);

UPDATE rolling_splits_final
SET prev_km_split = null
WHERE(RIGHT (split_dist::text ,3))::int<010;
-- This removes sections of run before the first km is completed

UPDATE rolling_splits_final
SET prev_5km_split = null
WHERE(RIGHT (split_dist::text ,3))::int<050;
-- This removes sections of run before the first 5km is completed

UPDATE rolling_splits_final
SET prev_10km_split = null
WHERE(RIGHT (split_dist::text ,3))::int<100;
-- This removes sections of run before the first 10km is completed

UPDATE rolling_splits_final
SET prev_half_marathon_split = null
WHERE(RIGHT (split_dist::text ,3))::int<210;
-- This removes sections of run before the first half marathon is completed

DELETE FROM rolling_splits_final
WHERE split_time IS null;
-- Removes rows where split times incorrectly calculated due to nulls

SELECT *
FROM rolling_splits_final
ORDER BY split_dist ASC

-----END-----

```
</Details>

OUTPUTS

The SQL code uses the raw merged inputs of Latidute, Longitide, Elevation and Timestamp to calculate and create a number of tables to be used in the visualisation. These include fields that are or could be useful for further analysis, most notably:
- Distance
- Speed
- Splits Times (for every 100m/1km/5km/10km and Half Marathon)
- Primary Keys for each split
- Rolling Split Times (Using a window function to more accurately find Personal Bests, granularity 100m)
- 'Point' Speed (speed between consecutive data points, susceptible to error due to vaugaries of GPS connection)

As Tableau Public cannot link directly to PostgreSQL the tables have to be exported from PostgreSQL as .csv files. The exported tables are then collated in a .csv file where a Marco is run that converts the date types for various columns to those more easi;y manipulated wiothin Tableau.
CHECK PREVOIOUS PARAGRAPH

### Data Analysis










### Data Visualisation








