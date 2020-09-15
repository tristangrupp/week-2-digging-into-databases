# Homework 1

**Due: Sept 15, 2020 by 11:59pm ET**

## Datasets

* Indego Bikeshare data station status data
* Indego Trip data
  - Q2 2020
  - Q2 2019

All data is available from [Indego's Data site](https://www.rideindego.com/about/data/). Feel free to use the [Station Data from last week](https://raw.githubusercontent.com/MUSA-509/week-1-introductions/master/data/indego_station_status.geojson) if it is not already in your account.

Load all three datasets into your CARTO account.

## 1. How many bike trips in Q2 2019?


```SQL
SELECT count(*)
FROM tristangrupp.indego_trips_2019_q2
```

**Result:** 206354

## 2. What is the percent change in trips in Q2 2020 as compared to Q2 2019?

Using only the table from Q2 2020 and the number calculated in the previous question, find the percent change of number of trips in Q2 2020 as compared to 2019. Remember you can do calculations in the select clause.
 
```SQL
SELECT ((206354 - COUNT(*))/206354) as change
FROM tristangrupp.indego_trips_2020_q2
```

**Result:**
For whatever reason I get a result of 0. This is how I would do it. Something is awry with the divide by 206354. 

__Bonus: If you want to get fancier here, you can cast the result to a string and concatenate a `'%'` to the end. For example, `(10 + 3.2)::text || '%' AS perc_change`. This uses the type casting (number to string) and string concatenation operator (`||`, double pipes) that's essentially a `+` for strings.__

## 3. What are the average durations of each year?

**Average duration of a trip for 2019.**

```SQL
SELECT AVG(duration)
FROM tristangrupp.indego_trips_2019_q2
```
**Result:**
23.67

**Average duration of a trip for 2020.**

```SQL
SELECT AVG(duration)
FROM tristangrupp.indego_trips_2020_q2
```
**Result:**
39.23

**What do you notice about the difference in trip lengths? Give a few explanations for why there could be a difference here.**

**Answer:**
The average trip length increased from 2019 to 2020.
As people get more used to using the service and bike lanes improve across Philly, people become more comfortable taking long trips. More stations may have been added as well in areas farther from Center City, where the first stations would likely have been set up first. The larger coverage of bike stations would increase the average trip length. People are spending more time outdoors than last year because of Covid. Traveling to green spaces would be longer commutes than other, more nearby recreation that are closed.

## 4. What is the longest duration trip?

```SQL
SELECT MAX(plan_duration)
FROM tristangrupp.indego_trips_2019_q2

```
**Result:**
365 days

**Why are there so many trips of this duration?**

**Answer:**
People purchase year-long Indego subscriptions. 

## 5. How many trips were shorter than 10 minutes?

```SQL
SELECT COUNT(*)
FROM tristangrupp.indego_trips_2019_q2
WHERE duration < 10
```

**Result:**
74958

## 6. How many trips started on one day and ended in the next?

Hint 1: date strings can be parsed using the text type to datetime type conversion function [`to_timestamp`](https://www.postgresql.org/docs/12/functions-formatting.html). See the section on [Template Patterns for Date/Time Formatting](https://www.postgresql.org/docs/12/functions-formatting.html#FUNCTIONS-FORMATTING-DATETIME-TABLE) for options on choosing the right string format. The 2020 dataset has the timestamp in a slightly unusual format so you need to tell PostgreSQL how to parse it. The 2019 data should already be in a good format.

Hint 2: Days of the month can be retrieved from a timestamp using the [EXTRACT](https://www.postgresql.org/docs/12/functions-datetime.html#FUNCTIONS-DATETIME-EXTRACT) function. See also some of the follow alongs from the Lecture in week 2.

```SQL
SELECT COUNT (*)
  FROM
(
  	SELECT
  	EXTRACT(DAY from start_time) as date_day,
  	EXTRACT(DAY from end_time) as date_day2
  FROM tristangrupp.indego_trips_2019_q2
  ) as innertable
WHERE date_day2 > date_day
```

**Result:**
1692 for longer than 1 day trips
1600 for next day, with different WHERE statement: WHERE (date_day2 - date_day) = 1


## 7. Give the five most popular stations between 7am and 10am.

Using the Q2 2019 trip data, give the five most popular stations.

Hint: Use the `EXTRACT` function to get the hour of the day from the timestamp.

```SQL
-- Enter your SQL query here

SELECT COUNT (*)
  FROM
(
  	SELECT
  	EXTRACT(HOUR from start_time) as date_hour,
  	EXTRACT(HOUR from end_time) as date_hour2
  FROM tristangrupp.indego_trips_2019_q2
  ) as innertable
WHERE date_hour BETWEEN 7 and 10
GROUP BY start_station
ORDER BY start_stationcount DSC
LIMIT 5
```

**Result:**
I want to come back to this one. I am having trouble applying other things (where, group, order) to column aliases.
When I used an inner table in the last question, it worked for the where clause. The additional group by to find stations isn't working. Things stop working after the WHERE clause above. My logic here is: get count of rides between 7-10am, find stations between 7-10 am, group by the start_station, order descending, limit to the 5 highest values. 

## 8. List all the passholder types and number of trips for each.

In one query, give a list of all `passholder_type` options and the number of trips taken by `passholder_type`.

```SQL
SELECT passholder_type, COUNT(passholder_type)
FROM tristangrupp.indego_trips_2019_q2
GROUP BY passholder_type
```

**Result:**
(Day Pass ... 34197)
(Indego30 ... 133344)
(Indego365 ... 37843)
(IndegoFlex ... 851)
(NULL ... 35)
(Walk-up ... 84)

## 9. Using the station status dataset, find the distance in meters of all stations from Meyerson Hall.

```SQL
SELECT
    name,
    ST_Distance(
      the_geom::geography,
      ST_SetSRID(ST_MakePoint(-75.19265679, 39.9522405), 4326)::geography
    ) as distance
FROM tristangrupp.indego_station_status
ORDER BY the_geom::geography <-> ST_SetSRID(ST_MakePoint(-75.19265679, 39.9522405), 4326)::geography
```

Don't worry about listing the full result, just give the query.

## 10. What is the average distance (in meters) of all stations from Meyerson Hall?

```SQL
SELECT
    AVG(
    ST_Distance(
      the_geom::geography,
      ST_SetSRID(ST_MakePoint(-75.19265679, 39.9522405), 4326)::geography
      )
    ) as avgdistance
FROM tristangrupp.indego_station_status
```

**Result:**
2844.78

## 11. How many stations are within 1km of Meyerson Hall?

```SQL
SELECT COUNT (*)
  FROM
(
SELECT
    name,
    ST_Distance(
      the_geom::geography,
      ST_SetSRID(ST_MakePoint(-75.19265679, 39.9522405), 4326)::geography
    ) as distance
FROM tristangrupp.indego_station_status
    ) as innertable
WHERE distance < 1000
```

**Result:**
14

## 12. How many stations are within 2km of Meyerson Hall?

```SQL

SELECT COUNT (*)
  FROM
(
SELECT
    name,
    ST_Distance(
      the_geom::geography,
      ST_SetSRID(ST_MakePoint(-75.19265679, 39.9522405), 4326)::geography
    ) as distance
FROM tristangrupp.indego_station_status
    ) as innertable
WHERE distance < 2000
```

**Result:**
44

## 13. Which station is furthest from Meyerson Hall?

Write a query that returns only one line, and only gives the station id, station name, and distance from Meyerson Hall.

```SQL
SELECT
	id,
    name,
    ST_Distance(
      the_geom::geography,
      ST_SetSRID(ST_MakePoint(-75.19265679, 39.9522405), 4326)::geography
    ) as distance
FROM tristangrupp.indego_station_status
ORDER BY the_geom::geography <-> ST_SetSRID(ST_MakePoint(-75.19265679, 39.9522405), 4326)::geography DESC
LIMIT 1
```

**Result:**
ID: 3183 --- Name: 15th & Kitty Hawk --- Distance: 7049.17

## 14. Which station is closest to Meyerson Hall?

Write a query that returns only one line, and only gives the station id, station name, and distance from Meyerson Hall.


```SQL
SELECT
	id,
    name,
    ST_Distance(
      the_geom::geography,
      ST_SetSRID(ST_MakePoint(-75.19265679, 39.9522405), 4326)::geography
    ) as distance
FROM tristangrupp.indego_station_status
ORDER BY the_geom::geography <-> ST_SetSRID(ST_MakePoint(-75.19265679, 39.9522405), 4326)::geography ASC
LIMIT 1
```

**Result:**
ID: 3208 --- Name: 34th & Spruce --- Distance: 201.73
