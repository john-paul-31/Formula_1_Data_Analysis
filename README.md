# Formula 1 World Championship Data Analysis Using SQL & Tableau
![Logo](https://github.com/john-paul-31/Formula_1_Data_Analysis/blob/main/Logo.png)

## Database Schema
![App Screenshot](https://github.com/john-paul-31/Formula_1/blob/main/ER%20Diagram.png)
---
## Top 5 Drivers with Most Wins.

![CTE Logo](https://img.shields.io/badge/-CTE-green) ![Window Function logo](https://img.shields.io/badge/-Window%20Function-yellow)

```
WITH winners AS (
    SELECT
        driverId,
        COUNT(`position`) AS Number_of_Wins
    FROM
        results r
    WHERE
        `position` = 1
    GROUP BY
        driverId
)
SELECT
    CONCAT(d.forename, " ", d.surname) AS Driver,
    Number_of_Wins
FROM
    winners w
    JOIN drivers d ON d.driverId = w.driverId
ORDER BY
    Number_of_Wins DESC
LIMIT
    10;
```
---
## Drivers with Lowest Average Number of Overtakes

```
WITH RaceOvertakes AS (
    SELECT
        lt.raceId,
        lt.driverId,
        lt.lap,
        lt.position,
        LAG(lt.position) OVER (PARTITION BY lt.raceId, lt.driverId ORDER BY lt.lap) AS previous_position
    FROM
        lap_times lt
)
SELECT
    ro.driverId,
    d.surname,
    AVG(CASE WHEN ro.position < ro.previous_position THEN 1 ELSE 0 END) AS average_overtakes
FROM
    RaceOvertakes ro
INNER JOIN
    drivers d ON d.driverId = ro.driverId
GROUP BY
    ro.driverId, d.surname
ORDER BY
    average_overtakes ASC;
```
---
## Drivers who Won for Multiple Constructors

```
SELECT
    d.driverId,
    CONCAT(d.forename, ' ', d.surname) AS name,
    GROUP_CONCAT(DISTINCT c.name ORDER BY c.name SEPARATOR ', ') AS winning_constructors
FROM
    drivers d
    JOIN results r ON d.driverId = r.driverId
    JOIN constructors c ON r.constructorId = c.constructorId
WHERE
    r.position = 1
GROUP BY
    d.driverId, d.forename, d.surname
HAVING
    COUNT(DISTINCT r.constructorId) > 1;

```
---
## Constructors with most 1-2 Finishes

```
WITH RaceWinners AS (
    SELECT
        r.raceId,
        r.constructorId AS winnerConstructorId,
        r.position AS winnerPosition
    FROM
        results r
    WHERE
        r.position = 1
),
RaceRunnersUp AS (
    SELECT
        r.raceId,
        r.constructorId AS runnerUpConstructorId,
        r.position AS runnerUpPosition
    FROM
        results r
    WHERE
        r.position = 2
)
SELECT
    c.name AS constructorName,
    COUNT(*) AS num_1_2_finishes
FROM
    RaceWinners w
INNER JOIN
    RaceRunnersUp r
    ON w.raceId = r.raceId
    AND w.winnerConstructorId = r.runnerUpConstructorId
INNER JOIN
    constructors c
    ON w.winnerConstructorId = c.constructorId
GROUP BY
    w.winnerConstructorId, c.name
ORDER BY
    num_1_2_finishes DESC;

```
---
## Drivers with highest average finishing position while starting outside Top 5 Grid Positions

```

WITH RaceData AS (
    SELECT
        q.raceId,
        q.driverId,
        r.position AS finishingPosition,
        q.position AS startingPosition
    FROM
        qualifying q
    JOIN
        results r ON q.raceId = r.raceId AND q.driverId = r.driverId
    WHERE
        q.position > 5  -- Starting position greater than 5
)
SELECT
    d.surname,
    AVG(rd.finishingPosition) AS averageFinishingPosition
FROM
    RaceData rd
JOIN
    drivers d ON rd.driverId = d.driverId
GROUP BY
    rd.driverId, d.forename, d.surname
ORDER BY
    averageFinishingPosition ASC;

```
---
## Drivers with Most Podiums in Rookie Season

```
WITH DriverFirstYear AS (
    SELECT
        r.driverId,
        MIN(rs.year) AS firstYear
    FROM
        results r
    INNER JOIN races rs ON r.raceId = rs.raceId
    WHERE
        r.position <= 3  -- Podium finishes (1st, 2nd, or 3rd)
    GROUP BY
        r.driverId
)
SELECT
    d.driverId,
    d.forename,
    d.surname,
    COALESCE(podiums.podiumFinishes, 0) AS podiumFinishes,
    year(dfy.firstYear) as Rookie_Year
FROM
    drivers d
LEFT JOIN (
    SELECT
        r.driverId,
        COUNT(*) AS podiumFinishes
    FROM
        results r
    INNER JOIN races rs ON r.raceId = rs.raceId
    INNER JOIN DriverFirstYear dfy ON r.driverId = dfy.driverId AND rs.year = dfy.firstYear
    WHERE
        r.position <= 3  -- Podium finishes (1st, 2nd, or 3rd)
    GROUP BY
        r.driverId
) podiums ON d.driverId = podiums.driverId
INNER JOIN DriverFirstYear dfy ON d.driverId = dfy.driverId
ORDER BY
    d.driverId;

```
---
## Constructors who Won Races with 5 or More Different Drivers

```

SELECT
    c.name AS constructor_name,
    GROUP_CONCAT(DISTINCT d.forename ORDER BY d.forename SEPARATOR ', ') AS winning_drivers
FROM
    constructors c
    JOIN results r ON c.constructorId = r.constructorId
    JOIN drivers d ON r.driverId = d.driverId
WHERE
    r.position = 1
GROUP BY
    c.name
HAVING
    COUNT(DISTINCT r.driverId) >= 5;

```


