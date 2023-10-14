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

## Process

### Data Collection

Data for 

### Data Cleaning/Preparing


The airline group to which an airline belongs has been researched manually for each airline found in the data. Little other choice is available when looking to include this data given a central source is hard to find and the differing legal statuses between owners, subsidiaries, partners etc. This is important to include when looking into competition between airlines especially when the grouings are not always intuative (e.g. **Ryanair Group** own *Ryanair* and *Ryanair UK* as well as less obviously owning *Malta Air*, *Lauda Motion*, *Lauda Europe* and *Buzz*).


Basic sense checking of the data throughout the data cleaning process was underaken using pivot tables within Excel. This aided in ensuring the various formulas were working as expected and allowed the investigation of elements of the data to be used in the final visualisation. 

Consideration was given to the addition of regional airport groupings for the UK and other countries with many airports found within this data set. However, these are less intuative depending on the country and give an unecessary extra level of granularity that would be unused for most countries given the low amount of airports.

### Data Analysis



Further insights could be gained by including data across the different months of the year. This would demonstrate the seasonl changes in both flight numbers as well as routes used. The initial data cleaning and preparation would be much the same with the addition of a date for each flight (given the available data, only the month in which the flight took place is available) and a method to visualise the changes over time.

## Data Visulaisation

There is a complex web of airline groupings that, whilst difficult to convey in the final visualisation is impotant to show and is available in the tooltips of the Airline - Flights section. 

A possible additon to this dashboard would be to further demonstrate the airline groupings, although perhaps more usefull in a global context as opposed to flights arriving in the UK.
