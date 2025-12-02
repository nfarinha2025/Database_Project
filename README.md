1.Database Summary Report

Liverpool Football Club.

Liverpool Football Club is one of the most successful and globally recognized football clubs, competing in the English Premier League. The club manages a large roster of professional players, coaches, medical staff, scouts, and administrative personnel. They operate out of Anfield as their home stadium and train at the AXA Training Centre. To maintain elite performance, Liverpool FC relies on detailed tracking of player performance, match statistics, training schedules, staff roles, and injuries.

Managing this information manually or across scattered documents creates inefficiencies. The club needs accurate, centralized information that coaches, analysts, and staff can use to make informed decisions. A relational database allows Liverpool FC to store mission-critical information in an organized, secure, and queryable way.

How Users Will Use the Database   

Coaches will use the database to view player details, check fitness status, track match performance, review goals and assists, and access training attendance. Analysts will use the database to run queries about match statistics, player availability, and performance trends. The medical team will manage injury histories and expected recovery dates. Administrative staff will add new players, schedule matches, and maintain staff records.

End users will interact with the database through SQL queries, views, and dashboards connected to applications or reporting tools. They will perform routine operations such as inserting match results, updating injuries, accessing performance reports, and reviewing roster changes. The database serves as a unified system supporting the entire professional football operations of Liverpool FC.

2.ER Model

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
        VARCHAR(10) venue
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
    STAFF   ||--|| STAFF_ROLES : "assigned"
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

### Create Tables
```
CREATE TABLE players (
    player_id INT(11) AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    birthdate DATE,
    position VARCHAR(30),
    nationality VARCHAR(30),
    squad_number INT(11)
);

CREATE TABLE matches (
    match_id INT(11) AUTO_INCREMENT PRIMARY KEY,
    match_date DATE,
    venue VARCHAR(10),
    opponent_id INT(11),
    competition_id INT(11),
    FOREIGN KEY (opponent_id) REFERENCES opponents(opponent_id),
    FOREIGN KEY (competition_id) REFERENCES competitions(competition_id)
);

CREATE TABLE player_stats (
    stat_id INT(11) AUTO_INCREMENT PRIMARY KEY,
    player_id INT(11),
    match_id INT(11),
    minutes_played INT(11),
    goals INT(11),
    assists INT(11),
    shots INT(11),
    passes INT(11),
    FOREIGN KEY (player_id) REFERENCES players(player_id),
    FOREIGN KEY (match_id) REFERENCES matches(match_id)
);

CREATE TABLE injuries (
    injury_id INT(11) AUTO_INCREMENT PRIMARY KEY,
    player_id INT(11),
    injury_type VARCHAR(100),
    date_injured DATE,
    expected_return DATE,
    FOREIGN KEY (player_id) REFERENCES players(player_id)
);

CREATE TABLE staff (
    staff_id INT(11) AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    hire_date DATE,
    role_id INT(11),
    FOREIGN KEY (role_id) REFERENCES staff_roles(role_id)
);

CREATE TABLE staff_roles (
    role_id INT(11) AUTO_INCREMENT PRIMARY KEY,
    role_name VARCHAR(50) NOT NULL
);

CREATE TABLE opponents (
    opponent_id INT(11) AUTO_INCREMENT PRIMARY KEY,
    opponent_name VARCHAR(100) NOT NULL,
    country VARCHAR(50),
    stadium VARCHAR(100)
);

CREATE TABLE competitions (
    competition_id INT(11) AUTO_INCREMENT PRIMARY KEY,
    competition_name VARCHAR(100) NOT NULL,
    competition_type VARCHAR(50)
);

```
