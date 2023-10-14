# UK Air Travel Data Process Walkthrough

The project can be found here - LINK TO TABLEAU PUBLIC

## Aims

The purpose of this project was to investigate the routes and punctuality of airlines and airports for the UK aviation market.
For this project I intended to:
- Map airline routes
- Show number of flights on routes
- Give punctuality information for both airports and airlines
- Allow for intuative visualisation/comparison across routes and airlines 

## Tools
- **Excel** for initial data cleaning an preparation
  - LOOKUP formulas
- **Tableau** for final visualisations
  - plotting routes
  - tooltip use

## Process

### Data Collection

Data for this project was sourced from the UK CAA [(Civil Aviation Authority)](https://www.caa.co.uk/data-and-analysis/uk-aviation-market/airports/uk-airport-data/uk-airport-data-2023/january-2023/)

It should be noted that this data only includes arrivals at major UK airports. The reporting of this data to only include these airports is a choice made by the CAA. However this does cover the vast majority of all UK passenger air traffic.

*January 2023 was chosen arbitrarily and seasonal changes could be seen with the addition of the data from other months*


### Data Cleaning/Preparing


AIRPORT MASTER TABLE

AIRPORT CODES 

Occasional confusion can be found ithin the airport codes themselves and simialry named towns across the world. For example the mixup of Syndey Airport (Australia, ~44,000,000 annual passengers) and Sydney Airport (Canada, ~180,000 annual passengers)
It is important to always be checking if data at any part of the process makes sense. I only spotted this error at the visualisation stage and had to go back to correct within Excel.

DELAY PERCENTAGES TO RAW FIGURES

WORKING WITH TIMES IN EXCEL, DECIMAL TIME, EXTRACTING ONLY TIME NOT DATETIME 


ROUTE INFO
This was produced by the concatenation of the origin airport and the arrival airport.


The airline group to which an airline belongs has been researched manually for each airline found in the data. Little other choice is available when looking to include this data given a central source is hard to find and the differing legal statuses between owners, subsidiaries, partners etc. This is important to include when looking into competition between airlines especially when the grouings are not always intuative (e.g. **Ryanair Group** own *Ryanair* and *Ryanair UK* as well as less obviously owning *Malta Air*, *Lauda Motion*, *Lauda Europe* and *Buzz*).


Basic sense checking of the data throughout the data cleaning process was underaken using pivot tables within Excel. This aided in ensuring the various formulas were working as expected and allowed the investigation of elements of the data to be used in the final visualisation. 

Consideration was given to the addition of regional airport groupings for the UK and other countries with many airports found within this data set. However, these are less intuative depending on the country and give an unecessary extra level of granularity that would be unused for most countries given the low amount of airports.

### Data Analysis

DOMINATION OF RYANIR IN EUROPE - STANSTED HUB - LOWER FREQUENCY ROUTES
BA OF LONGHAUL - HIGH FREQUENCY ROUTES 
SCREEN SHOTS OF FUNCTIONALITY

PUNCTULAITY DISTRIBUTION GRAPHIC - AIRLINE COMPARISON, RYANIAR V GOOD

Further insights could be gained by including data across the different months of the year. This would demonstrate the seasonl changes in both flight numbers as well as routes used. The initial data cleaning and preparation would be much the same with the addition of a date for each flight (given the available data, only the month in which the flight took place is available) and a method to visualise the changes over time.

## Data Visulaisation

There is a complex web of international airline groupings that, whilst difficult to convey in the final visualisation is impotant to show and is available in the tooltips of the Airline - Flights section. 

A possible additon to this dashboard would be to further demonstrate the airline groupings, although perhaps more usefull in a global context as opposed to flights arriving in the UK.

The colour scheme and fonts (sans-serif white and yellow on a black background) have been selected to evoke the airport departure board colour schemes that are ubiquitous across the world.
The punctiality pie chart follow an intuitive Green - Red traffic light system, and the airlines are coloured to match the main colour of their livery. Both these elements attempt to aid in making the dashboard more easilly and quickly readable. The only drawback being that many airlines use similar reds and blues. However given the number of airlines within the data set it is impossible to give distinct colours to each airline and this compromise is unaviodable.

