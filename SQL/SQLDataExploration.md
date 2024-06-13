# 🦠 Exploration of Global COVID-19 Dataset with SQL
The data on the coronavirus pandemic is updated daily, is open source, and publically available [here](https://ourworldindata.org/covid-deaths).

### Total Cases vs Total Deaths
```sql
SELECT location, date, total_cases, total_deaths, (total_deaths/total_cases) * 100 AS DeathPercentage
FROM CovidDeaths
WHERE location = 'Malaysia'
ORDER BY 1, 2
```
Output:

![image](https://github.com/RyanTAnalytics/ryantang-portfolio/assets/170547496/8f721020-2b26-444e-b71f-411e40700c4f)

### Total Cases vs Population
```sql
SELECT location, date, population, total_cases, (total_cases/population) * 100 AS InfectionPercentage
FROM CovidDeaths
WHERE location = 'Malaysia'
ORDER BY 1, 2
```
Output:

![image](https://github.com/RyanTAnalytics/ryantang-portfolio/assets/170547496/ad97809f-572b-4b9b-8741-b8dda212122b)

### Countries with the Highest Infection Rate vs Populations
```sql
SELECT location, population, MAX(total_cases) AS HighestInfectionCount, MAX((total_cases/population)) * 100 AS InfectionPercentage
FROM CovidDeaths
WHERE population <> 0
GROUP BY location, population
ORDER BY InfectionPercentage DESC
```
Output:

![image](https://github.com/RyanTAnalytics/ryantang-portfolio/assets/170547496/c712d8f7-8910-40ce-8f9c-7218f449d340)

### Countries with the Highest Death Count per Population
```sql
SELECT location, MAX(total_deaths) AS TotalDeathCount
FROM CovidDeaths
GROUP BY location
ORDER BY TotalDeathCount DESC
```
Output:

![image](https://github.com/RyanTAnalytics/ryantang-portfolio/assets/170547496/0a8d9922-63b5-47f9-bf57-c76cac1f0857)

### Countries with the Highest Death Count per Continent
```sql
SELECT continent, MAX(total_deaths) AS TotalDeathCount
FROM CovidDeaths
GROUP BY continent
ORDER BY TotalDeathCount DESC
```
Output:

![image](https://github.com/RyanTAnalytics/ryantang-portfolio/assets/170547496/6cfed593-7fc8-4bc3-a5e7-09aed3267021)

### Global Numbers
```sql
SELECT SUM(new_cases) as total_cases, SUM(new_deaths) as total_deaths, (SUM(new_deaths)/SUM(new_cases))*100 as DeathPercentage
FROM CovidDeaths
```
Output:

![image](https://github.com/RyanTAnalytics/ryantang-portfolio/assets/170547496/114eb4ed-1cba-4854-8cb4-87e6b11cb75c)

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
Output:

![image](https://github.com/RyanTAnalytics/ryantang-portfolio/assets/170547496/fce1deed-00a1-49ea-b902-26ad9cf08c72)

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



