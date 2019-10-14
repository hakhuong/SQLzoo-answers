# SQLzoo-answers
My answers for [SQLzoo](https://sqlzoo.net) tutorials questions 

## Sections:
1. [SELECT basics](#select-basics)
2. [SELECT from WORLD](#select-from-world)
3. [SELECT from NOBEL (harder questions)](#select-from-nobel)
4. [NESTED SELECT](#nested-select)
5. [SUM and COUNT](#sum-and-count)
6. [JOIN](#join)
7. [More JOIN](#more-join)
8. [Using NULL](#using-null)
9. [Self JOIN](#self-join)

## SELECT basics
Some simple queries to get you started

1.
```sql
  SELECT population FROM world
    WHERE name = 'Germany'
```
2.
```sql
  SELECT name, gdp/population FROM world
    WHERE area > 5000000
```
3.
```sql
  SELECT name, population FROM world
    WHERE name IN ('Ireland','Iceland','Denmark');
```
4.
```sql
  SELECT name, area FROM world
    WHERE area BETWEEN 200000 AND 250000
```
## SELECT from WORLD
1.
```sql
SELECT name, continent, population FROM world
```
2.
```sql
SELECT name FROM world
WHERE population>200000000
```
3.
Give the name and the per capita GDP for those countries with a population of at least 200 million
```sql
SELECT name, gdp/population as per_capita_GDP 
FROM world
WHERE population >=200000000
ORDER BY per_capita_GDP desc
```
4.
Show the name and population in millions for the countries of the continent 'South America'. Divide the population by 1000000 to get population in millions.
```sql
SELECT name, population / 1000000 
FROM world
WHERE  continent = 'South America'
```
5.
Show the name and population for France, Germany, Italy
```sql
SELECT name,population FROM world
  WHERE name IN ('France','Germany','Italy')
```
6.
Show the countries which have a name that includes the word 'United'
```sql
SELECT name FROM world
  WHERE name LIKE '%United%'
```
7.
Two ways to be big: A country is big if it has an area of more than 3 million sq km or it has a population of more than 250 million.

Show the countries that are big by area or big by population. Show name, population and area.
```sql
select name, population, area from world
  where population > 250000000 or area > 3000000
```
8.
Exclusive OR (XOR). Show the countries that are big by area or big by population but not both. Show name, population and area.
```sql
SELECT name, population, area
FROM world
WHERE area > 3000000 XOR population >250000000
```
9.
For South America show population in millions and GDP in billions both to 2 decimal places.
```sql
SELECT name, 
  ROUND(population/1000000, 2) as population_m, 
  ROUND(GDP/1000000000, 2) as GDP_b
FROM world
WHERE continent = 'South America'
```
10.
Show per-capita GDP for the trillion dollar countries to the nearest $1000.
```sql
SELECT name, ROUND(GDP/population,-3) as GDP_per_captica
FROM world
WHERE GDP >= 1000000000000
```
11. 
Show the name and capital where the name and the capital have the same number of characters.
```
SELECT name, capital
  FROM world
 WHERE LENGTH(name) = LENGTH(capital)
 ```
 
 12.
The capital of Sweden is Stockholm. Both words start with the letter 'S'.

**Show the name and the capital where the first letters of each match. Don't include countries where the name and the capital are the same word.**
 ```
 SELECT name, capital
FROM world
WHERE LEFT(name, 1) = LEFT(capital, 1) 
AND name <> capital
```
`<>` is the NOT EQUAL operation

13.
Equatorial Guinea and Dominican Republic have all of the vowels (a e i o u) in the name. They don't count because they have more than one word in the name.

**Find the country that has all the vowels and no spaces in its name.**

You can use the phrase name NOT LIKE '%a%' to exclude characters from your results.
The query shown misses countries like Bahamas and Belarus because they contain at least one 'a'
```
SELECT name
   FROM world
WHERE name LIKE '%a%' AND
  name LIKE '%e%' AND
  name LIKE '%o%' AND
  name LIKE '%i%' AND
  name LIKE '%u%' AND
  name NOT LIKE '% %'
```

## SELECT from Nobel (Harder questions
12. 
Find all details of the prize won by EUGENE O'NEILL (**escape single quote**)
```
SELECT * FROM nobel 
WHERE winner = 'Eugene O''NEILL'
```
13.
Knights in order

List the winners, year and subject where the winner starts with Sir. Show the the most recent first, then by name order.

```
SELECT winner, yr, subject FROM nobel
WHERE winner LIKE 'Sir%' 
ORDER BY yr DESC, winner
```

14.
The expression subject IN ('Chemistry','Physics') can be used as a value - it will be 0 or 1.

Show the 1984 winners and subject ordered by subject and winner name; but list Chemistry and Physics last.
**Use Boolean value to order**
```
SELECT winner, subject
  FROM nobel
 WHERE yr=1984
 ORDER BY subject IN ('Physics','Chemistry'), subject, winner
```

## Nested Select 
2.
Show the countries in Europe with a per capita GDP greater than 'United Kingdom'.
```
SELECT name from world
WHERE continent = 'Europe' 
AND gdp/population > (SELECT gdp/population 
                       FROM world WHERE name = 'United Kingdom')
```
3.

List the name and continent of countries in the continents containing either Argentina or Australia. Order by name of the country.

```
SELECT name, continent FROM world
WHERE continent IN (SELECT continent FROM world 
                    WHERE name IN('Argentina', 'Australia'))
ORDER BY name
```

4.
Which country has a population that is more than Canada but less than Poland? Show the name and the population.

```
SELECT name, population FROM world
WHERE population > (SELECT population FROM world WHERE name = 'Canada') 
AND population < (SELECT population FROM world WHERE name = 'Poland') 
```

5.
Germany (population 80 million) has the largest population of the countries in Europe. Austria (population 8.5 million) has 11% of the population of Germany.

**Show the name and the population of each country in Europe. Show the population as a percentage of the population of Germany.**
```
SELECT name, 
CONCAT(ROUND(population/(SELECT population FROM world WHERE name = 'Germany') *100,0), '%')  as pop_compared_to_Ger
FROM world
WHERE continent = 'Europe'
```

6.
Which countries have a GDP greater than every country in Europe? [Give the name only.] (Some countries may have NULL gdp values)
```
SELECT name FROM world
WHERE gdp > ALL(SELECT gdp FROM world WHERE gdp >0 AND continent = 'Europe')
```
For this question, **ALL()** is used to compare with list of value 
Condition gdp >0 is needed because it can contain NULL

7.
Find the largest country (by area) in each continent, show the continent, the name and the area:

```
SELECT continent, name, area FROM world x
  WHERE area >= ALL
    (SELECT area FROM world y
        WHERE y.continent=x.continent
          AND area>0)
```
 This is **correlated** or **synchronized sub-query**.
 
 8.
List each continent and the name of the country that comes first alphabetically.
```
SELECT continent, name FROM world as x
WHERE name = (SELECT name FROM world as y 
              WHERE x.continent = y.continent 
              ORDER BY name LIMIT 1) 
```

9. **Harder question**

Find the continents where all countries have a population <= 25000000. Then find the names of the countries associated with these continents. Show name, continent and population.
```
SELECT name, continent, population FROM world
WHERE continent <> ALL(SELECT continent FROM world
WHERE population > 25000000)
```
10.**Harder question**

Some countries have populations more than three times that of any of their neighbours (in the same continent). Give the countries and continents.
```
SELECT x.name, x.continent
FROM world AS x
WHERE x.population / 3 >  ALL (
  SELECT y.population
  FROM world AS y
  WHERE x.continent = y.continent AND x.name != y.name)
```

 
