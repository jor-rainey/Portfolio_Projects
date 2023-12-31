# Strava Running Data Process Walkthrough

The project can be found here - [Strava Running Project](https://public.tableau.com/app/profile/jake.rainey/viz/StravaRunningDataProject/StravaRunningData)

## Aims
Strava is an app that is used to record exercise, predominantly running and cycling using GPS data. I have used Strava solely for running over the past few years. The free version of Strava has limited functionality and whilst the GPS data for all runs is recorded it is difficult to see any trends or compare activities.
For this project I therefore intended to:
- Identify long term trends in distance and speed
- Easily view different runs with split times, elevation change and map
- Easily identify Personal Bests at various distances
- Look for other interesting trends


## Tools
- **Excel** for initial data cleaning and format changes
   - XML import wizard
   - Macros
- **SQL** *(PostgreSQL)* for more in depth data cleaning and calculating additional information
     - Temp tables
     - CTEs
     - Window functions
     - String manipulation
     - Type casting
     - Lag functions
     - Others
- **Tableau** for final visualizations
     - Building Dashboards
     - Creating Theme
     - Navigation through dashboard


## Process

### Data Collection
Strava allows for the bulk download of all data held for any account. Once requested I have access to all the data that Strava has associated with my account. The relevant information for this project is a .csv file with the metadata for all activities and .gpx files for each individual activity. For Strava each run counts as a single 'activity'

*It should be noted that the Strava app appears to try and take a GPS reading every second, therefore for nearly all cases a new datapoint is created every second*

### Data Cleaning/Preparing
The .csv file with the activity metadata requires very little cleaning. The majority of columns are irrelevant and are deleted leaving only a few columns with relevant information: date, distance, time etc.


However The Strava download with a .gpx file for each activity requires more extensive preparation. .gpx files contain GPS data in .xml data format. 

.gpx or .xml files cannot be imported directly into PostgreSQL. However using the XML import wizard in Excel all the .gpx files can be combined and converted to a .csv file. Once imported into Excel a large number of irrelevant rows and columns are removed. This leaves only columns for Latitude, Longitude, Elevation and Timestamp.

*This method will have to be altered once the Excel row limit is reached, most likely limiting the XML import to only include activities since the previous xml import. A further option is that, given that the granularity of the data is one data point every second, every other data point could be deleted without a significant impact on the outputs*

In order to import the activity data to PostgreSQL a table has to be created with appropriate column headings and data types matching that contained in the .csv file.

The input for the Merged_activity_gpx table is updated by referencing the updated .csv that now includes the new GPS data for new activities.

#### PostgreSQL Code

The full PostgreSQL code can be seen here:

<Details>

<Summary>PostrgreSQL Code</Summary>

```pgsql
-- Full code to Tableau ready for Strava GPX data

--Creating calculated columns for later use
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
--This removes the unnecessary column

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
--This removes the unnecessary column

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
--This removes the unnecessary column

DROP TABLE IF EXISTS splits_1000m;
-- Removes intermediate table
-----END-----

-- Creates rolling split times
-----START-----
--Fastest 100m

-- Due to inaccuracies in the gps readings and small distance, results not to be trusted
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

The SQL code uses the raw merged inputs of Latitude, Longitude, Elevation and Timestamp to calculate and create a number of tables to be used in the visualization. These include fields that are or could be useful for further analysis, most notably:
- Distance
- Speed
- Splits Times (for every 100m/1km/5km/10km and Half Marathon)
- Primary Keys for each split
- Rolling Split Times (Using a window function to more accurately find Personal Bests, granularity 100m)
- 'Point' Speed (speed between consecutive data points, susceptible to error due to vagaries of GPS connection)

As Tableau Public cannot link directly to PostgreSQL the tables have to be exported from PostgreSQL as .csv files. The exported tables are then collated in an Excel file where a Marco is run that converts the date types and units for various columns to those more easily manipulated within Tableau.

### Data Analysis

Using Tableau I have created a number of visualizations from this data to make trends, personal bests and general run information more accessible.
The full dashboard can be found on my Tableau Public profile [here](https://public.tableau.com/app/profile/jake.rainey/viz/StravaRunningDataProject/StravaRunningData) along with the full interactivity. However the screenshots below can demonstrate how the information is displayed along with some of the information that can be gathered.

#### Strava Project Page 1

![Strava Project Page 1](https://github.com/jor-rainey/ImagesforReadMe/blob/main/Strava%20Project%20Screenshots/Strava%20Pg1.png)

This page gives a short introduction to the data being displayed and then shows some headline information. Notably the kilometerage increase since 2020, increase typical run length and increase in elevation change. All runs that take place in the two main locations also have their paths overlayed highlighting most frequent running routes.

*Background map information and Latitude and Longitude information is hidden across this dashboard.*

---
#### Strava Project Page 2

![Strava Project Page 2](https://github.com/jor-rainey/ImagesforReadMe/blob/main/Strava%20Project%20Screenshots/Strava%20Pg2.png)

The Distance and Average Pace graphic demonstrates the increase in typical run length and increase in typical run pace over the years. Additionally a general decrease in run pace as run length increases is apparent. Clusters of runs at just beyond 5km, 10km and half marathon distances can also be seen. 

The Split Time Distribution graph follows an expected pattern. Given the large enough sample size the majority of splits cluster around the average split time while tailing away at the faster and slower ends. The graph is capped at a max of 6:00mins/km removing outliers (occasionally where there are errors in the GPS data) and the long tail to the distribution. This allows more of the available graph space to be used for the relevant information, in this case the vast majority of split times. The other dimension on this graph indicates the elevation change of each split. While it can be seen that where the elevation increases across a split the split time slows (and visa-versa) this is better demonstrated in visualizations later in the dashboard. 

Further information on this page includes Personal Bests and Running Distance Total split per year where the drop off in mid 2023 correlates with an injury and recovery time.

A filter on the page can adjust each visualization to show each year individually.

---
#### Strava Project Page 3 - 10km

![Strava Project Page 3 - 10km](https://github.com/jor-rainey/ImagesforReadMe/blob/main/Strava%20Project%20Screenshots/Strava%20Pg3.png)

This page is split to show the same information for 1km, 5km, 10km and half-marathon distances.

All pages show the improvement of Personal Best progression over time and, given the available white space on the graphs, space is available for annotations that point out other notable clusters/trends at each distance.

---
#### Strava Project Page 4

![Strava Project Page 4](https://github.com/jor-rainey/ImagesforReadMe/blob/main/Strava%20Project%20Screenshots/Strava%20Pg4.png)

The Split Time Distribution graphic on this page plots the split time distribution along the length of each run. A slowing of the pace at greater distances maps as expected. However there appears to be little change between km 1 and around km 16. This is likely because shorter slower runs in 2020/2021 bring down the average split time at lower distances whereas once I started running greater distances my standard pace was quicker, broadly balancing out the averages for the first 16km.

The Split Time v Elevation Change graphic nicely demonstrates an optimum elevation change for maximum pace. The trend line shows quicker times where there is a negative elevation change as expected but also that once the elevation change is greater than -14m/km the positive effects begin to reduce. When filtered for different years the profile of runs is very apparent, barely any elevation change during 2020 (as all runs were along a river) and far greater elevation change in later years when most runs were in a different location.

*Outliers have been removed to better display trends for typical elevation changes and paces, plotting all points would leave a lot of white space and make conclusions difficult to draw due to difficulty in reading the graph*

---

#### Strava Project Page 5

![Strava Project Page 5](https://github.com/jor-rainey/ImagesforReadMe/blob/main/Strava%20Project%20Screenshots/Strava%20Pg5.png)

Here any run can be selected and the route is mapped showing elevation change and split time. This page acts as an index and mirrors the information displayed on Strava for each run. 

---


### Data Visualisation

Throughout the visualization a consistent simple style, colour scheme and layout is used. This aids in the understanding and ease of use of the visualizations as well as creating a more professional look.

Buttons for navigation give more of a flow through the data than the standard Tableau pages that are also available to use as navigation through the dashboard.

The layout has been configured specifically for a laptop browser window. However the layout and readability would have to be assessed for different screen types depending on the anticipated use cases 











