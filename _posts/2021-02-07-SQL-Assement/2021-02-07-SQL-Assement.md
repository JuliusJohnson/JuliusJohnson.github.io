---
title: SQL Interview Practice -- Reviewing a SQL Assesment
date: 2021-02-07 12:00:00 -05:00
tags: [SQL, Analytics, Google Cloud Platform, Big Query]
description: A walkthough of a SQL assesment I did as part of an interview in December of 2020. 
---

## Introduction

Through the back half of 2020, I was doing some job interviews, and I took many SQL assessments and tests. It opened my mind to what I didn’t know! For example, when I first started interviewing, I had no idea what DQL(Data Query Language) or DML(Data Manipulation Language) meant. 

Outside of the absolute basics of SQL, I’m self-taught. Finding solutions to my daily problems is much more important than making sure that I knew the language to communicate my knowledge. 

The following question assessment is from an actual company, which shall remain nameless out of respect. I didn’t get the job, they said my SQL needed some work, so I decided to work on it. There were some pretty interesting questions, and I hope to present my solutions here.

_<span style="text-decoration:underline;">Notes:</span>_

_[[Datasets, Results, and Queries can be found on GitHub]](https://github.com/JuliusJohnson/SQL-Assesment-Dec-2020)_

_<span style="text-decoration:underline;">All queries were written and executed in Google Cloud Platform’s (GCP) BigQuery</span>_


### Question 1: Data Integrity & Cleanup

Alphabetically list all the country codes in the continent map table that appear more than once. Countries with no country code make them display as "N/A" and display them first in the list.


```
SELECT
CASE WHEN country_code IS NULL THEN 'N/A' ELSE country_code END Countries_w_Multiple_Occurances
--,COUNT(CASE WHEN country_code IS NULL THEN 'N/A' ELSE country_code END) Number_of_occurances [This Line was used to check results. I imagine the final list should be a single array]

FROM `portfolio-304103.sql_challenge.map`

GROUP BY Countries_w_Multiple_Occurances
HAVING COUNT(Countries_w_Multiple_Occurances) > 1
ORDER BY CASE WHEN Countries_w_Multiple_Occurances = 'N/A' THEN 0 else 1 end -- Never knew could apply conditional logic to an ORDER BY Clause!
```


For the most part, this query is fairly straightforward and relatively elementary. A simple understanding of how each statement works will get you pretty close to a solution.

What stumped me during my initial attempt, which I tried to solve using joins, was how to display NULL values as ‘N/A’ and force them to be at the top. Forcing ordering/position/rank is a recurring question on SQL assessments. However, I have never done this in my professional gig.

It turns out you may apply conditional clauses to ORDER BY statements! My previous study and studies since taking this assessment have never mentioned this. When I found this solution on Stackoverflow, I double-checked W3C and some other resources to make sure I wasn’t paying attention; they never mentioned this. Once I figured out the “trick,” the problem became trivial.

[[Results]](https://github.com/JuliusJohnson/SQL-Assesment-Dec-2020/blob/main/Results/Question_1_Results.csv)


### Question 2: List the Top 10 Countries by year over year % GDP per capita growth between 2011 & 2012.

The final product should include columns for Rank, Country Name, Country Code, Continent, and Growth Percent

_<span style="text-decoration:underline;">Note: The question uses all tables provided</span>_

This query was super fun. I have two different approaches to this query. 

Original attempt:


```
SELECT
 RANK() OVER (ORDER BY Growth_Percent DESC ) AS rank,
 country_name,
 country_code,
 continents_name as continent,
 growth_percent
FROM (
 SELECT
   country.country_name,
   capita.country_code,
   continent.continents_name
   ,ROUND(SAFE_DIVIDE(gdp_per_capita,LAG(gdp_per_capita, 1) OVER (ORDER BY capita.country_code, capita.year))-1,4) AS growth_percent
   ,capita.year
 FROM
   `salesloft-123.salesloft.per_capita` AS capita
 LEFT JOIN `salesloft-123.salesloft.countries` AS country ON country.country_code = capita.country_code
 LEFT JOIN `salesloft-123.salesloft.map` AS map ON map.country_code = capita.country_code
 LEFT JOIN `salesloft-123.salesloft.continents` AS continent ON continent.continents_code = map.continent_code
 WHERE year IN(2011, 2012)
   ) AS Growth_task
WHERE year = 2012
 LIMIT 10
```


Essentially, I calculate the GDP growth of every country by utilizing the ‘LAG’ windowing function. The Lag function allows the user to access a row of data at a specific offset. I wouldn’t say this is orthodox, but I was trying to understand window functions better at the time of the assessment, and this solution allowed me to minimize the number of joins I performed.

Also, shoutout to Google Big Query for implementing a ‘SAFE_DIVIDE’ function!

Attempt 2:


```
SELECT
rank() OVER (ORDER BY PctGrowth DESC) Rank
,country.country_name Country_Name
,base.country_code Country_Code
,cont.continents_name Continent_Name
,PctGrowth Growth_Percent
FROM(
select
p.country_code
,(c.gdp_per_capita / p.gdp_per_capita)-1 PctGrowth
--,p.gdp_per_capita gdp2011 [Used to check results]
--,c.gdp_per_capita gdp2012 [Used to check results]
from `portfolio-304103.sql_challenge.per_capita` as p
FULL OUTER JOIN (
select
country_code
,gdp_per_capita
from `portfolio-304103.sql_challenge.per_capita`
WHERE year = 2012
) AS c on c.country_code = p.country_code
WHERE p.YEAR = 2011
) base
LEFT JOIN `portfolio-304103.sql_challenge.countries` as country on country.country_code = base.country_code
LEFT JOIN `portfolio-304103.sql_challenge.map` as map on map.country_code = base.country_code
INNER JOIN `portfolio-304103.sql_challenge.continents` as cont on cont.continents_code = map.continent_code
ORDER BY pctGrowth DESC
LIMIT 10 -- Alternatively if we want to be sure this was correct we could add a filter that finds where 'Rank >= 10'
--This method would probably be slower on extremely large data sets, but more robust with more dynamic data.
```


Although I’m confident in the correctness of my solution, I wanted to produce a query that was more accessible to understand. 

[[Result]](https://github.com/JuliusJohnson/SQL-Assesment-Dec-2020/blob/main/Results/Question_2_Results.csv)


### Question 3. For the year 2012, compare the percentage share of GDP Per Capita for the following regions: North America (NA), Europe (EU), and the Rest of the World. Your result should look something like:


```
North America  | Europe | Rest of the World
------ | ------ | -------------
X%  | Y%  | Z%
```


Question 3 isn’t too tricky if you don’t apply an actual transposition of the data. I googled some solutions, and some things regarding cursors were suggested, but those aren’t typically best practices, and we are dealing with static data. 

I was incapable of solving this problem when I initially took this assessment. I didn’t realize you could specify multiple tables within the ‘FROM’ statement, and selecting single values or single columns wasn’t something I embraced in my code(Remember, I don’t have any formal training). I did set embark on a quest to formalize my knowledge and read through [Sams Teach Yourself SQL in 10 Minutes](https://www.amazon.com/Sams-Teach-Yourself-SQL-Minutes-ebook/dp/B004KAB8YS). This book was tremendously helpful in filling in many of the gaps I had in my SQL understanding.


```
SELECT
*
FROM
(
SELECT
AVG(gdp_per_capita) AS North_America -- 'GROUP BY' statement note needed because we are returning a single value
FROM `portfolio-304103.sql_challenge.per_capita` as base
LEFT JOIN `portfolio-304103.sql_challenge.map` as map on map.country_code = base.country_code
INNER JOIN `portfolio-304103.sql_challenge.continents` as cont on cont.continents_code = map.continent_code
WHERE year = 2012 AND cont.continents_name = 'North America'
) NA,
(
SELECT
AVG(gdp_per_capita) AS Europe -- 'GROUP BY' statement note needed because we are returning a single value
FROM `portfolio-304103.sql_challenge.per_capita` as base
LEFT JOIN `portfolio-304103.sql_challenge.map` as map on map.country_code = base.country_code
INNER JOIN `portfolio-304103.sql_challenge.continents` as cont on cont.continents_code = map.continent_code
WHERE year = 2012 AND cont.continents_name = 'Europe'
) EU,
(
SELECT
AVG(gdp_per_capita) AS Rest_of_the_World -- 'GROUP BY' statement note needed because we are returning a single value
FROM `portfolio-304103.sql_challenge.per_capita` as base
LEFT JOIN `portfolio-304103.sql_challenge.map` as map on map.country_code = base.country_code
INNER JOIN `portfolio-304103.sql_challenge.continents` as cont on cont.continents_code = map.continent_code
WHERE year = 2012 AND cont.continents_name != 'Europe' AND cont.continents_name != 'North America'
) Rest_of_the_World
```


[[Results]](https://github.com/JuliusJohnson/SQL-Assesment-Dec-2020/blob/main/Results/Question_3_Results.csv)


## Conclusion:

I had fun revisiting these SQL challenges with less pressure, more knowledge, and more time. I still have a lot more to learn and I’m so excited to learn more and work with people much smarter than I am. 
