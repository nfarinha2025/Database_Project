### 1.Database Summary Report

### Liverpool Football Club.

Liverpool Football Club is one of the most successful and globally recognized football clubs, competing in the English Premier League. The club manages a large roster of professional players, coaches, medical staff, scouts, and administrative personnel. They operate out of Anfield as their home stadium and train at the AXA Training Centre. To maintain elite performance, Liverpool FC relies on detailed tracking of player performance, match statistics, training schedules, staff roles, and injuries.

Managing this information manually or across scattered documents creates inefficiencies. The club needs accurate, centralized information that coaches, analysts, and staff can use to make informed decisions. A relational database allows Liverpool FC to store mission-critical information in an organized, secure, and queryable way.

### How Users Will Use the Database   

Coaches will use the database to view player details, check fitness status, track match performance, review goals and assists, and access training attendance. Analysts will use the database to run queries about match statistics, player availability, and performance trends. The medical team will manage injury histories and expected recovery dates. Administrative staff will add new players, schedule matches, and maintain staff records.

End users will interact with the database through SQL queries, views, and dashboards connected to applications or reporting tools. They will perform routine operations such as inserting match results, updating injuries, accessing performance reports, and reviewing roster changes. The database serves as a unified system supporting the entire professional football operations of Liverpool FC.

### 2.ER Model

``` mermaid
erDiagram
    PLAYERS {
        INT player_id PK
        VARCHAR(50) first_name
        VARCHAR(50) last_name
        DATE birthdate
        VARCHAR(30) position
        VARCHAR(30) nationality
        INT squad_number
    }

    MATCHES {
        INT match_id PK
        DATE match_date
        VARCHAR(30) venue
        INT opponent_id FK
        INT competition_id FK
    }

    PLAYER_STATS {
        INT stat_id PK
        INT player_id FK
        INT match_id FK
        INT minutes_played
        INT goals
        INT assists
        INT shots
        INT passes
    }

    INJURIES {
        INT injury_id PK
        INT player_id FK
        VARCHAR(100) injury_type
        DATE date_injured
        DATE expected_return
    }

    STAFF {
        INT staff_id PK
        VARCHAR(50) first_name
        VARCHAR(50) last_name
        DATE hire_date
        INT role_id FK
    }

    STAFF_ROLES {
        INT role_id PK
        VARCHAR(50) role_name
    }

    OPPONENTS {
        INT opponent_id PK
        VARCHAR(100) opponent_name
        VARCHAR(50) country
        VARCHAR(100) stadium
    }

    COMPETITIONS {
        INT competition_id PK
        VARCHAR(100) competition_name
        VARCHAR(50) competition_type
    }

    PLAYERS ||--o{ PLAYER_STATS : "records stats"
    MATCHES ||--o{ PLAYER_STATS : "includes stats"
    PLAYERS ||--o{ INJURIES : "suffers"
    STAFF }o--|| STAFF_ROLES : "assigned"
    MATCHES }o--|| OPPONENTS : "against"
    MATCHES }o--|| COMPETITIONS : "part of"
```

Overall Design Description 

The Liverpool FC database is designed around core operational entities: players, matches, player statistics, injuries, and staff. The central table is PLAYERS, which connects to both PLAYER_STATS and INJURIES through foreign keys. This allows a single player to have multiple match records and injury records. The MATCHES table allows each match to be linked to multiple player stat entries, ensuring a full statistical record.

Normalization was applied to avoid redundancy: player information appears only in PLAYERS; match-level details are isolated in MATCHES; player performance details are in PLAYER_STATS; and injuries are stored in INJURIES. No denormalization was required, but PLAYER_STATS is intentionally wide because each row describes a full performance summary for one player in one match.

Table Purpose 

PLAYERS — Stores the roster of professional Liverpool FC players, their squad numbers, nationality, birthdate, and playing positions. This table is referenced by several other tables because players are the foundation of the system.

MATCHES — Stores information about each Liverpool game, including date, opponent, competition, and venue. It is used to link performance statistics to actual match events.

PLAYER_STATS — Stores all match-specific performance data such as goals, assists, minutes played, and passes. It links each player to each match they participated in.

INJURIES — Tracks player injuries, injury types, and expected recovery dates so coaches and staff can monitor availability.

STAFF — Stores non-player personnel such as coaches, physios, and analysts, with their roles and hire dates.

---

## Create Tables
```sql

-- Base tables
CREATE TABLE players (
    player_id INT AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    birthdate DATE,
    position VARCHAR(30),
    nationality VARCHAR(30),
    squad_number INT
);

CREATE TABLE opponents (
    opponent_id INT AUTO_INCREMENT PRIMARY KEY,
    opponent_name VARCHAR(100) NOT NULL,
    country VARCHAR(50),
    stadium VARCHAR(100)
);

CREATE TABLE competitions (
    competition_id INT AUTO_INCREMENT PRIMARY KEY,
    competition_name VARCHAR(100) NOT NULL,
    competition_type VARCHAR(50)
);

CREATE TABLE staff_roles (
    role_id INT AUTO_INCREMENT PRIMARY KEY,
    role_name VARCHAR(50) NOT NULL
);

-- Dependent tables
CREATE TABLE matches (
    match_id INT AUTO_INCREMENT PRIMARY KEY,
    match_date DATE,
    venue VARCHAR(50),
    opponent_id INT,
    competition_id INT,
    FOREIGN KEY (opponent_id) REFERENCES opponents(opponent_id),
    FOREIGN KEY (competition_id) REFERENCES competitions(competition_id)
);

CREATE TABLE staff (
    staff_id INT AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    hire_date DATE,
    role_id INT,
    FOREIGN KEY (role_id) REFERENCES staff_roles(role_id)
);

-- Tables depending on matches/players
CREATE TABLE player_stats (
    stat_id INT AUTO_INCREMENT PRIMARY KEY,
    player_id INT,
    match_id INT,
    minutes_played INT,
    goals INT,
    assists INT,
    shots INT,
    passes INT,
    FOREIGN KEY (player_id) REFERENCES players(player_id),
    FOREIGN KEY (match_id) REFERENCES matches(match_id)
);

CREATE TABLE injuries (
    injury_id INT AUTO_INCREMENT PRIMARY KEY,
    player_id INT,
    injury_type VARCHAR(100),
    date_injured DATE,
    expected_return DATE,
    FOREIGN KEY (player_id) REFERENCES players(player_id)
);


```

## Insert Data

```sql
-- PLAYERS
INSERT INTO players (player_id, first_name, last_name, birthdate, position, nationality, squad_number) VALUES
(1, 'Mohamed', 'Salah', '1992-06-15', 'Forward', 'Egypt', 11),
(2, 'Virgil', 'van Dijk', '1991-07-08', 'Defender', 'Netherlands', 4),
(3, 'Alisson', 'Becker', '1992-10-02', 'Goalkeeper', 'Brazil', 1),
(4, 'Trent', 'Alexander-Arnold', '1998-10-07', 'Defender', 'England', 66),
(5, 'Jordan', 'Henderson', '1990-06-17', 'Midfielder', 'England', 14),
(6, 'Andrew', 'Robertson', '1994-03-11', 'Defender', 'Scotland', 26),
(7, 'Fabinho', 'Tavares', '1993-10-23', 'Midfielder', 'Brazil', 3),
(8, 'Roberto', 'Firmino', '1991-10-02', 'Forward', 'Brazil', 9),
(9, 'Sadio', 'Mané', '1992-04-10', 'Forward', 'Senegal', 10),
(10, 'James', 'Milner', '1986-01-04', 'Midfielder', 'England', 7),
(11, 'Naby', 'Keita', '1995-02-10', 'Midfielder', 'Guinea', 8),
(12, 'Joel', 'Matip', '1991-08-08', 'Defender', 'Cameroon', 32),
(13, 'Joe', 'Gomez', '1997-05-23', 'Defender', 'England', 12),
(14, 'Curtis', 'Jones', '2001-01-30', 'Midfielder', 'England', 17),
(15, 'Luis', 'Diaz', '1997-01-13', 'Forward', 'Colombia', 23),
(16, 'Darwin', 'Nunez', '1999-06-24', 'Forward', 'Uruguay', 27),
(17, 'Diogo', 'Jota', '1996-12-04', 'Forward', 'Portugal', 20),
(18, 'Thiago', 'Alcantara', '1991-04-11', 'Midfielder', 'Spain', 6),
(19, 'Ibrahima', 'Konate', '1999-05-25', 'Defender', 'France', 5),
(20, 'Caoimhin', 'Kelleher', '1998-11-23', 'Goalkeeper', 'Ireland', 62),
(21, 'Alex', 'Oxlade-Chamberlain', '1993-08-15', 'Midfielder', 'England', 15),
(22, 'Daniel', 'Sturridge', '1989-09-01', 'Forward', 'England', 15),
(23, 'Philippe', 'Coutinho', '1992-06-12', 'Midfielder', 'Brazil', 10),
(24, 'Pepe', 'Reina', '1982-08-31', 'Goalkeeper', 'Spain', 25),
(25, 'Steven', 'Gerrard', '1980-05-30', 'Midfielder', 'England', 8);

--  OPPONENTS
INSERT INTO opponents (opponent_id, opponent_name, country, stadium) VALUES
(1, 'Tottenham Hotspur', 'England', 'Tottenham Hotspur Stadium'),
(2, 'Crystal Palace', 'England', 'Selhurst Park'),
(3, 'Chelsea', 'England', 'Stamford Bridge'),
(4, 'Real Madrid', 'Spain', 'Santiago Bernabéu'),
(5, 'Southampton', 'England', 'St Mary''s Stadium'),
(6, 'Arsenal', 'England', 'Emirates Stadium'),
(7, 'Manchester United', 'England', 'Old Trafford'),
(8, 'Leicester City', 'England', 'King Power Stadium'),
(9, 'Manchester City', 'England', 'Etihad Stadium'),
(10, 'Napoli', 'Italy', 'Stadio Diego Armando Maradona'),
(11, 'Porto', 'Portugal', 'Estádio do Dragão');

-- STAFF_ROLES
INSERT INTO staff_roles (role_id, role_name) VALUES
(1, 'Manager'),
(2, 'Assistant Coach'),
(3, 'Goalkeeping Coach'),
(4, 'Fitness Coach'),
(5, 'Physio'),
(6, 'Analyst');


-- STAFF
INSERT INTO staff (staff_id, first_name, last_name, hire_date, role_id) VALUES
-- Manager
(1, 'Jürgen', 'Klopp', '2015-10-08', 1),

-- Assistant Coaches
(2, 'Pepijn', 'Lijnders', '2018-06-01', 2),
(3, 'Peter', 'Krawietz', '2015-10-08', 2),

-- Goalkeeping Coach
(4, 'John', 'Achterberg', '2009-06-01', 3),

-- Fitness Coaches
(5, 'Andreas', 'Kornmayer', '2016-07-01', 4),
(6, 'Andreas', 'Schlumberger', '2020-12-01', 4),

-- Medical / Physio
(7, 'Lee', 'Nobes', '2018-11-01', 5),
(8, 'Christopher', 'Roach', '2020-08-01', 5),

-- Analysts
(9, 'Greg', 'Mathieson', '2017-07-01', 6),
(10, 'James', 'French', '2012-07-01', 6);


-- COMPETITIONS
INSERT INTO competitions (competition_id, competition_name, competition_type) VALUES
(1, 'Premier League', 'League'),
(2, 'Champions League', 'Cup');

-- MATCHES
INSERT INTO matches (match_id, match_date, venue, opponent_id, competition_id) VALUES
(1, '2019-06-01', 'Neutral', 1, 2),
(2, '2020-06-25', 'Home', 2, 1),
(3, '2020-07-22', 'Home', 3, 1),
(4, '2021-05-23', 'Home', 2, 1),
(5, '2022-05-28', 'Neutral', 4, 2),
(6, '2023-05-28', 'Away', 5, 1),
(7, '2023-04-09', 'Home', 6, 1),
(8, '2023-03-05', 'Home', 7, 1),
(9, '2023-02-21', 'Home', 4, 2),
(10, '2022-12-30', 'Home', 8, 1),
(11, '2022-11-06', 'Away', 1, 1),
(12, '2022-10-16', 'Home', 9, 1),
(13, '2022-09-07', 'Away', 10, 2),
(14, '2021-12-19', 'Away', 1, 1),
(15, '2021-11-24', 'Home', 11, 2),
(16, '2021-10-24', 'Away', 7, 1),
(17, '2020-12-16', 'Home', 1, 1);

-- INJURIES
INSERT INTO injuries (injury_id, player_id, injury_type, date_injured, expected_return) VALUES
(1, 2, 'Knee Ligament Injury', '2020-10-17', '2021-07-01'),
(2, 1, 'Hamstring Injury', '2020-02-15', '2020-03-15'),
(3, 4, 'Ankle Injury', '2021-12-05', '2022-01-15'),
(4, 6, 'Groin Injury', '2019-09-20', '2019-10-20'),
(5, 9, 'Muscle Injury', '2018-11-10', '2018-12-05'),
(6, 10, 'Hamstring Injury', '2017-04-15', '2017-05-10'),
(7, 11, 'Thigh Injury', '2021-08-15', '2021-09-15'),
(8, 12, 'Knee Injury', '2019-02-10', '2019-03-20'),
(9, 13, 'Ankle Injury', '2020-01-05', '2020-02-10'),
(10, 14, 'Calf Injury', '2022-03-01', '2022-03-25'),
(11, 15, 'Hamstring Injury', '2023-01-10', '2023-02-05'),
(12, 16, 'Foot Injury', '2023-02-15', '2023-03-20');

-- PLAYER_STATS
INSERT INTO player_stats (stat_id, player_id, match_id, minutes_played, goals, assists, shots, passes) VALUES
(65, 1, 12, 90, 5, 1, 2, 70),
(66, 5, 1, 85, 0, 0, 1, 55),
(67, 6, 1, 90, 0, 0, 0, 60),
(68, 7, 1, 75, 0, 0, 1, 40),
(69, 8, 1, 80, 1, 0, 3, 45),
(70, 4, 1, 90, 4, 1, 2, 70),
(71, 5, 1, 85, 0, 0, 1, 55),
(72, 6, 1, 90, 0, 0, 0, 60),
(73, 7, 1, 75, 0, 0, 1, 40),
(74, 8, 1, 80, 1, 0, 3, 45),
(75, 4, 1, 90, 0, 1, 2, 70),
(76, 5, 1, 85, 0, 0, 1, 55),
(77, 6, 1, 90, 0, 0, 0, 60),
(78, 7, 1, 75, 0, 0, 1, 40),
(79, 8, 1, 80, 1, 0, 3, 45),
(80, 4, 1, 90, 0, 1, 2, 70),
(81, 5, 1, 85, 0, 0, 1, 55),
(82, 6, 1, 90, 0, 0, 0, 60),
(83, 7, 1, 75, 0, 0, 1, 40),
(84, 8, 1, 80, 1, 0, 3, 45),
(85, 4, 1, 90, 0, 1, 2, 70),
(86, 5, 1, 85, 0, 0, 1, 55),
(87, 6, 1, 90, 0, 0, 0, 60),
(88, 7, 1, 75, 0, 0, 1, 40),
(89, 8, 1, 80, 1, 0, 3, 45),
(90, 4, 1, 90, 0, 1, 2, 70),
(91, 5, 1, 85, 0, 0, 1, 55),
(92, 6, 1, 90, 0, 0, 0, 60),
(93, 7, 1, 75, 0, 0, 1, 40),
(94, 8, 1, 80, 1, 0, 3, 45),
(95, 4, 1, 90, 0, 1, 2, 70),
(96, 5, 1, 85, 0, 0, 1, 55),
(97, 6, 1, 90, 0, 0, 0, 60),
(98, 7, 1, 75, 0, 0, 1, 40),
(99, 8, 1, 80, 1, 0, 3, 45),
(100, 4, 1, 90, 0, 1, 2, 70),
(101, 5, 1, 85, 0, 0, 1, 55),
(102, 6, 1, 90, 0, 0, 0, 60),
(103, 7, 1, 75, 0, 0, 1, 40),
(104, 8, 1, 80, 1, 0, 3, 45),
(105, 4, 1, 90, 0, 1, 2, 70),
(106, 5, 1, 85, 0, 0, 1, 55),
(107, 6, 1, 90, 0, 0, 0, 60),
(108, 7, 1, 75, 0, 0, 1, 40),
(109, 8, 1, 80, 1, 0, 3, 45),
(110, 4, 1, 90, 0, 1, 2, 70),
(111, 5, 1, 85, 0, 0, 1, 55),
(112, 6, 1, 90, 0, 0, 0, 60),
(113, 7, 1, 75, 0, 0, 1, 40),
(114, 8, 1, 80, 1, 0, 3, 45),
(115, 4, 1, 90, 0, 1, 2, 70),
(116, 5, 1, 85, 0, 0, 1, 55),
(117, 6, 1, 90, 0, 0, 0, 60),
(118, 7, 1, 75, 0, 0, 1, 40),
(119, 8, 1, 80, 1, 0, 3, 45),
(120, 4, 1, 90, 0, 1, 2, 70),
(121, 5, 1, 85, 0, 0, 1, 55),
(122, 6, 1, 90, 0, 0, 0, 60),
(123, 7, 1, 75, 0, 0, 1, 40),
(124, 8, 1, 80, 1, 0, 3, 45),
(125, 4, 1, 90, 0, 1, 2, 70),
(126, 5, 1, 85, 0, 0, 1, 55),
(127, 6, 1, 90, 0, 0, 0, 60),
(128, 7, 1, 75, 0, 0, 1, 40),
(129, 8, 1, 80, 1, 0, 3, 45),
(130, 4, 1, 90, 0, 1, 2, 70),
(131, 5, 1, 85, 0, 0, 1, 55),
(132, 6, 1, 90, 0, 0, 0, 60),
(133, 7, 1, 75, 0, 0, 1, 40),
(134, 8, 1, 80, 1, 0, 3, 45);

```

## Queries

```sql
-- SELECT using ORDER BY two or more columns.

SELECT player_id, first_name, last_name, position, squad_number
FROM players
ORDER BY position ASC, squad_number ASC;

-- SELECT using a calculated field with a meaningful column heading.

SELECT stat_id, player_id, match_id,
       minutes_played / 60 AS hours_played
FROM player_stats
LIMIT 5;

-- SELECT using a MariaDB function

SELECT match_id, match_date, MONTH(match_date) AS match_month
FROM matches
LIMIT 5;

-- SELECT with an aggregation

SELECT player_id, SUM(goals) AS total_goals
FROM player_stats
GROUP BY player_id
HAVING SUM(goals) > 5
ORDER BY total_goals DESC;

-- Join of three or more tables

SELECT p.first_name, p.last_name, m.match_date, ps.goals
FROM player_stats ps
INNER JOIN players p ON ps.player_id = p.player_id
INNER JOIN matches m ON ps.match_id = m.match_id
WHERE ps.goals > 0
LIMIT 5;

--Left JOIN

SELECT s.staff_id, s.first_name, s.last_name, sr.role_name
FROM staff s
LEFT JOIN staff_roles sr ON s.role_id = sr.role_id;

-- UPDATE query

UPDATE staff
SET role_id = 4
WHERE staff_id = 1;

-- DELETE query

DELETE FROM injuries
WHERE injury_id = 12;

-- Create a Transaction with either ROLLBACK or COMMIT and demonstrate this transaction.

START TRANSACTION;

UPDATE players
SET squad_number = 99
WHERE player_id = 1;

-- Check temporary change
SELECT player_id, first_name, last_name, squad_number
FROM players
WHERE player_id = 1;

ROLLBACK;

-- Verify rollback
SELECT player_id, first_name, last_name, squad_number
FROM players
WHERE player_id = 1;
```

# Liverpool Player Performance Reports

## Reports

- [Total Goals by Player (PDF)](Total%20Goals%20by%20Player.pdf)
- [Player Performance Statistics (PDF)](stats.pdf)


These reports were built in **Microsoft Power BI**, a tool that makes it easy to turn data into clear visuals.
##

## Description

The Total Goals by Player report uses a clustered bar chart to compare cumulative goals across players. This visualization makes it easy to identify top scorers and quickly assess overall contributions. Salah stands out as the most consistent scorer, while Díaz and Núñez provide secondary support. The Goals per Match Report presents a table showing match dates, opponents, and goals scored by Salah, Díaz, and Núñez. This format highlights game‑by‑game contributions, revealing patterns such as Salah’s steady scoring and the more variable outputs of Díaz and Núñez. Together, the chart and table provide both a high‑level overview and detailed match insights. The combination demonstrates how Power BI can be used to capture trends and granular detail, strengthening the analysis of player impact.
##

## Delete
```sql
SET FOREIGN_KEY_CHECKS = 0;
-- Drop views first (if any exist)
DROP VIEW IF EXISTS player_stats_view;
DROP VIEW IF EXISTS match_summary_view;
DROP VIEW IF EXISTS staff_roles_view;

-- Drop tables 
DROP TABLE IF EXISTS player_stats;
DROP TABLE IF EXISTS matches;
DROP TABLE IF EXISTS players;
DROP TABLE IF EXISTS injuries;
DROP TABLE IF EXISTS staff;
DROP TABLE IF EXISTS staff_roles;
DROP TABLE IF EXISTS opponents;
DROP TABLE IF EXISTS competitions;
```

## Poster

- [Liverpool_fc_Poster (PDF)](Liverpool_fc_Poster.pdf)
