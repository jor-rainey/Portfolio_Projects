# Strava Running Data Process Walkthough

## Aims
Strava is an app that is used to record exercise, predominatly running and cycling using GPS data. I have used Strava solely for running over the past few years. The free versoin of Strava has limited functionality and whilst the GPS data for all runs is recorded it is dificult to see any trends or compare activites.
For this project I therefore intended to:
- Identify trends in distance/elevation and speed
- Easily identify Personal Bests at various distances
- Look for interesting trends
- Easily compare different runs

## Tools
- **Excel** for initial data cleaning and format changes
   - XML import wizard
- **SQL** *(PostgreSQL)* for more in depth data cleaning and calculating additional information
     - FUNCTIONS
- **Tableau** for final visulaisations

## Process

### Data Collection
Strava allows for the bulk downlaod of all data held for any account. Once requested I have access to all the data that Strava has associated with my account. The relevant information for this project is a .csv file with the metadata for all activities and .gpx files for each individual activity. For Strava each run counts as a single 'activity'

### Data Cleaning/Preparing
The .csv file with the activity metadata requires very little cleaning. The majority of columns are irrelevant and are deleted leaving only a few columns with relevant information: date, distance, time etc.


However The Strava download with a .gpx file for each activity require more extensive preparation. .gpx files contain GPS data in .xml data format. 

.gpx or .xml files cannot be imported dircetly into PostgreSQL. However using the XML import wizard in Excel all the .gpx files can be combined and converted to a .csv file. Once imported into Excel a large number of irrelevant rows and columns are removed. This leaves only columns for Latitude, Longitude, Elevation and Timestamp.

*This method will have to be altered once the excel row limit is reached, most likely be limiting the XML import to only include activities since the previous xml import*


CREATE TABLE FOR CSV PGX INPUT
In order to import the activity data to PostgreSQL a table has to be created with appropriate column headings and data types matching that contained in the .csv file.


INPUT DATA
The input for the Merged_activity_gpx table is updated by referencing the updated .csv that now includes the new GPS data for new activites.

WALKTHROUGH SQL CODE
FULL CODE CAN BE FOUND HERE LINK

```pgsql
SELECT *
FROM thing
```


OUTPUTS




### Data Analysis

### Data Visualisation








