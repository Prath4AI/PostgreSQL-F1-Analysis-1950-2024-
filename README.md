# PostgreSQL-F1-Analysis-1950-2024-
This F1 SQL analytics project explores race data to uncover driver and constructor performance trends, pit stop efficiency, qualifying vs. race results, and historical dominance using advanced SQL queries across seasons.

✅ 5 Basic SQL Questions

**1.	List all drivers who have raced in a specific year.**

SELECT distinct races.year, CONCAT(drivers.forename, ' ', drivers.surname) AS full_name FROM drivers 
join results
on results.driverid = drivers.driverid
join races
on races.raceid = results.raceid
where year = 2024

**2.	Find the number of races held each year.**

select count(raceid), year from races
group by year
order by year desc

**3.	Get the top 10 circuits with the most races hosted.**

select circuits.name, count(races.raceid) as total_races from circuits
join races
on races.circuitid=circuits.circuitid
group by circuits.name
order by total_races desc

**4.	List all constructors (teams) that participated in the 2021 season.**

SELECT 
    constructors.name, 
    races.year
FROM results
JOIN races ON races.raceId = results.raceId
JOIN constructors ON constructors.constructorId = results.constructorId
WHERE races.year = 2021
group by constructors.name, races.year

**5.	Show total number of wins per driver.**

select concat(drivers.forename,'',drivers.surname) as "Full Name", 
count(case when results.position in (1,2,3) then 1 end) as "Total wins" from results
join drivers on results.driverid = drivers.driverid
group by "Full Name"
order by "Total wins" desc
________________________________________
** 10 Mid-Level SQL Questions**

**1.	Which driver has the most podium finishes (top 3)?**

SELECT 
    CONCAT(drivers.forename, ' ', drivers.surname) AS full_name,
    COUNT(CASE WHEN results.position IN (1, 2, 3) THEN 1 END) AS podium_finishes
FROM drivers
JOIN results ON results.driverid = drivers.driverid
GROUP BY full_name
ORDER BY podium_finishes DESC
limit 3;

**2.	Show the average pit stop time per team for a given race.**

SELECT 
    races.year,
    races.name AS race_name, CONCAT(drivers.forename, ' ', drivers.surname) AS full_name,
    constructors.name AS team_name,
    pit_stops.duration
FROM pit_stops
JOIN races ON pit_stops.raceId = races.raceId
JOIN drivers ON pit_stops.driverId = drivers.driverId
JOIN results ON results.raceId = races.raceId AND results.driverId = drivers.driverId
JOIN constructors ON results.constructorId = constructors.constructorId
where races.year = 2023
GROUP BY full_name, races.year, races.name, constructors.name, pit_stops.duration
ORDER BY pit_stops.duration ASC;

**3.	Rank circuits by average lap time (use lap_times.csv).**
SELECT 
    CONCAT(d.forename, ' ', d.surname) AS "Driver Name",
    MAX(CASE WHEN r.year = 2022 THEN cs.position END) AS "2022 Rank",
    MAX(CASE WHEN r.year = 2022 THEN cs.points END) AS "2022 Points",
    MAX(CASE WHEN r.year = 2023 THEN cs.position END) AS "2023 Rank",
    MAX(CASE WHEN r.year = 2023 THEN cs.points END) AS "2023 Points"
FROM drivers d
JOIN 
    driver_standings cs ON d.driverId = cs.driverId
JOIN 
    races r ON cs.raceId = r.raceId
WHERE 
    r.year IN (2022, 2023)
    AND r.round = (SELECT MAX(round) FROM races r2 WHERE r2.year = r.year)
GROUP BY 
    d.driverId, "Driver Name"
ORDER BY 
    "2023 Rank" ASC;

**4.	Compare driver standings between two seasons side by side.**

SELECT 
    CONCAT(d.forename, ' ', d.surname) AS "Driver Name",
    MAX(CASE WHEN r.year = 2022 THEN cs.position END) AS "2022 Rank",
    MAX(CASE WHEN r.year = 2022 THEN cs.points END) AS "2022 Points",
    MAX(CASE WHEN r.year = 2023 THEN cs.position END) AS "2023 Rank",
    MAX(CASE WHEN r.year = 2023 THEN cs.points END) AS "2023 Points"
FROM 
    drivers d
JOIN 
    driver_standings cs ON d.driverId = cs.driverId
JOIN 
    races r ON cs.raceId = r.raceId
WHERE 
    r.year IN (2022, 2023)
    AND r.round = (SELECT MAX(round) FROM races r2 WHERE r2.year = r.year)
GROUP BY 
    d.driverId, "Driver Name"
ORDER BY 
    "2023 Rank" ASC;

**5.	Which driver improved the most positions from qualifying to final result?**

SELECT 
    CONCAT(drivers.forename, ' ', drivers.surname) AS "Driver Name", 
    round(AVG(qualifying.position),0) AS "Avg Position", 
    round(AVG(results.rank),0) AS "Avg Rank",
    ROUND(
  ((ROUND(AVG(qualifying.position), 0) - ROUND(AVG(results.rank), 0)) / 
   NULLIF(ROUND(AVG(qualifying.position), 0), 0)) * 100, 0
) AS "Improvement %",
    races.year
FROM drivers
JOIN qualifying ON qualifying.driverId = drivers.driverId
JOIN results ON results.driverId = drivers.driverId
JOIN races ON results.raceId = races.raceId
WHERE races.year = 2024
GROUP BY drivers.driverId, drivers.forename, drivers.surname, races.year
ORDER BY "Improvement %" DESC;

**6.	Find the youngest and oldest drivers on the grid in each season.**

SELECT 
    CONCAT(drivers.forename, ' ', drivers.surname) AS "Driver Name", 
    drivers.dob,
    DATE_PART('year', AGE(CURRENT_DATE, drivers.dob)) AS age,
    races.year
FROM 
    drivers
JOIN 
    results ON results.driverId = drivers.driverId
JOIN 
    races ON races.raceId = results.raceId
WHERE 
    races.year = 2007
GROUP BY 
    drivers.driverId, drivers.forename, drivers.surname, drivers.dob, races.year
ORDER BY 
    age;

**7.	Show races where the pole-sitter (qualifying position = 1) did not win.**

SELECT 
    CONCAT(drivers.forename, ' ', drivers.surname) AS driver_name, 
    races.name AS race_name,
    qualifying.position AS qualifying_position, 
    results.rank AS finishing_rank, races.year
FROM drivers
JOIN qualifying ON qualifying.driverId = drivers.driverId
JOIN races ON races.raceId = qualifying.raceId
JOIN results ON results.driverId = drivers.driverId AND results.raceId = qualifying.raceId
WHERE 
    qualifying.position = 1 
    AND results.rank NOT IN (1, 2, 3) and year = 2024
ORDER BY 
    finishing_rank;

**8.	Find the number of times a team finished with both cars in the top 5.**

SELECT 
    constructors.name AS team_name,
    races.year,
    races.name AS race_name,
    COUNT(rank) AS drivers_in_top5
FROM results
JOIN constructors ON results.constructorId = constructors.constructorId
JOIN races ON results.raceId = races.raceId
WHERE results.position <= 5
GROUP BY constructors.name, races.year, races.name, results.raceId
HAVING COUNT(*) >= 2 and year = 2023
ORDER BY team_name, races.year;
_______________________________________
**Advanced-Level SQL Questions**

**1.	Build a season-long points leaderboard using driver_standings.csv.**

SELECT
CONCAT(drivers.forename, ' ', drivers.surname) AS driver_name,
drivers.nationality,
constructors.name, sum(results.points) as total_points, races.year from drivers
join results on results.driverid = drivers.driverid
join constructors on constructors.constructorid = results.constructorid
join races on races.raceid = results.raceid
where races.year = 2024
group by driver_name, drivers.nationality, constructors.name, races.year
order by total_points desc

**2.	Calculate average position gain/loss from qualifying to final result per driver.**

SELECT 
    CONCAT(drivers.forename, ' ', drivers.surname) AS driver_name,
    ROUND(AVG(qualifying.position - results.rank), 2) AS avg_position_change
FROM drivers
JOIN qualifying ON qualifying.driverId = drivers.driverId
JOIN results ON results.driverId = drivers.driverId 
            AND results.raceId = qualifying.raceId
GROUP BY drivers.driverId, drivers.forename, drivers.surname
ORDER BY avg_position_change;

**3.	Find most consistent drivers (least standard deviation in finishing positions).**

SELECT 
    CONCAT(drivers.forename, ' ', drivers.surname) AS driver_name,
    ROUND(STDDEV_POP(results.rank), 2) AS rank_stddev,
    COUNT(*) AS races_count
FROM drivers
JOIN results ON results.driverId = drivers.driverId
JOIN races ON results.raceId = races.raceId
WHERE races.year = 2023
GROUP BY drivers.driverId, drivers.forename, drivers.surname
HAVING COUNT(*) > 3
ORDER BY rank_stddev ASC;

**4.	Analyze DNF (Did Not Finish) patterns by driver or constructor.**

SELECT 
    CONCAT(drivers.forename, ' ', drivers.surname) AS driver_name,
	races.year, status.status
FROM results
JOIN drivers ON results.driverId = drivers.driverId
join races on races.raceid = results.raceid
join status on status.status_id = results.statusid
WHERE results.statusId IN (
    SELECT status_id FROM status WHERE status NOT ILIKE '%finished%'
) and year = 2024
GROUP BY driver_name, races.year, status.status


**5.	Calculate pit stop efficiency per constructor over a season.**

SELECT 
    constructors.name AS constructor_name,
    ROUND(AVG(pit_stops.duration), 2) AS avg_pit_stop_time,
    COUNT(*) AS total_pit_stops,
    races.year
FROM pit_stops
JOIN races ON pit_stops.raceId = races.raceId
JOIN results ON results.raceId = races.raceId AND results.driverId = pit_stops.driverId
JOIN constructors ON results.constructorId = constructors.constructorId
WHERE races.year = 2023
GROUP BY constructors.name, races.year
ORDER BY avg_pit_stop_time ASC;

**6.	Track how the constructor championship battle evolved race-by-race.**

SELECT 
    races.round,
    races.name AS race_name,
    races.date,
    constructors.name AS constructor_name,
    SUM(constructor_standings.points) AS total_points_after_race
FROM constructor_standings
JOIN constructors ON constructor_standings.constructorId = constructors.constructorId
JOIN races ON constructor_standings.raceId = races.raceId
WHERE races.year = 2023
GROUP BY races.round, races.name, races.date, constructors.name
ORDER BY races.round, total_points_after_race DESC;

**7.	Detect outlier races with unusually high pit stops or DNFs.**

WITH longest_pitstop_per_race AS (
    SELECT 
        raceid,
        MAX(duration) AS longest_pit_stop
    FROM pit_stops
    GROUP BY raceid
)

SELECT 
    races.name AS race_name,
    races.year,
    pit.longest_pit_stop,
    COUNT(CASE WHEN status.status != 'Finished' THEN 1 END) AS total_dnf
FROM races
JOIN results ON results.raceid = races.raceid
JOIN status ON status.status_id = results.statusid
LEFT JOIN longest_pitstop_per_race pit ON pit.raceid = races.raceid
WHERE races.year = 2023
GROUP BY races.name, races.year, pit.longest_pit_stop
ORDER BY total_dnf DESC;



**8.	Create a win ratio metric for drivers with over X race entries.**

SELECT 
  CONCAT(drivers.forename, ' ', drivers.surname) AS driver_name,
  COUNT(CASE WHEN results.rank IN (1, 2, 3) THEN 1 END) * 1.0 / COUNT(DISTINCT races.raceId) AS win_ratio
FROM results
JOIN drivers ON drivers.driverId = results.driverId
JOIN races ON races.raceId = results.raceId
GROUP BY driver_name
ORDER BY win_ratio DESC;

**9.	Correlate qualifying performance with final race result across seasons.**

SELECT 
    races.year,
    results.grid,
    results.positionOrder
FROM results
JOIN races ON races.raceId = results.raceId
WHERE results.grid > 0
  AND results.positionOrder > 0;

**10.	Analyze driver head-to-head performance vs. their teammate.**

WITH driver_points AS (
    SELECT 
        constructors.name AS constructor_name,
        CONCAT(drivers.forename, ' ', drivers.surname) AS driver_name,
        SUM(results.points) AS total_points,
        RANK() OVER (PARTITION BY constructors.name ORDER BY SUM(results.points) DESC) AS rank
    FROM results
    JOIN drivers ON drivers.driverId = results.driverId
    JOIN constructors ON constructors.constructorId = results.constructorId
    JOIN races ON races.raceId = results.raceId
    WHERE races.year = 2022
    GROUP BY constructors.name, drivers.driverId),

top_2 AS (SELECT * FROM driver_points WHERE rank <= 2)

SELECT 
    c.constructor_name,
    MAX(CASE WHEN c.rank = 1 THEN c.driver_name END) AS driver_1,
    MAX(CASE WHEN c.rank = 1 THEN c.total_points END) AS points_1,
    MAX(CASE WHEN c.rank = 2 THEN c.driver_name END) AS driver_2,
    MAX(CASE WHEN c.rank = 2 THEN c.total_points END) AS points_2
FROM top_2 c
GROUP BY c.constructor_name
ORDER BY constructor_name;

**11.	Find the driver with the highest number of races without a podium.**

WITH driver_points AS (
    SELECT 
        constructors.name AS constructor_name,
        CONCAT(drivers.forename, ' ', drivers.surname) AS driver_name,
        SUM(results.points) AS total_points,
        RANK() OVER (PARTITION BY constructors.name ORDER BY SUM(results.points) DESC) AS ranks
    FROM results
    JOIN drivers ON drivers.driverId = results.driverId
    JOIN constructors ON constructors.constructorId = results.constructorId
    JOIN races ON races.raceId = results.raceId
    WHERE races.year = 2022
    GROUP BY constructors.name, drivers.driverId),

top_2 AS (SELECT * FROM driver_points WHERE ranks <= 2)

SELECT 
    c.constructor_name,
    MAX(CASE WHEN c.ranks = 1 THEN c.driver_name END) AS driver_1,
    MAX(CASE WHEN c.ranks = 1 THEN c.total_points END) AS points_1,
    MAX(CASE WHEN c.ranks = 2 THEN c.driver_name END) AS driver_2,
    MAX(CASE WHEN c.ranks = 2 THEN c.total_points END) AS points_2
FROM top_2 c
GROUP BY c.constructor_name
ORDER BY constructor_name;

**12.	Identify the most dominant races (driver led from pole to win).**

SELECT 
    CONCAT(drivers.forename, ' ', drivers.surname) AS driver_name,
    races.year,
    races.name AS race_name,
    circuits.name AS circuit_name
FROM results
JOIN drivers ON drivers.driverId = results.driverId
JOIN races ON races.raceId = results.raceId
JOIN circuits ON circuits.circuitId = races.circuitId
WHERE results.grid = 1  
  AND results.positionOrder = 1 and races.year = 2024 
ORDER BY races.year;
13.	Rank drivers by performance trend across 5-year periods (rolling average).
WITH driver_yearly_points AS (
    SELECT 
        drivers.driverId,
        CONCAT(drivers.forename, ' ', drivers.surname) AS driver_name,
        races.year,
        SUM(results.points) AS total_points
    FROM results
    JOIN drivers ON drivers.driverId = results.driverId
    JOIN races ON races.raceId = results.raceId
    GROUP BY drivers.driverId, races.year
),
rolling_avg_points AS (
    SELECT 
        driver_name,
        year,
        AVG(total_points) OVER (
            PARTITION BY driver_name
            ORDER BY year
            ROWS BETWEEN 4 PRECEDING AND CURRENT ROW
        ) AS rolling_avg_5yr
    FROM driver_yearly_points
)
SELECT *
FROM rolling_avg_points
ORDER BY year, rolling_avg_5yr DESC;
