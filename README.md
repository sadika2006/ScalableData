# ScalableData


--a) The name and population of all cities in descending order of population. [3460]

select name, population from City 
order by population asc;

 -- b) All cities (name) that are located on both a river and a lake. [19]
 
SELECT DISTINCT City, Country, Province

FROM located

WHERE River IS NOT NULL

  AND Lake IS NOT NULL;

--select * from located;

-- c) Name and population of all German cities (country code: D) in descending order of population. [85]

select distinct C.name City, C.population from city C

where C.country= 'D'

order by C.population desc;

-- d) All languages (name) spoken in member countries of the EU. [56]

SELECT distinct L.Name

FROM Language L

--INNER JOIN Country C ON C.Code = L.Country

inner JOIN isMember M ON  L.country = M.Country 

WHERE M.Organization = 'EU' AND M.Type = 'member';



--e) All languages (name) spoken in member countries of the EU and in how many countries (of the EU) they 
-- are spoken, sorted in descending order by number of countries. [56]

SELECT  L.Name Language, count(M.Country)
FROM Language L
inner JOIN isMember M ON  L.country = M.Country 
WHERE M.Organization = 'EU' AND M.Type = 'member'
group by L.Name
order by count(M.Country) desc ;


--f) Name of the capital, population and name of the country of all capitals in which more than 30% of the 
-- country's population lives, in descending order of their number of inhabitants. [33]

select distinct C.capital, C.population Country_pop, C.name Country, City.population City_pop
from Country C
inner join City  on City.Name = C.Capital AND City.Country = C.code
where 0.3*(C.population) < City.population
order by City.population desc
;

-- g) All countries (name) with at least one mountain over 4,000 m high  [43]
select distinct C.name from country C
inner join geo_mountain GM on C.code = GM.country 
inner join mountain M on M.name = GM.mountain
where M.elevation > 4000 ;

SELECT DISTINCT C.Name
FROM Country C
INNER JOIN geo_Mountain GM ON C.Code = GM.Country 
INNER JOIN Mountain M ON GM.Mountain = M.Name
WHERE M.Elevation > 4000;

SELECT COUNT(DISTINCT Country) AS CountryCount
FROM (
    SELECT C.Code AS Country
    FROM Country C
    INNER JOIN geo_Mountain GM 
        ON C.Code = GM.Country 
    INNER JOIN Mountain M 
        ON GM.Mountain = M.Name
    WHERE M.Elevation > 4000
) AS UniqueCountries;



-- h) All countries (name) with at least one city, 
--which has more inhabitants than the capital of the country. [37]
SELECT DISTINCT C.Name
FROM Country C
JOIN City CityOther ON C.Code = CityOther.Country 
						and C.Capital != CityOther.Name
JOIN City CityCapital ON C.Capital = CityCapital.Name 
                      AND C.Code = CityCapital.Country
WHERE CityOther.Population > CityCapital.Population;

-- i) The names of all countries with cities having over one million inhabitants and 
--the respective total population of all cities with over one million inhabitants
-- of that country in order of this total population. [92]

Select Distinct C.name, --City.name, City.population
(SELECT SUM(Population) 
                 FROM City 
                 WHERE Country = C.Code AND Population > 1000000) AS Total_Population
from country C
join City on City.country = C.code
where City.population > 1000000
ORDER BY Total_Population DESC;


--j) All countries (name) bordering Germany and their total border length. [9  --solve


----fit

SELECT DISTINCT B.*, C.name AS countryName2, C1.name AS countryName1
FROM borders B
JOIN country C ON C.code = B.country2
JOIN country C1 ON C1.code = B.country1
WHERE C.name = 'Germany' OR C1.name = 'Germany';
--perfect with good assending
select C.name, A.*, c1.name as neighbour 
from (SELECT country1 AS CountryCode, country2 AS NeighborCountry
        FROM borders
        UNION ALL
        SELECT country2 AS CountryCode, country1 AS NeighborCountry
        FROM borders) A
		JOIN country C ON C.code = A.CountryCode
		JOIN country C1 ON C1.code = A.NeighborCountry
		 WHERE A.CountryCode = 'D'



--k)The names of all countries whose share of inhabitants living in the country's cities with over
--a millioninhabitants is greater than 30%, sorted in ascending order by this share. [14]

SELECT 
    Country.Name AS CountryName,
    (SUM(City.Population) / Country.Population) * 100 AS ShareOfInhabitants
FROM 
    Country
JOIN 
    City ON Country.Code = City.Country
WHERE 
    City.Population > 1000000
GROUP BY 
    Country.Name, Country.Population
HAVING 
    (SUM(City.Population) / Country.Population) * 100 > 30
ORDER BY 
    ShareOfInhabitants ASC;

--l) All countries (name) that have more lakes than mountains, in ascending order of the number of lakes. 
--Only countries that have both lakes and mountains should be considered. [19

Select  C.name as country ,count( DISTINCT M.mountain) as SumMountain , count( DISTINCT L.lake) as SumLake
from Country C
inner join Geo_mountain M on M.country = C.code
inner join geo_lake L on L.country = C.code and L.country = M.country
group by C.name
having count( DISTINCT M.mountain) < count( DISTINCT L.lake)
ORDER BY 
    SumLake ASC;

SELECT 
    C.name AS Country, 
    COUNT(DISTINCT L.Lake) AS NumberOfLakes, 
    COUNT(DISTINCT M.Mountain) AS NumberOfMountains
FROM 
    Country C
INNER JOIN 
    Geo_mountain M ON M.country = C.code
INNER JOIN 
    Geo_lake L ON L.country = C.code
GROUP BY 
    C.name
HAVING 
    COUNT(DISTINCT L.Lake) > COUNT(DISTINCT M.Mountain)
ORDER BY 
    NumberOfLakes ASC;

-- m) All countries (name) that have more neighboring countries than Germany. [3

--lets calculate how many neighbouring country germany has

SELECT DISTINCT B.*, C.name AS countryName2, C1.name AS countryName1
FROM borders B
JOIN country C ON C.code = B.country2
JOIN country C1 ON C1.code = B.country1
WHERE C.name = 'Germany' OR C1.name = 'Germany';




SELECT C.name, NC.NeighborCount
FROM country C
JOIN (
    SELECT 
        CountryCode,
        COUNT(*) AS NeighborCount
    FROM (
        SELECT country1 AS CountryCode, country2 AS NeighborCountry
        FROM borders
        UNION ALL
        SELECT country2 AS CountryCode, country1 AS NeighborCountry
        FROM borders
    ) AS AllBorders
    GROUP BY CountryCode
) AS NC ON C.code = NC.CountryCode
WHERE NC.NeighborCount > (
    SELECT COUNT(*)
    FROM (
        SELECT country1 AS CountryCode, country2 AS NeighborCountry
        FROM borders
        UNION ALL
        SELECT country2 AS CountryCode, country1 AS NeighborCountry
        FROM borders
    ) AS GermanyBorders
    WHERE CountryCode = 'D'
)
ORDER BY C.name;




--n) The city with the largest population and the country in which it is located. [1]
select 
	city.name  as city, 
	C.name as country, 
	city.population 
	from city
	inner join country C on C.code = city.country
where city.population = (select max(population) from city)
LIMIT 1;


select L.Name from Language L, isMember M
where M.Organization = 'EU' AND M.Type = 'member';



Select distinct Organization from ismember order by Organization;

Select * from Language


 
