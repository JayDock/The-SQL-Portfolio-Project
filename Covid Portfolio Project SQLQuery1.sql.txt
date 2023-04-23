select * from CovidDeaths_

SELECT column_name, data_type
FROM information_schema.columns
WHERE table_name = 'CovidDeaths_'

-- Here since we imported as flat file source in CSV, all the data types became varchar. We have to change them to the initial format

--Looking for total cases v/s total deaths (To find the deaths for the entire cases in %)

ALTER TABLE CovidDeaths_
ALTER COLUMN date DATE;

--ALTER TABLE CovidDeaths_
--ALTER COLUMN total_cases INT;

--ALTER TABLE CovidDeaths_
--ALTER COLUMN total_deaths FLOAT;

SELECT location, date, total_cases, new_cases, total_deaths, population
FROM CovidDeaths_

SELECT location, date, total_cases, total_deaths, (CAST(total_deaths as FLOAT) / NULLIF(CAST(total_cases as FLOAT), 0))*100 as death_ratio
FROM CovidDeaths_

-- Here the data type of the columns "total_deaths" and "total_cases" should be converted from varchar to FLOAT
-- NULLIF is used to avoid divide by zero error.

SELECT location, date, total_cases, total_deaths, (CAST(total_deaths as FLOAT) / NULLIF(CAST(total_cases as FLOAT), 0))*100 as death_ratio
FROM CovidDeaths_
WHERE location like '%kingdom%'

-- Looking at total case v/s population (To find what percent of population got covidfd)

SELECT location, date, total_cases, total_deaths, (CAST(total_deaths as FLOAT) / NULLIF(CAST(population as FLOAT), 0))*100 as population_infected_ratio
FROM CovidDeaths_

	
SELECT location, date, total_cases, total_deaths, (CAST(total_deaths as FLOAT) / NULLIF(CAST(population as FLOAT), 0))*100 as population_infected_ratio
FROM CovidDeaths_
WHERE location like '%kingdom%'


-- Looking at countries with highest infection rate compared to their population


SELECT location, 
       population, 
       MAX(total_cases) as Higest_Infection_Count, 
       (MAX(total_cases)/NULLIF(CONVERT(float, population), 0))*100 as Population_Infected_Ratio
FROM CovidDeaths_
--WHERE location like '%kingdom%'
GROUP BY location, population
ORDER BY Population_Infected_Ratio desc


SELECT location, 
       population, 
       MAX(total_cases) as Higest_Infection_Count, 
       (MAX(total_cases)/NULLIF(CONVERT(float, population), 0))*100 as Population_Infected_Ratio
FROM CovidDeaths_
WHERE location like '%kingdom%'
GROUP BY location, population
ORDER BY Population_Infected_Ratio desc

--Countries with highest death count per population

SELECT location,
MAX(total_deaths) as Total_Death_Count
FROM CovidDeaths_
WHERE TRIM(continent) <> ''
GROUP BY location
ORDER BY Total_Death_Count desc

--Here we avoided all the continents

--Breaking down by continents

--Showing continents with the higest death count per population

SELECT continent,
MAX(total_deaths) as Total_Death_Count
FROM CovidDeaths_
WHERE TRIM(continent) <> ''
GROUP BY continent
ORDER BY Total_Death_Count desc

--Global Numbers

SELECT date, SUM(CAST(new_cases as int)) as total_cases, SUM(CAST(new_deaths as int)) as total_deaths,
ROUND((SUM(CAST(new_deaths as float))/NULLIF(SUM(CAST(new_cases as float)),0))*100, 2) as death_percentage
FROM CovidDeaths_
WHERE continent is not null
GROUP BY date


-- Looking at total populations v/s vaccinations

select * 
from CovidDeaths_ deat
JOIN CovidVaccinations vacc
on deat.location = vacc.location
and deat.date = vacc.date


select deat.continent, deat.location, deat.date, deat.population, vacc.new_vaccinations
from CovidDeaths_ deat
JOIN CovidVaccinations vacc
on deat.location = vacc.location
and deat.date = vacc.date
where deat.continent is not null


-- Using CTE

WITH PopvsVac (continent, location, date, population, new_vaccinations, cumulative_vaccinations)
AS
(
    SELECT deat.continent, deat.location, deat.date, CONVERT(BIGINT, deat.population) AS population, CONVERT(BIGINT, vacc.new_vaccinations) AS new_vaccinations,
    SUM(CONVERT(BIGINT, vacc.new_vaccinations)) over (PARTITION BY deat.location ORDER BY deat.location, deat.date) 
    AS cumulative_vaccinations
    FROM CovidDeaths_ deat
    JOIN CovidVaccinations vacc
    ON deat.location = vacc.location
    AND deat.date = vacc.date
    WHERE deat.continent IS NOT NULL
)
SELECT * , (cast(cumulative_vaccinations as float)/cast(population as float))*100 as Vaccination_Percentage
FROM PopvsVac
WHERE population > 0



WITH PopvsVac (continent, location, date, population, new_vaccinations, cumulative_vaccinations)
AS
(
    SELECT deat.continent, deat.location, deat.date, CONVERT(BIGINT, deat.population) AS population, CONVERT(BIGINT, vacc.new_vaccinations) AS new_vaccinations,
    SUM(CONVERT(BIGINT, vacc.new_vaccinations)) over (PARTITION BY deat.location ORDER BY deat.location, deat.date) 
    AS cumulative_vaccinations
    FROM CovidDeaths_ deat
    JOIN CovidVaccinations vacc
    ON deat.location = vacc.location
    AND deat.date = vacc.date
    WHERE deat.continent IS NOT NULL
)
SELECT * , (cast(cumulative_vaccinations as float)/cast(population as float))*100 as Vaccination_Percentage
FROM PopvsVac
WHERE population > 0 AND location like '%kingdom%'



-- Creating view to store data for later visualisations

CREATE view VaccinatedPopulationRatio as 
SELECT deat.continent, deat.location, deat.date, CONVERT(BIGINT, deat.population) AS population, CONVERT(BIGINT, vacc.new_vaccinations) AS new_vaccinations,
    SUM(CONVERT(BIGINT, vacc.new_vaccinations)) over (PARTITION BY deat.location ORDER BY deat.location, deat.date) 
    AS cumulative_vaccinations
    FROM CovidDeaths_ deat
    JOIN CovidVaccinations vacc
    ON deat.location = vacc.location
    AND deat.date = vacc.date
    WHERE deat.continent IS NOT NULL

SELECT * FROM VaccinatedPopulationRatio


-- Tableau Tables Query


SELECT SUM(cast(new_cases as float)) as total_cases, SUM(cast(new_deaths as float)) as total_deaths,
ROUND(SUM(cast(new_deaths as float))/NULLIF(SUM(CAST(new_cases as float)),0)*100, 2) as DeathPercentage
FROM CovidDeaths_
WHERE continent is not null


SELECT continent, SUM(cast(new_deaths as int)) as TotalDeathCount
FROM CovidDeaths_
WHERE continent IS NOT NULL
AND location NOT IN ('World', 'European Union', 'International')
GROUP BY continent
ORDER BY TotalDeathCount DESC


SELECT location, 
       population, 
       MAX(total_cases) as Higest_Infection_Count, 
       (MAX(total_cases)/NULLIF(CONVERT(float, population), 0))*100 as Population_Infected_Ratio
FROM CovidDeaths_
--WHERE location like '%kingdom%'
GROUP BY location, population
ORDER BY Population_Infected_Ratio desc


SELECT location, population, date, 
       MAX(total_cases) as HighestInfectionCount, 
       MAX((total_cases/NULLIF(CONVERT(float, population), 0))*100) as PercentPopulationInfected
FROM CovidDeaths_
--Where location like '%states%'
GROUP BY location, population, date
ORDER BY PercentPopulationInfected desc



















