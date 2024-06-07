### Covid-19 Data Exploration

#### Overview
This project involves exploring and analyzing Covid-19 data using various SQL skills. The goal is to derive insights about the impact of Covid-19 across different countries and continents. Below is an explanation of each step, rephrased for clarity.

#### Skills Demonstrated
- Joins
- Common Table Expressions (CTEs)
- Temporary Tables
- Window Functions
- Aggregate Functions
- Creating Views
- Converting Data Types

#### Steps and Explanation

1. **Initial Data Exploration**
   - **Code:**
     ```sql
     SELECT *
     FROM PortfolioProject..CovidDeaths
     WHERE continent IS NOT NULL 
     ORDER BY location, date;
     ```
   - **Explanation:** Retrieves all records from the CovidDeaths table where the continent is specified, ordering them by location and date to get a structured view of the data.

2. **Selecting Initial Data**
   - **Code:**
     ```sql
     SELECT Location, date, total_cases, new_cases, total_deaths, population
     FROM PortfolioProject..CovidDeaths
     WHERE continent IS NOT NULL 
     ORDER BY location, date;
     ```
   - **Explanation:** Selects specific columns to focus on, such as location, date, total cases, new cases, total deaths, and population, ordered by location and date for easy analysis.

3. **Total Cases vs Total Deaths**
   - **Code:**
     ```sql
     SELECT Location, date, total_cases, total_deaths, 
            (total_deaths/total_cases)*100 AS DeathPercentage
     FROM PortfolioProject..CovidDeaths
     WHERE location LIKE '%states%'
       AND continent IS NOT NULL 
     ORDER BY location, date;
     ```
   - **Explanation:** Calculates the death percentage for countries with 'states' in their names, showing the likelihood of dying if infected by Covid-19 in those countries.

4. **Total Cases vs Population**
   - **Code:**
     ```sql
     SELECT Location, date, Population, total_cases,  
            (total_cases/population)*100 AS PercentPopulationInfected
     FROM PortfolioProject..CovidDeaths
     ORDER BY location, date;
     ```
   - **Explanation:** Shows the percentage of the population infected with Covid-19 for each location, providing insight into the spread of the virus.

5. **Countries with the Highest Infection Rate**
   - **Code:**
     ```sql
     SELECT Location, Population, 
            MAX(total_cases) AS HighestInfectionCount,  
            MAX((total_cases/population))*100 AS PercentPopulationInfected
     FROM PortfolioProject..CovidDeaths
     GROUP BY Location, Population
     ORDER BY PercentPopulationInfected DESC;
     ```
   - **Explanation:** Identifies countries with the highest infection rate relative to their population by calculating the maximum infection count and infection percentage.

6. **Countries with the Highest Death Count**
   - **Code:**
     ```sql
     SELECT Location, 
            MAX(CAST(Total_deaths AS INT)) AS TotalDeathCount
     FROM PortfolioProject..CovidDeaths
     WHERE continent IS NOT NULL 
     GROUP BY Location
     ORDER BY TotalDeathCount DESC;
     ```
   - **Explanation:** Lists countries with the highest number of deaths, providing a perspective on the mortality impact of Covid-19.

7. **Continents with the Highest Death Count**
   - **Code:**
     ```sql
     SELECT continent, 
            MAX(CAST(Total_deaths AS INT)) AS TotalDeathCount
     FROM PortfolioProject..CovidDeaths
     WHERE continent IS NOT NULL 
     GROUP BY continent
     ORDER BY TotalDeathCount DESC;
     ```
   - **Explanation:** Shows which continents have the highest death counts, helping to understand the global impact.

8. **Global Numbers**
   - **Code:**
     ```sql
     SELECT SUM(new_cases) AS total_cases, 
            SUM(CAST(new_deaths AS INT)) AS total_deaths, 
            (SUM(CAST(new_deaths AS INT))/SUM(new_cases))*100 AS DeathPercentage
     FROM PortfolioProject..CovidDeaths
     WHERE continent IS NOT NULL 
     ORDER BY total_cases, total_deaths;
     ```
   - **Explanation:** Provides a global summary of total cases, total deaths, and death percentage, giving an overall view of the pandemic's impact.

9. **Total Population vs Vaccinations**
   - **Code:**
     ```sql
     SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
            SUM(CONVERT(INT, vac.new_vaccinations)) OVER (PARTITION BY dea.Location ORDER BY dea.location, dea.Date) AS RollingPeopleVaccinated
     FROM PortfolioProject..CovidDeaths dea
     JOIN PortfolioProject..CovidVaccinations vac
         ON dea.location = vac.location
        AND dea.date = vac.date
     WHERE dea.continent IS NOT NULL 
     ORDER BY dea.location, dea.date;
     ```
   - **Explanation:** Displays the cumulative number of people vaccinated in each location, helping to understand vaccination progress.

10. **Using CTE to Calculate Vaccination Percentage**
    - **Code:**
      ```sql
      WITH PopvsVac (Continent, Location, Date, Population, New_Vaccinations, RollingPeopleVaccinated) AS
      (
          SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
                 SUM(CONVERT(INT, vac.new_vaccinations)) OVER (PARTITION BY dea.Location ORDER BY dea.location, dea.Date) AS RollingPeopleVaccinated
          FROM PortfolioProject..CovidDeaths dea
          JOIN PortfolioProject..CovidVaccinations vac
              ON dea.location = vac.location
             AND dea.date = vac.date
          WHERE dea.continent IS NOT NULL 
      )
      SELECT *, (RollingPeopleVaccinated/Population)*100 AS PercentPopulationVaccinated
      FROM PopvsVac;
      ```
    - **Explanation:** Uses a Common Table Expression (CTE) to calculate and display the percentage of the population vaccinated in each location.

11. **Using Temp Table to Calculate Vaccination Percentage**
    - **Code:**
      ```sql
      DROP TABLE IF EXISTS #PercentPopulationVaccinated;
      CREATE TABLE #PercentPopulationVaccinated
      (
          Continent NVARCHAR(255),
          Location NVARCHAR(255),
          Date DATETIME,
          Population NUMERIC,
          New_vaccinations NUMERIC,
          RollingPeopleVaccinated NUMERIC
      );

      INSERT INTO #PercentPopulationVaccinated
      SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
             SUM(CONVERT(INT, vac.new_vaccinations)) OVER (PARTITION BY dea.Location ORDER BY dea.location, dea.Date) AS RollingPeopleVaccinated
      FROM PortfolioProject..CovidDeaths dea
      JOIN PortfolioProject..CovidVaccinations vac
          ON dea.location = vac.location
         AND dea.date = vac.date;

      SELECT *, (RollingPeopleVaccinated/Population)*100 AS PercentPopulationVaccinated
      FROM #PercentPopulationVaccinated;
      ```
    - **Explanation:** Uses a temporary table to store and calculate the vaccination percentage, allowing for further analysis and manipulation.

12. **Creating a View for Later Use**
    - **Code:**
      ```sql
      CREATE VIEW PercentPopulationVaccinated AS
      SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
             SUM(CONVERT(INT, vac.new_vaccinations)) OVER (PARTITION BY dea.Location ORDER BY dea.location, dea.Date) AS RollingPeopleVaccinated
      FROM PortfolioProject..CovidDeaths dea
      JOIN PortfolioProject..CovidVaccinations vac
          ON dea.location = vac.location
         AND dea.date = vac.date
      WHERE dea.continent IS NOT NULL;
      ```
    - **Explanation:** Creates a view to store the calculated data, which can be used later for visualizations and further analysis.
