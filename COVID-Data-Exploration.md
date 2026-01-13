## Project Overview

**Project Title**: COVID Data Exploration

This project is designed to demonstrate SQL skills and techniques typically used by data analysts to explore COVID data from vaccination to the death toll. The project involves finding insights within the COVID vaccinations and death data that can be used for visualizations and foster better business intelligence. I will be using Microsoft SQL Server to complete this project.

## Skills Used
- Converting Data Types(Cast and Convert)
- Aggregate Functions(MAX, SUM, )
- Window Functions(OVER.. Partition By)
- CTE's
- Temp Table
- Views

## Objectives

1. **Create the Database**
2. **Importing the Data**
3. **Exploring the Data**

## Project Structure

### 1. Create the Database
- Before beginning to explore the data, we must have a database to explore. To begin, I loaded up Microsoft SQL Server and connected to my server. I create a new database called "COVIDProject".
### 2. Importing the Data
- Now that we have the database setup, let's import the data. The data was taken from ourworldindata.org/covid-deaths. The dataset was downloaded as an excel file, so I took a look to see what was inside. After digging around the data in excel, I decided to split up the excel file into two separate excel sheets to create two tables to work with instead of just one. This allows me to have better visibility of the data and allows for some more flexibility when querying. 
### 3. Explore the Data
- When taking a look at all the data, I noticed that sometimes the continent column would be null while the location is filled in. When this was happening, I noticed that the data not only had the statistics for each country, but combined the statistics for each contintent as well. In order to avoid this interfering with my data exploration, I used a where statement to filter out when the continent column was null.
- Another problem I ran into while exploring the data was having the wrong data type for calculations. For example, total deaths was inputted as an nvarchar and couldn't be used in an aggregate function. I had to convert the data type using the cast expression in order to get the right results.
  
```sql

-- Looking at Total Cases vs Total Deaths
-- Shows likelihood of dying if you contract COVID in your country

SELECT Location, total_cases, total_deaths, (total_deaths/total_cases) * 100 as DeathPercentage
FROM PortfolioProject..CovidDeaths
WHERE continent is NOT NULL
ORDER BY location, date

-- Decided to take a look at US total cases vs total deaths

SELECT Location, Date, total_cases, total_deaths, (total_deaths/total_cases) * 100 as DeathPercentage
FROM PortfolioProject..CovidDeaths
WHERE Location = 'United States'
AND continent is NOT NULL
ORDER BY location, date


-- Looking at Total Cases vs Population
-- Shows percentage of population infected with COVID in your country 

SELECT Location, Date, total_cases, population, (total_cases/population) * 100 as PercentPopulationInfected
FROM PortfolioProject..CovidDeaths
WHERE continent is NOT NULL
ORDER BY location, date

-- Looking at countries with Highest Infection Rate Compared to Population

SELECT Location, Population, MAX(total_cases) as HighestInfectionCount, MAX((total_cases/population)) * 100 as InfectionRate
FROM PortfolioProject..CovidDeaths
WHERE continent is NOT NULL
GROUP BY location, population
ORDER BY InfectionRate DESC

-- Showing countries with Highest Death Count per Population

SELECT Location, Population, MAX(CAST(total_deaths as int)) as TotalDeathCount, MAX((total_deaths/population)) * 100 as DeathRate
FROM PortfolioProject..CovidDeaths 
WHERE continent is NOT NULL
GROUP BY location, population
ORDER BY TotalDeathCount DESC

-- CONTINENT NUMBERS

-- Showing continents with Highest Death Count per Population

SELECT Location, Population, MAX(CAST(total_deaths as int)) as TotalDeathCount, MAX((total_deaths/population)) * 100 as DeathRate
FROM PortfolioProject..CovidDeaths 
WHERE continent is NULL
GROUP BY location, population
ORDER BY TotalDeathCount DESC

-- GLOBAL NUMBERS 

-- Showing worldwide total cases vs total deaths

SELECT SUM(new_cases) as Total_Cases, SUM(CAST(new_deaths as int)) as Total_Deaths, SUM(CAST(new_deaths as int))/SUM(new_cases) * 100 as DeathPercentage
FROM PortfolioProject..CovidDeaths
WHERE continent is NOT NULL

-- Looking at Total Population vs Vaccinations
-- Window Functions

SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
SUM(CONVERT(int, vac.new_vaccinations)) OVER (Partition BY dea.location ORDER BY dea.location, dea.date) as RollingTotalVaccinated,
SUM(CONVERT(int, vac.new_vaccinations)) OVER (Partition BY dea.location ORDER BY dea.location, dea.date)/dea.population * 100 as PercentPopVaccinated
FROM PortfolioProject..CovidDeaths dea
JOIN PortfolioProject..CovidVaccinations vac
	ON dea.location = vac.location 
	AND dea.date = vac.date
WHERE dea.continent is NOT NULL
GROUP BY dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
ORDER BY dea.location, dea.date

-- USE CTE instead of only window functions

With PopvsVac (Continent, Location, Date, Population, New_Vacciantions, RollingTotalVaccinated)
as
(
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
SUM(CONVERT(int, vac.new_vaccinations)) OVER (Partition BY dea.location ORDER BY dea.location, dea.date) as RollingTotalVaccinated
FROM PortfolioProject..CovidDeaths dea
JOIN PortfolioProject..CovidVaccinations vac
	ON dea.location = vac.location 
	AND dea.date = vac.date
WHERE dea.continent is NOT NULL
GROUP BY dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
)
SELECT *, (RollingTotalVaccinated/Population) * 100 as PercentPopVaccinated 
FROM PopvsVac

-- Use TEMP TABLE instead of CTE

DROP table if exists #PercentPopulationVaccinated
Create table #PercentPopulationVaccinatinated
(
Continent nvarchar(255),
Location nvarchar(255),
Date datetime,
Population numeric,
New_vaccinations numeric,
RollingTotalVaccinated numeric
)

INSERT INTO #PercentPopulationVaccinatinated
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
SUM(CONVERT(int, vac.new_vaccinations)) OVER (Partition BY dea.location ORDER BY dea.location, dea.date) as RollingTotalVaccinated
FROM PortfolioProject..CovidDeaths dea
JOIN PortfolioProject..CovidVaccinations vac
	ON dea.location = vac.location 
	AND dea.date = vac.date
WHERE dea.continent is NOT NULL
GROUP BY dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations

SELECT *, (RollingTotalVaccinated/Population) * 100 as PercentPopVaccinated 
FROM #PercentPopulationVaccinatinated


-- Create a view for later visualizations

Create View PercentPopulationVaccinated as

SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
SUM(CONVERT(int, vac.new_vaccinations)) OVER (Partition BY dea.location ORDER BY dea.location, dea.date) as RollingTotalVaccinated
FROM PortfolioProject..CovidDeaths dea
JOIN PortfolioProject..CovidVaccinations vac
	ON dea.location = vac.location 
	AND dea.date = vac.date
WHERE dea.continent is NOT NULL
GROUP BY dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations


```

## Conclusion

Throughout this project, I have gained a better understanding of how to sift through a large dataset and find insights that will help create better business decisions. This dataset was absolutely massive, and I plan to do more exploration on it in the future. I utilized multiple different methods to query the data while finding some really interesting insights about how countries were affected by COVID and the measures they took against it. Overall, it was a lot of fun to look through the past and find some new things about COVID around the world.
