# 🦠 Exploration of Global COVID-19 Dataset with SQL
The data on the coronavirus pandemic is updated daily, is open source, and publically available [here](https://ourworldindata.org/covid-deaths).

### Total Cases vs Total Deaths
```sql
SELECT location, date, total_cases, total_deaths, (total_deaths/total_cases) * 100 AS DeathPercentage
FROM CovidDeaths
WHERE location = 'Malaysia'
ORDER BY 1, 2
```

### Total Cases vs Population
```sql
SELECT location, date, population, total_cases, (total_cases/population) * 100 AS InfectionPercentage
FROM CovidDeaths
WHERE location = 'Malaysia'
ORDER BY 1, 2
```

### Countries with the Highest Infection Rate vs Populations
```sql
SELECT location, population, MAX(total_cases) AS HighestInfectionCount, MAX((total_cases/population)) * 100 AS InfectionPercentage
FROM CovidDeaths
WHERE population <> 0
GROUP BY location, population
ORDER BY InfectionPercentage DESC
```

### Countries with the Highest Death Count per Population
```sql
SELECT location, MAX(total_deaths) AS TotalDeathCount
FROM CovidDeaths
GROUP BY location
ORDER BY TotalDeathCount DESC
```

### Countries with the Highest Death Count per Continent
```sql
SELECT continent, MAX(total_deaths) AS TotalDeathCount
FROM CovidDeaths
GROUP BY continent
ORDER BY TotalDeathCount DESC
```
### Global Numbers
```sql
SELECT SUM(new_cases) as total_cases, SUM(new_deaths) as total_deaths, (SUM(new_deaths)/SUM(new_cases))*100 as DeathPercentage
FROM PortfolioProject..CovidDeaths
```

### Total Population vs Vaccinations
```sql
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, SUM(vac.new_vaccinations) OVER (Partition by dea.location ORDER BY dea.location, dea.date) AS RollingPeopleVaccinated
FROM CovidDeaths dea
JOIN CovidVaccinations vac
	ON dea.location = vac.location AND
	dea.date = vac.date
WHERE dea.continent <> ' '
ORDER BY 2, 3
```

### Creating View to Store Data for Later Visualizations and Analysis
```sql
CREATE VIEW PopulationVaccinated AS
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, SUM(vac.new_vaccinations) OVER (Partition by dea.location ORDER BY dea.location, dea.date) AS RollingPeopleVaccinated
FROM CovidDeaths dea
JOIN CovidVaccinations vac
	ON dea.location = vac.location AND
	dea.date = vac.date
WHERE dea.continent <> ' '
```



