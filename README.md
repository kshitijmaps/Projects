# Covid_Asia 
-- To showcase the knowledge and help the budding SQL developers with a project of recent data

-- This project analyse the Covid cases, deaths, vaccinations, etc. collected from Our World Data for Covid fro the Asian market. As it is the largest market for most of the upcoming industires.

-- These are writen on SQLite Studios


-- Checking the data we have imported for the porject

Select *
From CovidDeaths
Where Continent != '' 
order by 3,4;

Select *
From CovidVaccinations
Where Continent != '' 
order by 3,4;


-- Selecting the data for starting analysis

Select Location, date, total_cases, new_cases, total_deaths, population
From CovidDeaths
Where Continent != '' 
order by 1,2;


--Checking the probability of death to covid in covid positive people
--T0tal cases vs Toal Death (One can comment out asia condition and use Continent != '' to study the world data)

Select Location, date, total_deaths, total_cases, Round((cast(total_deaths as real)/total_cases)*100,4)
From CovidDeaths
Where Continent = 'Asia'
-- Where Continent != ''
order by 1,2;


-- Checking the probability of contracting Covid
-- or percent population infected (One can comment out asia condition and use Continent != '' to study the world data) 

Select Location, date, Population, total_cases, Round((cast(total_cases as real)/population)*100,4)
From CovidDeaths
Where continent ='Asia'
-- Where Continent != ''
order by 1,2;


-- Checking the countries with higest death count compared to population
-- (One can comment out asia condition and use Continent != '' to study the world data) 

Select Location, MAX(Total_deaths) as TotalDeathCount
From CovidDeaths
Where continent ='Asia'
-- Where Continent != '' 
Group by Location
order by TotalDeathCount desc;


-- Checkeing the countires with higest infection per population
-- (One can comment out asia condition and use Continent != '' to study the world data)

Select Location, Population, MAX(total_cases) as HighestInfectionCount,  Round((cast(total_cases as real)/population),4)*100 as PercentPopulationInfected
From CovidDeaths
Where continent ='Asia'
-- Where Continent != '' 
Group by Location, Population
order by PercentPopulationInfected desc;


-- BREAKING THINGS DOWN BY CONTINENT

-- Showing contintents with the highest death count per population

Select continent, MAX(cast(Total_deaths as int)) as TotalDeathCount
From CovidDeaths
Where continent != '' 
Group by continent
order by TotalDeathCount desc;


-- Total Global numbers of Covid cases and deaths

Select SUM(new_cases) as total_cases, SUM(new_deaths) as total_deaths, SUM(cast(new_deaths as int))/SUM(New_Cases)*100 as DeathPercentage
From CovidDeaths
where continent != ''
order by 1,2;


-- checking the perccentage population that have receied first does of vacinnation
-- Population VS First does of Vaccination

Select dth.continent, dth.location, dth.date, dth.population, vcn.new_vaccinations
, SUM(vcn.new_vaccinations) OVER (Partition by dth.Location Order by dth.location, dth.Date) as RollingPeopleVaccinated
From CovidDeaths dth
Join CovidVaccinations vcn
	On dth.location = vcn.location
	and dth.date = vcn.date
where dth.continent = 'Asia'
-- where dth.continent != ''
order by 2,3;


-- Now we will use CTE (Common Table expression) to perform calculations on Partitoin By over our last query

With PopvsVac (Continent, Location, Date, Population, New_Vaccinations, RollingPeopleVaccinated)
as
(
Select dth.continent, dth.location, dth.date, dth.population, vcn.new_vaccinations
, SUM(vcn.new_vaccinations) OVER (Partition by dth.Location Order by dth.location, dth.Date) as RollingPeopleVaccinated
From CovidDeaths dth
Join CovidVaccinations vcn
	On dth.location = vcn.location
	and dth.date = vcn.date
where dth.continent = 'Asia'
-- where dth.continent != ''
order by 2,3
)
Select *, Round((RollingPeopleVaccinated/Population)*100,4)
From PopvsVac;


-- Using Temp Table to perform calculation like last query

DROP Table if exists PercentPopulationVaccinated;

Create Table PercentPopulationVaccinated
(
Continent nvarchar(255),
Location nvarchar(255),
Date date,
Population numeric,
New_vaccinations numeric,
RollingPeopleVaccinated numeric
);

Insert Into PercentPopulationvaccinated
Select dth.continent, dth.location, dth.date, dth.population, vcn.new_vaccinations
, SUM(vcn.new_vaccinations) OVER (Partition by dth.Location Order by dth.location, dth.Date) as RollingPeopleVaccinated

From CovidDeaths dth
Join CovidVaccinations van
	On dth.location = vcn.location
	and dth.date = vcn.date;

Select *, Round((Cast(RollingPeopleVaccinated as Real)/Population)*100,4)
From PercentPopulationVaccinated
where Continent = 'Asia';
-- where Continent != '';


-- For later visulaization of our data, we will create a view

Create View PercentPopulationVaccinated as
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
From CovidDeaths dea
Join CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent = 'Asia';
-- Where dea.continent != '';
