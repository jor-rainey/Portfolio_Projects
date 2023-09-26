# Energy Production Data Process Walkthrough

The project can be found here - LINK

## Aims

The majority of greenhouse gas emissions globally come from energy production (72%, Climate Analysis Indicators Tool (World Resources Institute, 2017))
For this project I therefore intended to:
- Create a comparison tool for countries emissions broken down by energy source
- Allow for comparisions over time
- Show context allowing for a countries size/population
- Easy/consistent side by side visualisation
  
## Tools

- **SQL** *(PostgreSQL)* for data cleaning and preparation
  - Joins
  - Temp Tables
  - CTEs
  - Type Casting
  - Others

- **Power BI** for data cleaning and preparation along with final visualisations
  - Power Query Editor
  - Merges
  - Unpivot
  - Others


## Process

### Data Collection

All data for this project was sourced from the the World Bank Open Data website (https://data.worldbank.org/) This was in order to provide cosistent data across all countries from a trustworthy source. 
Data included the percentage of energy generated each year for each country split per energy source type (hydroelectric, coal etc.) along with other useful secondary information such as population.


### Data Cleaning/Preparation

Each energy source had its own Excel file and was available for download only in a "wide" format. 

EXCEL SCREENSHOT WIDE FORMAT

In order to ensure a more workable data set these Excel files were transformed from a "wide" to "long" format within PostgreSQL.
While "wide" data tends to be easier to interpret and read, it is far easier to create visualisations and reports when the data is in "long" format. This is therefore often a key step irrespective of the visualisation software used.

In addition to this transformation the data sources were collated, using a variety of joins, into one table with the addition of a reference id column, a concatenation of year and coutry name.

Further work in PostgreSQL included the tidying of data using type casts and simple calculations. For example using population figures to generate per capita information.

#### PostgreSQL Code

*This code includes a section relating to net energy imports that which has not been included in the final visualisations. Net imports are necessarily coverd in the production values of other countries*

*In addition, while this code runs effectively, the same results of this code can surely be achieved more efficiently due to the frequent repetition*


Power BI can connect directly to the PostgreSQL database and tables created by running the PostgreSQL code. Once this connection has been established additional data preparation is required using the Power BI Power Query Editor.

Following the amalgamation of all the energy sources data into one table this again needs to be trandformed from "wide" to "long" format. This can be achieved by unpivoting the data allowing the energy source to be a more easily visualtsed variable.

BEFORE AND AFTER UNPIVOTING SCREENSHPTS

The final few adjustments within the Power Query Editor included filtering out non-countries from the country list (MIddle Income, OEDC, EU, etc.), the additional column indicating whether or not the energy source is "Green", tweaking of units and finally a merge of the queries to bring in population and power consumption/capita.  

FINAL POEWR QUERY SCREENSHOT


### Data Analysis

The final Power BI dashboard can be found with full interactivity HERE LINKLINKLINK

However the screenshot below can be used to compare the energy production sources of Germany and France since 1960 as well as giving context of population and power consumption per capita.

SCREENSHOT OF FINAL DASHBOARD



France and Germany to be interesting and appropriate countires to compare. This is due to the many similarities in terms of; geography, politics, population and energy consumption.
However clear differences can still be seen. Most notably both countries have Nuclear power take up an increasing proportion of energy production in the 1970s. In France this continues through the 1980s to level out at around 75% of total production leaving a broadly stable energy production make up for the past 25 years. 
Politically in Germany nuclear power faced more difficulties and after similar initial investment as France the proportion of nuclear energy tapered away to around a mere 15% for the most recent available year. In lieu of nuclear power Germany can be seen to be relying more heavilly on coal and has a significant recent uptick in renewable energy.



### Data Visualisation

For this dashboard, created with ease of two country comparison in mind, it was important to show as much useful clear information as possible on one page in a side by side format. 

The main intention of each of the graphs is to show the overview and long term trends of each variable over time. However more detailed figures can be found in the tooltips of all graphs, and the corss filtering aides in the comparison of the particualr selected variable.

The colour schemes have been intentionally chosen to aid in the easy identification of each energy source. This helps to mitigate against any crowding of the graphs especially 'Domestic Energy Production Composition kWh'. The approximate limit for useful numbers of variables shown on a line graph is being reached in this example.

CLOSE UP SCREENSHOT

An important aspect of building the dashboard was to ensure that the y-axis scales of all graphs of both countries remain the same. This ensured that when comparing two countries with vastly different energy production amounts the difference between them was obviously apparent to the viewer of the dashboard. While it should be anticipated that, for example, China and Cameroon are not terribly useful countries to compare due to their massive population and energy consumption differences. If the y-axis scales of energy production are allowd to act independantly for each country there is scope for confusion or incorrect conclusions to be drawn.

EXAMPLE SCREENSHOT

A fairly neutral background colour scheme is used here and instructions are given to the user for selecting countries to copare in a bid to ease usability.


