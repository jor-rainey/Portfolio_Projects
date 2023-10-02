# Energy Production Data Process Walkthrough

The project can be found [here](https://github.com/jor-rainey/Portfolio_Projects/blob/main/Energy_Production_Project/Energy%20Production%20Project.pbix)

## Aims

The majority of greenhouse gas emissions globally come from energy production (72%, Climate Analysis Indicators Tool (World Resources Institute, 2017)) In order to gain more insight into the largest greenhouse gas emmiting countries I set out to create a way to compare them giving context to the raw figures.

For this project I therefore intended to:
- Create a comparison tool for countries emissions broken down by energy source
- Allow for comparisions over time
- Show further context allowing for a countries size/population
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
  - Design for usablilty/end user consideration
  - Others


## Process

### Data Collection

All data for this project was sourced from the the World Bank Open Data website (https://data.worldbank.org/) This was in order to provide cosistently formatted data across all countries from a trustworthy source.
The data from the World Bank included the percentage of energy generated each year for each country split per energy source type (hydroelectric, coal etc.) along with other useful secondary information such as population.


### Data Cleaning/Preparation

Each energy source had its own Excel file and was available for download only in a "wide" format as below: 

![Excel Wide](https://github.com/jor-rainey/ImagesforReadMe/blob/main/Energy%20Project%20Screenshots/Excel%20Wide.png)

In order to ensure a more workable data set these Excel files were transformed from a "wide" to "long" format within PostgreSQL.
While "wide" data tends to be easier to interpret and read, it is far easier to create visualisations and reports when the data is in "long" format. This is therefore often a key step irrespective of the visualisation software used.

In addition to this transformation the data sources were collated, using a number of joins, into one table with the addition of a reference id column (a concatenation of year and coutry name) Further work in PostgreSQL included the tidying of data using type casts and simple calculations. For example using population figures to generate per capita information.

#### PostgreSQL Code
The full PostgeSQL code used for this project can be found here:

<Details>

<Summary>PostgreSQL Code</Summary>

```pgsql

-- Wide to Long for raw data x7

----------ENERGY PRODUCTION----------
----- COAL -----
DROP TABLE IF EXISTS coal_long;
CREATE TEMP TABLE coal_long AS(
SELECT country_name,
	values.*,
	CONCAT (country_name, year) AS id
FROM public."coal_percentage_raw" AS c
	CROSS JOIN LATERAL(
	VALUES
		(c."1960", '1960'),
		(c."1961", '1961'),
		(c."1962", '1962'),
		(c."1963", '1963'),
		(c."1964", '1964'),
		(c."1965", '1965'),
		(c."1966", '1966'),
		(c."1967", '1967'),
		(c."1968", '1968'),
		(c."1969", '1969'),
		(c."1970", '1970'),
		(c."1971", '1971'),
		(c."1972", '1972'),
		(c."1973", '1973'),
		(c."1974", '1974'),
		(c."1975", '1975'),
		(c."1976", '1976'),
		(c."1977", '1977'),
		(c."1978", '1978'),
		(c."1979", '1979'),
		(c."1980", '1980'),
		(c."1981", '1981'),
		(c."1982", '1982'),
		(c."1983", '1983'),
		(c."1984", '1984'),
		(c."1985", '1985'),
		(c."1986", '1986'),
		(c."1987", '1987'),
		(c."1988", '1988'),
		(c."1989", '1989'),
		(c."1990", '1990'),
		(c."1991", '1991'),
		(c."1992", '1992'),
		(c."1993", '1993'),
		(c."1994", '1994'),
		(c."1995", '1995'),
		(c."1996", '1996'),
		(c."1997", '1997'),
		(c."1998", '1998'),
		(c."1999", '1999'),
		(c."2000", '2000'),
		(c."2001", '2001'),
		(c."2002", '2002'),
		(c."2003", '2003'),
		(c."2004", '2004'),
		(c."2005", '2005'),
		(c."2006", '2006'),
		(c."2007", '2007'),
		(c."2008", '2008'),
		(c."2009", '2009'),
		(c."2010", '2010'),
		(c."2011", '2011'),
		(c."2012", '2012'),
		(c."2013", '2013'),
		(c."2014", '2014'),
		(c."2015", '2015'),
		(c."2016", '2016'),
		(c."2017", '2017'),
		(c."2018", '2018'),
		(c."2019", '2019'),
		(c."2020", '2020'),
		(c."2021", '2021')
	) AS values(coal_percent, year)
ORDER BY country_name, year
);


--CHECKER
SELECT id, country_name, year, coal_percent FROM coal_long;
-- COAL --

---- HYDRO ----

DROP TABLE IF EXISTS hydro_long;
CREATE TEMP TABLE hydro_long AS(
SELECT country_name,
	values.*,
	CONCAT (country_name, year) AS id
FROM public."hydro_percentage_raw" AS h
	CROSS JOIN LATERAL(
	VALUES
		(h."1960", '1960'),
		(h."1961", '1961'),
		(h."1962", '1962'),
		(h."1963", '1963'),
		(h."1964", '1964'),
		(h."1965", '1965'),
		(h."1966", '1966'),
		(h."1967", '1967'),
		(h."1968", '1968'),
		(h."1969", '1969'),
		(h."1970", '1970'),
		(h."1971", '1971'),
		(h."1972", '1972'),
		(h."1973", '1973'),
		(h."1974", '1974'),
		(h."1975", '1975'),
		(h."1976", '1976'),
		(h."1977", '1977'),
		(h."1978", '1978'),
		(h."1979", '1979'),
		(h."1980", '1980'),
		(h."1981", '1981'),
		(h."1982", '1982'),
		(h."1983", '1983'),
		(h."1984", '1984'),
		(h."1985", '1985'),
		(h."1986", '1986'),
		(h."1987", '1987'),
		(h."1988", '1988'),
		(h."1989", '1989'),
		(h."1990", '1990'),
		(h."1991", '1991'),
		(h."1992", '1992'),
		(h."1993", '1993'),
		(h."1994", '1994'),
		(h."1995", '1995'),
		(h."1996", '1996'),
		(h."1997", '1997'),
		(h."1998", '1998'),
		(h."1999", '1999'),
		(h."2000", '2000'),
		(h."2001", '2001'),
		(h."2002", '2002'),
		(h."2003", '2003'),
		(h."2004", '2004'),
		(h."2005", '2005'),
		(h."2006", '2006'),
		(h."2007", '2007'),
		(h."2008", '2008'),
		(h."2009", '2009'),
		(h."2010", '2010'),
		(h."2011", '2011'),
		(h."2012", '2012'),
		(h."2013", '2013'),
		(h."2014", '2014'),
		(h."2015", '2015'),
		(h."2016", '2016'),
		(h."2017", '2017'),
		(h."2018", '2018'),
		(h."2019", '2019'),
		(h."2020", '2020'),
		(h."2021", '2021')
	) AS values(hydro_percent, year)
ORDER BY country_name, year
);

--CHECKER
SELECT id, country_name, year, hydro_percent FROM hydro_long;
-- HYDRO --

----- NATURAL GAS -----


DROP TABLE IF EXISTS natural_gas_long;
CREATE TEMP TABLE natural_gas_long AS(
SELECT country_name,
	values.*,
	CONCAT (country_name, year) AS id
FROM public."natural_gas_percentage_raw" AS g
	CROSS JOIN LATERAL(
	VALUES
		(g."1960", '1960'),
		(g."1961", '1961'),
		(g."1962", '1962'),
		(g."1963", '1963'),
		(g."1964", '1964'),
		(g."1965", '1965'),
		(g."1966", '1966'),
		(g."1967", '1967'),
		(g."1968", '1968'),
		(g."1969", '1969'),
		(g."1970", '1970'),
		(g."1971", '1971'),
		(g."1972", '1972'),
		(g."1973", '1973'),
		(g."1974", '1974'),
		(g."1975", '1975'),
		(g."1976", '1976'),
		(g."1977", '1977'),
		(g."1978", '1978'),
		(g."1979", '1979'),
		(g."1980", '1980'),
		(g."1981", '1981'),
		(g."1982", '1982'),
		(g."1983", '1983'),
		(g."1984", '1984'),
		(g."1985", '1985'),
		(g."1986", '1986'),
		(g."1987", '1987'),
		(g."1988", '1988'),
		(g."1989", '1989'),
		(g."1990", '1990'),
		(g."1991", '1991'),
		(g."1992", '1992'),
		(g."1993", '1993'),
		(g."1994", '1994'),
		(g."1995", '1995'),
		(g."1996", '1996'),
		(g."1997", '1997'),
		(g."1998", '1998'),
		(g."1999", '1999'),
		(g."2000", '2000'),
		(g."2001", '2001'),
		(g."2002", '2002'),
		(g."2003", '2003'),
		(g."2004", '2004'),
		(g."2005", '2005'),
		(g."2006", '2006'),
		(g."2007", '2007'),
		(g."2008", '2008'),
		(g."2009", '2009'),
		(g."2010", '2010'),
		(g."2011", '2011'),
		(g."2012", '2012'),
		(g."2013", '2013'),
		(g."2014", '2014'),
		(g."2015", '2015'),
		(g."2016", '2016'),
		(g."2017", '2017'),
		(g."2018", '2018'),
		(g."2019", '2019'),
		(g."2020", '2020'),
		(g."2021", '2021')
	) AS values(natural_gas_percent, year)
ORDER BY country_name, year
);

--CHECKER
SELECT id, country_name, year, natural_gas_percent FROM natural_gas_long;

-- NATURAL GAS -- 

----- NUCLEAR -----


DROP TABLE IF EXISTS nuclear_long;
CREATE TEMP TABLE nuclear_long AS(
SELECT country_name,
	values.*,
	CONCAT (country_name, year) AS id
FROM public."nuclear_percentage_raw" AS n
	CROSS JOIN LATERAL(
	VALUES
		(n."1960", '1960'),
		(n."1961", '1961'),
		(n."1962", '1962'),
		(n."1963", '1963'),
		(n."1964", '1964'),
		(n."1965", '1965'),
		(n."1966", '1966'),
		(n."1967", '1967'),
		(n."1968", '1968'),
		(n."1969", '1969'),
		(n."1970", '1970'),
		(n."1971", '1971'),
		(n."1972", '1972'),
		(n."1973", '1973'),
		(n."1974", '1974'),
		(n."1975", '1975'),
		(n."1976", '1976'),
		(n."1977", '1977'),
		(n."1978", '1978'),
		(n."1979", '1979'),
		(n."1980", '1980'),
		(n."1981", '1981'),
		(n."1982", '1982'),
		(n."1983", '1983'),
		(n."1984", '1984'),
		(n."1985", '1985'),
		(n."1986", '1986'),
		(n."1987", '1987'),
		(n."1988", '1988'),
		(n."1989", '1989'),
		(n."1990", '1990'),
		(n."1991", '1991'),
		(n."1992", '1992'),
		(n."1993", '1993'),
		(n."1994", '1994'),
		(n."1995", '1995'),
		(n."1996", '1996'),
		(n."1997", '1997'),
		(n."1998", '1998'),
		(n."1999", '1999'),
		(n."2000", '2000'),
		(n."2001", '2001'),
		(n."2002", '2002'),
		(n."2003", '2003'),
		(n."2004", '2004'),
		(n."2005", '2005'),
		(n."2006", '2006'),
		(n."2007", '2007'),
		(n."2008", '2008'),
		(n."2009", '2009'),
		(n."2010", '2010'),
		(n."2011", '2011'),
		(n."2012", '2012'),
		(n."2013", '2013'),
		(n."2014", '2014'),
		(n."2015", '2015'),
		(n."2016", '2016'),
		(n."2017", '2017'),
		(n."2018", '2018'),
		(n."2019", '2019'),
		(n."2020", '2020'),
		(n."2021", '2021')
	) AS values(nuclear_percent, year)
ORDER BY country_name, year
);

--CHECKER
SELECT id, country_name, year, nuclear_percent FROM nuclear_long;

-- NUCLEAR -- 

----- OIL -----


DROP TABLE IF EXISTS oil_long;
CREATE TEMP TABLE oil_long AS(
SELECT country_name,
	values.*,
	CONCAT (country_name, year) AS id
FROM public."oil_percentage_raw" AS o
	CROSS JOIN LATERAL(
	VALUES
		(o."1960", '1960'),
		(o."1961", '1961'),
		(o."1962", '1962'),
		(o."1963", '1963'),
		(o."1964", '1964'),
		(o."1965", '1965'),
		(o."1966", '1966'),
		(o."1967", '1967'),
		(o."1968", '1968'),
		(o."1969", '1969'),
		(o."1970", '1970'),
		(o."1971", '1971'),
		(o."1972", '1972'),
		(o."1973", '1973'),
		(o."1974", '1974'),
		(o."1975", '1975'),
		(o."1976", '1976'),
		(o."1977", '1977'),
		(o."1978", '1978'),
		(o."1979", '1979'),
		(o."1980", '1980'),
		(o."1981", '1981'),
		(o."1982", '1982'),
		(o."1983", '1983'),
		(o."1984", '1984'),
		(o."1985", '1985'),
		(o."1986", '1986'),
		(o."1987", '1987'),
		(o."1988", '1988'),
		(o."1989", '1989'),
		(o."1990", '1990'),
		(o."1991", '1991'),
		(o."1992", '1992'),
		(o."1993", '1993'),
		(o."1994", '1994'),
		(o."1995", '1995'),
		(o."1996", '1996'),
		(o."1997", '1997'),
		(o."1998", '1998'),
		(o."1999", '1999'),
		(o."2000", '2000'),
		(o."2001", '2001'),
		(o."2002", '2002'),
		(o."2003", '2003'),
		(o."2004", '2004'),
		(o."2005", '2005'),
		(o."2006", '2006'),
		(o."2007", '2007'),
		(o."2008", '2008'),
		(o."2009", '2009'),
		(o."2010", '2010'),
		(o."2011", '2011'),
		(o."2012", '2012'),
		(o."2013", '2013'),
		(o."2014", '2014'),
		(o."2015", '2015'),
		(o."2016", '2016'),
		(o."2017", '2017'),
		(o."2018", '2018'),
		(o."2019", '2019'),
		(o."2020", '2020'),
		(o."2021", '2021')
	) AS values(oil_percent, year)
ORDER BY country_name, year
);

--CHECKER
SELECT id, country_name, year, oil_percent FROM oil_long;

-- OIL -- 

----- RENEWABLES -----

DROP TABLE IF EXISTS renewable_long;
CREATE TEMP TABLE renewable_long AS(
SELECT country_name,
	values.*,
	CONCAT (country_name, year) AS id
FROM public."renewable_percentage_raw" AS r
	CROSS JOIN LATERAL(
	VALUES
		(r."1960", '1960'),
		(r."1961", '1961'),
		(r."1962", '1962'),
		(r."1963", '1963'),
		(r."1964", '1964'),
		(r."1965", '1965'),
		(r."1966", '1966'),
		(r."1967", '1967'),
		(r."1968", '1968'),
		(r."1969", '1969'),
		(r."1970", '1970'),
		(r."1971", '1971'),
		(r."1972", '1972'),
		(r."1973", '1973'),
		(r."1974", '1974'),
		(r."1975", '1975'),
		(r."1976", '1976'),
		(r."1977", '1977'),
		(r."1978", '1978'),
		(r."1979", '1979'),
		(r."1980", '1980'),
		(r."1981", '1981'),
		(r."1982", '1982'),
		(r."1983", '1983'),
		(r."1984", '1984'),
		(r."1985", '1985'),
		(r."1986", '1986'),
		(r."1987", '1987'),
		(r."1988", '1988'),
		(r."1989", '1989'),
		(r."1990", '1990'),
		(r."1991", '1991'),
		(r."1992", '1992'),
		(r."1993", '1993'),
		(r."1994", '1994'),
		(r."1995", '1995'),
		(r."1996", '1996'),
		(r."1997", '1997'),
		(r."1998", '1998'),
		(r."1999", '1999'),
		(r."2000", '2000'),
		(r."2001", '2001'),
		(r."2002", '2002'),
		(r."2003", '2003'),
		(r."2004", '2004'),
		(r."2005", '2005'),
		(r."2006", '2006'),
		(r."2007", '2007'),
		(r."2008", '2008'),
		(r."2009", '2009'),
		(r."2010", '2010'),
		(r."2011", '2011'),
		(r."2012", '2012'),
		(r."2013", '2013'),
		(r."2014", '2014'),
		(r."2015", '2015'),
		(r."2016", '2016'),
		(r."2017", '2017'),
		(r."2018", '2018'),
		(r."2019", '2019'),
		(r."2020", '2020'),
		(r."2021", '2021')
	) AS values(renewable_percent, year)
ORDER BY country_name, year
);

--CHECKER
SELECT id, country_name, year, renewable_percent FROM renewable_long;

-- RENEWABLES -- 
----------ENERGY PRODUCTION----------


----------ENERGY IMPORTS & OTHER MEASURES----------

----- CARBON COST -----
DROP TABLE IF EXISTS carbon_cost_long;
CREATE TEMP TABLE carbon_cost_long AS(
SELECT country_name,
	values.*,
	CONCAT (country_name, year) AS id
FROM public."cost_carbon_raw" AS cc
	CROSS JOIN LATERAL(
	VALUES
		(cc."1960", '1960'),
		(cc."1961", '1961'),
		(cc."1962", '1962'),
		(cc."1963", '1963'),
		(cc."1964", '1964'),
		(cc."1965", '1965'),
		(cc."1966", '1966'),
		(cc."1967", '1967'),
		(cc."1968", '1968'),
		(cc."1969", '1969'),
		(cc."1970", '1970'),
		(cc."1971", '1971'),
		(cc."1972", '1972'),
		(cc."1973", '1973'),
		(cc."1974", '1974'),
		(cc."1975", '1975'),
		(cc."1976", '1976'),
		(cc."1977", '1977'),
		(cc."1978", '1978'),
		(cc."1979", '1979'),
		(cc."1980", '1980'),
		(cc."1981", '1981'),
		(cc."1982", '1982'),
		(cc."1983", '1983'),
		(cc."1984", '1984'),
		(cc."1985", '1985'),
		(cc."1986", '1986'),
		(cc."1987", '1987'),
		(cc."1988", '1988'),
		(cc."1989", '1989'),
		(cc."1990", '1990'),
		(cc."1991", '1991'),
		(cc."1992", '1992'),
		(cc."1993", '1993'),
		(cc."1994", '1994'),
		(cc."1995", '1995'),
		(cc."1996", '1996'),
		(cc."1997", '1997'),
		(cc."1998", '1998'),
		(cc."1999", '1999'),
		(cc."2000", '2000'),
		(cc."2001", '2001'),
		(cc."2002", '2002'),
		(cc."2003", '2003'),
		(cc."2004", '2004'),
		(cc."2005", '2005'),
		(cc."2006", '2006'),
		(cc."2007", '2007'),
		(cc."2008", '2008'),
		(cc."2009", '2009'),
		(cc."2010", '2010'),
		(cc."2011", '2011'),
		(cc."2012", '2012'),
		(cc."2013", '2013'),
		(cc."2014", '2014'),
		(cc."2015", '2015'),
		(cc."2016", '2016'),
		(cc."2017", '2017'),
		(cc."2018", '2018'),
		(cc."2019", '2019'),
		(cc."2020", '2020'),
		(cc."2021", '2021')
	) AS values(carbon_cost, year)
ORDER BY country_name, year
);


--CHECKER
SELECT id, country_name, year, carbon_cost FROM carbon_cost_long;
-- CARBON COST --

----- ELECTRICITY ACCESS -----

DROP TABLE IF EXISTS access_long;
CREATE TEMP TABLE access_long AS(
SELECT country_name,
	values.*,
	CONCAT (country_name, year) AS id
FROM public."electricity_access_raw" AS a
	CROSS JOIN LATERAL(
	VALUES
		(a."1960", '1960'),
		(a."1961", '1961'),
		(a."1962", '1962'),
		(a."1963", '1963'),
		(a."1964", '1964'),
		(a."1965", '1965'),
		(a."1966", '1966'),
		(a."1967", '1967'),
		(a."1968", '1968'),
		(a."1969", '1969'),
		(a."1970", '1970'),
		(a."1971", '1971'),
		(a."1972", '1972'),
		(a."1973", '1973'),
		(a."1974", '1974'),
		(a."1975", '1975'),
		(a."1976", '1976'),
		(a."1977", '1977'),
		(a."1978", '1978'),
		(a."1979", '1979'),
		(a."1980", '1980'),
		(a."1981", '1981'),
		(a."1982", '1982'),
		(a."1983", '1983'),
		(a."1984", '1984'),
		(a."1985", '1985'),
		(a."1986", '1986'),
		(a."1987", '1987'),
		(a."1988", '1988'),
		(a."1989", '1989'),
		(a."1990", '1990'),
		(a."1991", '1991'),
		(a."1992", '1992'),
		(a."1993", '1993'),
		(a."1994", '1994'),
		(a."1995", '1995'),
		(a."1996", '1996'),
		(a."1997", '1997'),
		(a."1998", '1998'),
		(a."1999", '1999'),
		(a."2000", '2000'),
		(a."2001", '2001'),
		(a."2002", '2002'),
		(a."2003", '2003'),
		(a."2004", '2004'),
		(a."2005", '2005'),
		(a."2006", '2006'),
		(a."2007", '2007'),
		(a."2008", '2008'),
		(a."2009", '2009'),
		(a."2010", '2010'),
		(a."2011", '2011'),
		(a."2012", '2012'),
		(a."2013", '2013'),
		(a."2014", '2014'),
		(a."2015", '2015'),
		(a."2016", '2016'),
		(a."2017", '2017'),
		(a."2018", '2018'),
		(a."2019", '2019'),
		(a."2020", '2020'),
		(a."2021", '2021')
	) AS values(access_percent, year)
ORDER BY country_name, year
);


--CHECKER
SELECT id, country_name, year, access_percent FROM access_long;
-- ELECTRICITY ACCESS --

----- POWER CONSUMPTION -----

DROP TABLE IF EXISTS power_long;
CREATE TEMP TABLE power_long AS(
SELECT country_name,
	values.*,
	CONCAT (country_name, year) AS id
FROM public."power_consumption_raw" AS p
	CROSS JOIN LATERAL(
	VALUES
		(p."1960", '1960'),
		(p."1961", '1961'),
		(p."1962", '1962'),
		(p."1963", '1963'),
		(p."1964", '1964'),
		(p."1965", '1965'),
		(p."1966", '1966'),
		(p."1967", '1967'),
		(p."1968", '1968'),
		(p."1969", '1969'),
		(p."1970", '1970'),
		(p."1971", '1971'),
		(p."1972", '1972'),
		(p."1973", '1973'),
		(p."1974", '1974'),
		(p."1975", '1975'),
		(p."1976", '1976'),
		(p."1977", '1977'),
		(p."1978", '1978'),
		(p."1979", '1979'),
		(p."1980", '1980'),
		(p."1981", '1981'),
		(p."1982", '1982'),
		(p."1983", '1983'),
		(p."1984", '1984'),
		(p."1985", '1985'),
		(p."1986", '1986'),
		(p."1987", '1987'),
		(p."1988", '1988'),
		(p."1989", '1989'),
		(p."1990", '1990'),
		(p."1991", '1991'),
		(p."1992", '1992'),
		(p."1993", '1993'),
		(p."1994", '1994'),
		(p."1995", '1995'),
		(p."1996", '1996'),
		(p."1997", '1997'),
		(p."1998", '1998'),
		(p."1999", '1999'),
		(p."2000", '2000'),
		(p."2001", '2001'),
		(p."2002", '2002'),
		(p."2003", '2003'),
		(p."2004", '2004'),
		(p."2005", '2005'),
		(p."2006", '2006'),
		(p."2007", '2007'),
		(p."2008", '2008'),
		(p."2009", '2009'),
		(p."2010", '2010'),
		(p."2011", '2011'),
		(p."2012", '2012'),
		(p."2013", '2013'),
		(p."2014", '2014'),
		(p."2015", '2015'),
		(p."2016", '2016'),
		(p."2017", '2017'),
		(p."2018", '2018'),
		(p."2019", '2019'),
		(p."2020", '2020'),
		(p."2021", '2021')
	) AS values(power_consumption, year)
ORDER BY country_name, year
);


--CHECKER
SELECT id, country_name, year, power_consumption FROM power_long;


-- POWER CONSUMPTION --


----- ENERGY IMPORTS -----

DROP TABLE IF EXISTS imports_long;
CREATE TEMP TABLE imports_long AS(
SELECT country_name,
	values.*,
	CONCAT (country_name, year) AS id
FROM public."import_percentage_raw" AS i
	CROSS JOIN LATERAL(
	VALUES
		(i."1960", '1960'),
		(i."1961", '1961'),
		(i."1962", '1962'),
		(i."1963", '1963'),
		(i."1964", '1964'),
		(i."1965", '1965'),
		(i."1966", '1966'),
		(i."1967", '1967'),
		(i."1968", '1968'),
		(i."1969", '1969'),
		(i."1970", '1970'),
		(i."1971", '1971'),
		(i."1972", '1972'),
		(i."1973", '1973'),
		(i."1974", '1974'),
		(i."1975", '1975'),
		(i."1976", '1976'),
		(i."1977", '1977'),
		(i."1978", '1978'),
		(i."1979", '1979'),
		(i."1980", '1980'),
		(i."1981", '1981'),
		(i."1982", '1982'),
		(i."1983", '1983'),
		(i."1984", '1984'),
		(i."1985", '1985'),
		(i."1986", '1986'),
		(i."1987", '1987'),
		(i."1988", '1988'),
		(i."1989", '1989'),
		(i."1990", '1990'),
		(i."1991", '1991'),
		(i."1992", '1992'),
		(i."1993", '1993'),
		(i."1994", '1994'),
		(i."1995", '1995'),
		(i."1996", '1996'),
		(i."1997", '1997'),
		(i."1998", '1998'),
		(i."1999", '1999'),
		(i."2000", '2000'),
		(i."2001", '2001'),
		(i."2002", '2002'),
		(i."2003", '2003'),
		(i."2004", '2004'),
		(i."2005", '2005'),
		(i."2006", '2006'),
		(i."2007", '2007'),
		(i."2008", '2008'),
		(i."2009", '2009'),
		(i."2010", '2010'),
		(i."2011", '2011'),
		(i."2012", '2012'),
		(i."2013", '2013'),
		(i."2014", '2014'),
		(i."2015", '2015'),
		(i."2016", '2016'),
		(i."2017", '2017'),
		(i."2018", '2018'),
		(i."2019", '2019'),
		(i."2020", '2020'),
		(i."2021", '2021')
	) AS values(imports_percent, year)
ORDER BY country_name, year
);


--CHECKER
SELECT id, country_name, year, imports_percent FROM imports_long;


-- ENERGY IMPORTS --


----- POPULATION -----

DROP TABLE IF EXISTS population_long;
CREATE TEMP TABLE population_long AS(
SELECT country_name,
	values.*,
	CONCAT (country_name, year) AS id
FROM public."population_raw" AS p
	CROSS JOIN LATERAL(
	VALUES
		(p."1960", '1960'),
		(p."1961", '1961'),
		(p."1962", '1962'),
		(p."1963", '1963'),
		(p."1964", '1964'),
		(p."1965", '1965'),
		(p."1966", '1966'),
		(p."1967", '1967'),
		(p."1968", '1968'),
		(p."1969", '1969'),
		(p."1970", '1970'),
		(p."1971", '1971'),
		(p."1972", '1972'),
		(p."1973", '1973'),
		(p."1974", '1974'),
		(p."1975", '1975'),
		(p."1976", '1976'),
		(p."1977", '1977'),
		(p."1978", '1978'),
		(p."1979", '1979'),
		(p."1980", '1980'),
		(p."1981", '1981'),
		(p."1982", '1982'),
		(p."1983", '1983'),
		(p."1984", '1984'),
		(p."1985", '1985'),
		(p."1986", '1986'),
		(p."1987", '1987'),
		(p."1988", '1988'),
		(p."1989", '1989'),
		(p."1990", '1990'),
		(p."1991", '1991'),
		(p."1992", '1992'),
		(p."1993", '1993'),
		(p."1994", '1994'),
		(p."1995", '1995'),
		(p."1996", '1996'),
		(p."1997", '1997'),
		(p."1998", '1998'),
		(p."1999", '1999'),
		(p."2000", '2000'),
		(p."2001", '2001'),
		(p."2002", '2002'),
		(p."2003", '2003'),
		(p."2004", '2004'),
		(p."2005", '2005'),
		(p."2006", '2006'),
		(p."2007", '2007'),
		(p."2008", '2008'),
		(p."2009", '2009'),
		(p."2010", '2010'),
		(p."2011", '2011'),
		(p."2012", '2012'),
		(p."2013", '2013'),
		(p."2014", '2014'),
		(p."2015", '2015'),
		(p."2016", '2016'),
		(p."2017", '2017'),
		(p."2018", '2018'),
		(p."2019", '2019'),
		(p."2020", '2020'),
		(p."2021", '2021')
	) AS values(population, year)
ORDER BY country_name, year
);


--CHECKER
SELECT id, country_name, year, population FROM population_long;


-- POPULATION --



-- Joining tables for master energy production table 
DROP TABLE IF EXISTS energy;
CREATE TEMP TABLE energy AS (
SELECT
	coal_long.id, 
	coal_long.country_name,
	coal_long.year,
	coal_percent,
	hydro_percent,
	natural_gas_percent,
	nuclear_percent,
	oil_percent, 
	renewable_percent, 
	carbon_cost,
	access_percent, 
	power_consumption, 
	imports_percent,
	population
FROM coal_long

INNER JOIN hydro_long
ON coal_long.id = hydro_long.id

INNER JOIN natural_gas_long 
ON coal_long.id = natural_gas_long.id

INNER JOIN nuclear_long
ON coal_long.id = nuclear_long.id

INNER JOIN oil_long
ON coal_long.id = oil_long.id

INNER JOIN renewable_long
ON coal_long.id = renewable_long.id
	
INNER JOIN carbon_cost_long
ON coal_long.id = carbon_cost_long.id
	
INNER JOIN access_long
ON coal_long.id = access_long.id
	
INNER JOIN power_long
ON coal_long.id = power_long.id
	
INNER JOIN imports_long
ON coal_long.id = imports_long.id
	
INNER JOIN population_long
ON coal_long.id = population_long.id	
	
);


DROP TABLE IF EXISTS energy_output;
CREATE TABLE energy_output AS(
SELECT *,
SUM(coal_percent + hydro_percent + natural_gas_percent + nuclear_percent + oil_percent + renewable_percent) AS sum_check	
FROM energy
GROUP BY energy.id, energy.country_name, energy.year, energy.coal_percent, energy.hydro_percent, energy.natural_gas_percent, energy.nuclear_percent, energy.oil_percent, energy.renewable_percent, energy.carbon_cost, energy.access_percent, energy.power_consumption, energy.imports_percent, energy.population
);

SELECT * FROM energy_output
order by country_name asc, year asc;
--------------------------------


----- PERCENTAGES TO kWh/capita -----

DROP TABLE IF EXISTS energy_output_kWhpcapita;
CREATE TABLE energy_output_kWhpcapita AS(
SELECT id, 
	country_name,
	year,
	((coal_percent/100)*power_consumption)::decimal (16,3) AS coal_kWhpcapita,
	((hydro_percent/100)*power_consumption)::decimal (16,3) AS hydro_kWhpcapita,
	((natural_gas_percent/100)*power_consumption)::decimal (16,3) AS natural_gas_kWhpcapita,
	((nuclear_percent/100)*power_consumption)::decimal (16,3) AS nuclear_kWhpcapita,
	((oil_percent/100)*power_consumption)::decimal (16,3) AS oil_kWhpcapita,
	((renewable_percent/100)*power_consumption)::decimal (16,3) AS renewable_kWhpcapita,
	((imports_percent/100)*power_consumption)::decimal (16,3) AS imports_kWhpcapita,
	((sum_check/100)*power_consumption):: decimal (16,3) AS sum_check_kWhpcapita

FROM energy_output);

SELECT * FROM energy_output_kWhpcapita
order by country_name asc, year asc;
 ----------
 
 
 
----- PERCENTAGES TO kWh -----

DROP TABLE IF EXISTS energy_output_kWh;
CREATE TABLE energy_output_kWh AS(
SELECT id, 
	country_name,
	year,
	((coal_percent/100)*power_consumption*population)::decimal (16,3) AS coal_kWh,
	((hydro_percent/100)*power_consumption*population)::decimal (16,3) AS hydro_kWh,
	((natural_gas_percent/100)*power_consumption*population)::decimal (16,3) AS natural_gas_kWh,
	((nuclear_percent/100)*power_consumption*population)::decimal (16,3) AS nuclear_kWh,
	((oil_percent/100)*power_consumption*population)::decimal (16,3) AS oil_kWh,
	((renewable_percent/100)*power_consumption*population)::decimal (16,3) AS renewable_kWh,
	((imports_percent/100)*power_consumption*population)::decimal (16,3) AS imports_kWh,
	((sum_check/100)*power_consumption*population):: decimal (18,3) AS sum_check_kWh

FROM energy_output);

SELECT * FROM energy_output_kWh
order by country_name asc, year asc

```

</Details>


*This code includes a section relating to net energy imports that which has not been included in the final visualisations. Net imports are necessarily coverd in the production values of other countries*

*In addition, while this code runs effectively, the same results of this code can surely be achieved more efficiently due to the frequent repetition in the wide/long format transformation*


#### Power BI

Power BI can connect directly to the PostgreSQL database and tables created by running the PostgreSQL code. Once this connection has been established additional data preparation is required using the Power BI Power Query Editor.

Following the amalgamation of all the energy sources data into one table this again needs to be trandformed from "wide" to "long" format. This can be achieved by unpivoting the data allowing the energy source to be a more easily visualtsed variable.

The initial "wide" format can be seen here:

![Power Query Wide](https://github.com/jor-rainey/ImagesforReadMe/blob/main/Energy%20Project%20Screenshots/Power%20Query%20Wide.png)

And after transformation the "long" format can be seen here:
![Power Query Long](https://github.com/jor-rainey/ImagesforReadMe/blob/main/Energy%20Project%20Screenshots/Power%20Query%20Long.png)


The final few adjustments within the Power Query Editor included filtering out non-countries from the country list (MIddle Income, OEDC, EU, etc.), the additional column indicating whether or not the energy source is "Green", tweaking of units and finally a merge of the queries to bring in population and power consumption/capita. The addition of which can be seen below:

![Power Query Final](https://github.com/jor-rainey/ImagesforReadMe/blob/main/Energy%20Project%20Screenshots/Power%20Query%20Post%20Merge.png)


### Data Analysis

The final Power BI file can be found with full interactivity [here](https://github.com/jor-rainey/Portfolio_Projects/blob/main/Energy_Production_Project/Energy%20Production%20Project.pbix)

However the screenshot below can be used to compare the energy production sources of Germany and France since 1960 as well as giving context of population and power consumption per capita.

![Final Dashboard](https://github.com/jor-rainey/ImagesforReadMe/blob/main/Energy%20Project%20Screenshots/Dashboard%20Final.png)

The screenshot has France and Germany selected whcih are interesting and appropriate countires to compare. This is due to the many similarities in terms of; geography, politics, population and energy consumption.
However clear differences in energy production can still be seen. Most notably both countries have Nuclear power take up an increasing proportion of energy production in the 1970s. In France this continues through the 1980s to level out at around 75% of total production leaving a broadly stable energy production make up for the past 25 years. 
Politically in Germany nuclear power faced more difficulties and after similar initial investment as France the proportion of nuclear energy tapered away to around a mere 15% for the most recent available year. In lieu of nuclear power Germany can be seen to be relying more heavilly on coal and has a significant recent uptick in renewable energy.

As an extension to this project it could be interesting to delve deeper into the subcategories of Renewable Energy (e.g. solar, wind, etc.). However data for this and for years more recent that 2014, where more intersting trends are likely found, was less readilly available for such a wide range of countries.
If particualr countries were of interest it would also be possible to annotate graphs with potential causes of the change in energy production sources. This could range from technological advances to geo-political or policy changes particular to the country in question. 

### Data Visualisation

For this dashboard, created with ease of two country comparison in mind, it was important to show as much useful clear information as possible on one page in a side by side format. For this a neutral background colour scheme is used here and instructions are given to the user for selecting countries to compare to ease usability.
The formatting of each graph for each country is also important so that the years on the x axis line up vertically across the dashboard, again to ease the comparison across time.

The main intention of each of the graphs is to show the overview and long term trends of each variable over time. However more detailed figures can be found in the tooltips of all graphs, and the cross filtering aides in the comparison of the particualr selected variable.

The colour schemes of the graphs have been intentionally chosen to aid in the easy identification of each energy source (Renewable - Green, Hydro - Blue, etc.). This helps to mitigate against any crowding of the graphs especially 'Domestic Energy Production Composition kWh'. The approximate limit for useful numbers of variables shown on a line graph is being reached in this example.

![Dashboard Close up](https://github.com/jor-rainey/ImagesforReadMe/blob/main/Energy%20Project%20Screenshots/Close%20up.png)

An important aspect of building the dashboard was to ensure that the y-axis scales of all graphs of both countries remain the same. This ensured that when comparing two countries with vastly different energy production amounts the difference between them was obviously apparent to the viewer of the dashboard. While it should be anticipated that, for example, China and Cameroon are not terribly useful countries to compare due to their massive population and energy consumption differences. If the y-axis scales of energy production are allowd to act independantly for each country there is scope for confusion or incorrect conclusions to be drawn.

![Bad Example](https://github.com/jor-rainey/ImagesforReadMe/blob/main/Energy%20Project%20Screenshots/Bad%20Example.png)


