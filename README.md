# Formula 1 World Championship Data Analysis Using SQL & Tableau
![Logo](https://github.com/john-paul-31/Formula_1_Data_Analysis/blob/main/Logo.png)

## Database Schema
![App Screenshot](https://github.com/john-paul-31/Formula_1/blob/main/ER%20Diagram.png)

## Identify the Top 10 Most Successful Drivers.

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
### Result
| Driver             | Number_of_Wins |
|--------------------|--------------|
| Lewis Hamilton      | 103          |
| Michael Schumacher  | 91           |
| Sebastian Vettel    | 53           |
| Alain Prost         | 51           |
| Ayrton Senna        | 41           |
| Fernando Alonso     | 32           |
| Nigel Mansell       | 31           |
| Nelson Piquet       | 23           |
| Nico Rosberg        | 23           |
| Damon Hill          | 22           |
