# Project 1: COVID 19 Data Exploratory Analysis
![image](https://user-images.githubusercontent.com/106350577/170791992-92d83b1b-58b5-4a3c-8ed2-c9f6ac420aae.png)
The datasets that I will be using in my analysis come from our worldindata.org that tracks information about COVID 19 such as number of cases, deaths, vaccination count, etc...

[Datasets](https://ourworldindata.org/covid-deaths)

Firstly, I will use SQL in order to manipulate the dataset in order to answer multiple questions. 

Then, I will use Tableau to create a dashboard that is filled with visualizations of this dataset.

```
* Creating table 1 named CovidDeaths

CREATE TABLE coviddeaths (
  iso_code varchar(50),
  continent varchar(30),
  location varchar(50),
  date date,
  population bigint,
  total_cases bigint,
  new_cases bigint,
  new_cases_smoothed numeric(,3)
  total_deaths bigint,
  new_deaths bigint,
  new_deaths_smoothed numeric(,3),
  total_cases_per_million numeric(,3),
  new_cases_per_million numeric(,3),
  new_cases_smoothed_per_million numeric(,3),
  total_deaths_per_million numeric(,3),
  new_deaths_per_million numeric(,3),
  new_deaths_smoothed_per_million numeric(,3),
  reproduction_rate numeric(,3),
  icu_patients bigint,
  icu_patients_per_million numeric(,3),
  hosp_patients bigint,
  hosp_patients_per_million numeric(,3),
  weekly_icu_admissions numeric(,3),
  weekly_icu_admissions_per_million numeric(,3),
  weekly_hosp_admissions numeric(,3),
  weekly_hosp_admissions_per_million numeric(,3)
);

* Creating table 2 named covidvaccinations

CREATE TABLE CovidVaccinations (
  iso_code varchar(50),
  continent varchar(50),
  location varchar(50),
  date date,
  new_tests bigint,
  total_tests bigint,
  total_tests_per_thousand numeric(15,3),
  new_tests_per_thousand numeric(15,3),
  new_tests_smoothed bigint,
  new_tests_smoothed_per_thousand numeric(15,3),
  positive_rate numeric(15,3),
  tests_per_case numeric(15,3),
  test_units varchar(25),
  total_vaccinations bigint,
  people_vaccinated bigint,
  people_fully_vaccinated bigint,
  new_vaccinations bigint,
  new_vaccinations_smoothed bigint,
  total_vaccinations_per_hundred numeric(15,3),
  people_vaccinated_per_hundred numeric(15,3),
  people_fully_vaccinated_per_hundred numeric(15,3),
  new_vaccinations_smoothed_per_million bigint,
  stringency_index numeric(15,3),
  population_density numeric(15,3),
  median_age numeric(15,3),
  aged_65_older numeric(15,3),
  aged_70_older numeric(15,3),
  gdp_per_capita numeric(15,3),
  extreme_poverty numeric(15,3),
  cardiovasc_deathrate numeric(15,3),
  diabetes_prevalence numeric(15,3),
  female_smokers numeric(15,3),
  male_smokers numeric(15,3),
  handwashing_facilities numeric(15,3),
  hospital_beds_per_thousand numeric(15,3),
  life_expectancy numeric(15,3),
  human_development_index numeric(15,3)
);

* Making sure the datasets were successfully imported
SELECT * 
FROM coviddeaths;

SELECT * 
FROM covidvaccinations;

```

```
* Which countries have the highest/lowest deathrate of covid 19?

*Highest Deathrate
SELECT location,
max(total_cases) AS "Total Cases",
max(total_deaths) AS "Total Deaths",
(max(total_deaths)::numeric(15,2)/max(total_cases)*100)::decimal(5,2) AS "Death Rate"
FROM coviddeaths
GROUP BY location ORDER BY "Death Rate" DESC;

* Lowest Deathrate
SELECT location,
max(total_cases) AS "Total Cases",
max(total_deaths) AS "Total Deaths",
(max(total_deaths)::numeric(15,2)/max(total_cases)*100)::decimal(5,2) AS "Death Rate"
FROM coviddeaths
GROUP BY location ORDER BY "Death Rate" ASC;

* Which countries have the highest/lowest infection rate of covid 19?
*Highest Infection Rate

SELECT location,
population,
MAX(total_cases) AS "Total Cases",
(MAX(total_cases)::numeric(15,2)/population*100)::decimal(5,3) AS "Infection Rate"
FROM coviddeaths
GROUP BY location, population ORDER BY "Infection Rate" DESC;

*Lowest Infection Rate

SELECT location,
population,
MAX(total_cases) AS "Total Cases",
(MAX(total_cases)::numeric(15,2)/population*100)::decimal(5,3) AS "Infection Rate"
FROM coviddeaths
GROUP BY location, population ORDER BY "Infection Rate" ASC;


Which countries have the highest deathcount?
SELECT distinct(location), 
max(total_deaths) AS "Total Deaths"
FROM coviddeaths
WHERE continent is NOT NULL AND total_deaths is NOT NULL
GROUP BY location ORDER BY "Total Deaths" DESC

Which continents have the highest death count?
SELECT location, 
max(total_deaths) AS "Total Deaths"
FROM coviddeaths
WHERE continent is NULL AND total_deaths is NOT NULL
GROUP BY location ORDER BY "Total Deaths", location DESC limit 8

What is the global death rate?
SELECT location, 
max(total_deaths) AS "Total Deaths"
FROM coviddeaths
WHERE continent is NULL AND total_deaths is NOT NULL
GROUP BY location ORDER BY "Total Deaths" DESC limit 1

* Joining our two tables
This query shows how vaccinations have been growing by dates.

SELECT cdeath.continent,
cdeath.location,
cdeath.date,
cdeath.population,
vacc.new_vaccinations,
SUM(new_vaccinations) OVER (PARTITION BY cdeath.location ORDER BY cdeath.location, cdeath.date) AS "Current Vaccinations"
FROM coviddeaths as cdeath
INNER JOIN covidvaccinations as vacc
ON cdeath.location = vacc.location 
AND cdeath.continent = vacc.continent
AND cdeath.date = vacc.date
WHERE new_vaccinations IS NOT NULL
ORDER BY cdeath.location, cdeath.date

Using a CTE, we can calculate how the percentage of vaccinations has changed. 
WITH VacInfo (Continent, Location, Date, Population,New_vaccinations, "Current Vaccinations") AS 
(
SELECT cdeath.continent,
cdeath.location,
cdeath.date,
cdeath.population,
vacc.new_vaccinations,
SUM(new_vaccinations) OVER (PARTITION BY cdeath.location ORDER BY cdeath.location, cdeath.date) AS "Current Vaccinations"
FROM coviddeaths as cdeath
INNER JOIN covidvaccinations as vacc
ON cdeath.location = vacc.location 
AND cdeath.continent = vacc.continent
AND cdeath.date = vacc.date
WHERE new_vaccinations IS NOT NULL
ORDER BY cdeath.location, cdeath.date
)
SELECT *, (("Current Vaccinations")/Population*100)::numeric(5,2)
FROM VacInfo
```
# Project 2:
