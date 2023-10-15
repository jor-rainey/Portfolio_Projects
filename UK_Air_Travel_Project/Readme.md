# UK Air Travel Data Process Walkthrough

The project can be found here - [UK Air Travel Project](https://public.tableau.com/app/profile/jake.rainey/viz/UKAirTravelJan2023/UKAirTravelDashboard)

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
  - SUMIF formulas
  - Other basic formulas
- **Tableau** for final visualisations
  - Spatial calculations
  - tooltip use

## Process

### Data Collection

Punctuality and route data for this project was sourced from the UK CAA [(Civil Aviation Authority)](https://www.caa.co.uk/data-and-analysis/uk-aviation-market/airports/uk-airport-data/uk-airport-data-2023/january-2023/)

It should be noted that this data only includes arrivals at major UK airports. The reporting of this data to only include these airports is a choice made by the CAA. However this does cover the vast majority of all UK passenger air traffic.

*January 2023 was chosen arbitrarily and seasonal changes could be seen with the addition of the data from other months*

In order to plot airline routes data for the airport locations was sourced from [The Global Airport Database](https://www.partow.net/miscellaneous/airportdatabase/) 


### Data Cleaning/Preparing

With regard to the flight routes the raw data from the CAA only gives the arrival airport and the origin airport and country. This was supplemented within Excel with the data from the Global Airport Database to give a more complete picture of the routes in a more usable format. For example using aiport IATA codes as opposed to town/airport names. This helped to clarify the difference between similarly named places (e.g. DHAKA and DAKAR) or between multi airport cities (e.g. Milan Malpensa and Milan Linate). Principally this was acheved using Lookup functions, however given that the two data sources differed in format slightly, giving the town name vs airport name, occasional edge cases were found and corrected manually. 

For example the mixup of Syndey Airport (Australia, ~44,000,000 annual passengers) and Sydney Airport (Canada, ~180,000 annual passengers) It is important to always be aware if the data at any part of the process makes sense. I only spotted this error at the visualisation stage and had to go back to correct within Excel.
 
Collating this data also allowed for the concatenation of the IATA codes to form the recognisable route pairs, such as JFK-LHR.

This section of the data along with the associated fact tables for Origin Airport, Arrival Airport and Airline eventually became the main filterable sections of the final dashboard within Tableau.

SCREENSHOT OF AIRPORT FACT TABLE 

SCREENSHOT OF TABEAU INPUT SCROLL FAR LEFT



The airline group to which an airline belongs has been researched manually for each airline found in the data. Little other choice is available when looking to include this data given a central source is hard to find and the differing legal statuses between owners, subsidiaries, partners etc. This is important to include when looking into competition between airlines especially when the grouings are not always intuative (e.g. **Ryanair Group** own *Ryanair* and *Ryanair UK* as well as less obviously owning *Malta Air*, *Lauda Motion*, *Lauda Europe* and *Buzz*).

Basic sense checking of the data throughout the data cleaning process was underaken using pivot tables within Excel. This aided in ensuring the various formulas were working as expected and allowed the investigation of elements of the data to be used in the final visualisation. 

Consideration was given to the addition of regional airport groupings for the UK and other countries with many airports found within this data set. However, these are less intuative depending on the country and give an unecessary extra level of granularity that would be unused for most countries given the low amount of airports.



The other main stage in the data preparation was the changing of the punctuality data from percentages, as given by the CAA, to the count of flights. This reverted the aggregated data proveded by the CAA back to the raw data originally collected which is more prefereable to work with. The percentages then became represented later on within Tableau using the punctuality pie charts.
The process of changing this was achieved using simple Excel formuals as the total number of flights on each route was also given

SCREENSHOT OF TOTAL FLIGHTS FOR EACH DELAY CATEGORY

Another aspect of the reported data from the CAA was to include cancelled flights in the total number of flights on each route. I took the descision to remove these flights from the data as part of the cleaning process. This was due to the very few numbers of cancelled flights v actual flights and the fact that not ommiting these altered the punctuality percantages. 


The last section of data cleaned before the final visualisation was the average delays for routes, airports and airlines. 
This was calculated for each of these categories using SUMIF functions weighting the average delays from the raw data with consideration to the number of flights. 

SCREENSHOT FROM EXCEL OF FORMULA FOR AV DELAY

Care was taken in the calculation of the average delays to convert from the raw "Decimal Time" to a more understandable format. (e.g. 2.5 mins to 00:02:30)


### Data Analysis

Tabeau was used to create this dashboard, the full version of this can be found on my Tableau Public profile [here](https://public.tableau.com/app/profile/jake.rainey/viz/UKAirTravelJan2023/UKAirTravelDashboard) along with full interactivity. The screenshots below demonstrate some of the capabilities and isights available using this dashboard.

The following two screenshots show the differences between British Airways and Ryanair at their respective main hubs, Heathrow and Stansted:

#### British Airways Heathrow Arrivals
![BA Hub](https://github.com/jor-rainey/ImagesforReadMe/blob/main/Air%20Travel%20Project%20Screenshots/Heathrow%20BA%20Hub.png)

#### Ryanair Stansted Arrivals
![Ryanair Hub](https://github.com/jor-rainey/ImagesforReadMe/blob/main/Air%20Travel%20Project%20Screenshots/Stansted%20Ryanair%20Hub.png)

British Airways, the UK flag carier, are focussing on more frequent longer haul flights on major routes. While Ryanair operate a more point to point model with less frequent routes to more desitiations, nearly all of which are short/medium haul within Europe. 



Observations can also be made comparing flights from differing countries. For example flights from the USA are dominated by only a few UK airports and there is more fierce competition between different airlines on these routes, particularly JFK-LHR. 

#### USA Arrivals, Heathrow Punctuality 
![USA Arrivals](https://github.com/jor-rainey/ImagesforReadMe/blob/main/Air%20Travel%20Project%20Screenshots/USA%20Map%20%2B%20Tooltip.png)

Here we can see the few arrival airports (a consequency of the longhaul nature of the flights to the USA) and the punctulaity details at Heathrow 

#### USA Arrivals, Competition and Punctuality Distribution
![USA Arrivals Competition](https://github.com/jor-rainey/ImagesforReadMe/blob/main/Air%20Travel%20Project%20Screenshots/USA%20Competition%20%2BPunctuality.png)

The level of competetion between the airlines can be seen here along with the Punctuality Distribution of all arrivals from the USA. British Airways also tends to be an outlier in terms of punctuality for the most popular routes from the USA as can be seen in the example tooltip.



This information can be contrasted against flight arrivals from Germany.
The shorthaul nature of the flights from Germany allow for more point to point travel and whilst there is a similar level of comptetion between airlines these are largely kept on different routes. 

#### Germany Arrivals, Route Info
![Germany Arrivals Map](https://github.com/jor-rainey/ImagesforReadMe/blob/main/Air%20Travel%20Project%20Screenshots/Germany%20Map%20%2B%20Tooltip.png)
A more point to point route map is seen here given the short haul nature of these routes.

#### Germany Arrivals, Competition and Punctuality Distribution
![Germany Arrivals Competition](https://github.com/jor-rainey/ImagesforReadMe/blob/main/Air%20Travel%20Project%20Screenshots/Germany%20Competition%20%2B%20Punctuality.png)
A larger range of arrival airports seen in the most popular routes, with the punctuality distribution suggesting that shorter haul flights to less busy airports may aid punctulaity when compared to flights arriving from the USA.


Further insights could be gained by including data across the different months of the year. This would demonstrate the seasonal changes in both flight numbers as well as routes used. The initial data cleaning and preparation would be much the same with the addition of a date for each flight (given the available data, only the month in which the flight took place is available) and a method to visualise the changes over time.

## Data Visulaisation

There is a complex web of international airline groupings that, whilst difficult to convey in the final visualisation is impotant to show and is available in the tooltips of the Airline - Flights section. 

A possible additon to this dashboard would be to further demonstrate the airline groupings, although perhaps more usefull in a global context as opposed to flights arriving in the UK.

The colour scheme and fonts (sans-serif white and yellow on a black background) have been selected to evoke the airport departure board colour schemes that are ubiquitous across the world.
The punctiality pie chart follow an intuitive Green - Red traffic light system, and the airlines are coloured to match the main colour of their livery. Both these elements attempt to aid in making the dashboard more easilly and quickly readable. The only drawback being that many airlines use similar reds and blues. However given the number of airlines within the data set it is impossible to give distinct colours to each airline and this compromise is unaviodable.

