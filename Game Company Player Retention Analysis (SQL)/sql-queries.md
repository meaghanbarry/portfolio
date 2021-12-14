## Q1: Is 30-day rolling retention increasing or decreasing over the lifecycle of the game?

```
SELECT
 all_players.day_joined,
 player_count,
 retained,
 ROUND(retained / player_count, 3) AS fractional_retention

FROM (
 SELECT
   joined AS day_joined,
   COUNT(player_id) AS player_count
 FROM
   `project-test-329514.gamecompanydata.player_info`
 GROUP BY
   joined
 ) AS all_players -- table with day joined and total player count

JOIN (
 SELECT
   day_joined,
   SUM(CASE
     WHEN most_recent_game - day_joined >= 30 THEN 1
     ELSE 0
     END)
     AS retained
 FROM (
   SELECT
     p.player_id,
     joined AS day_joined,
     MAX(day) AS most_recent_game,
   FROM
     `project-test-329514.gamecompanydata.player_info` p
   JOIN
     `project-test-329514.gamecompanydata.matches_info` m
   ON
     p.player_id = m.player_id
   GROUP BY
     joined,
     p.player_id) -- table with player_id, day joined, and most recent game
 GROUP BY day_joined
 ) AS retained_players -- table with day joined and total retained player count

ON
 all_players.day_joined = retained_players.day_joined
```

## Q2: Does a player’s age affect their rolling 30-day retention rate?

```
Day Joined, Age, Player count, Retained, Not Retained.

SELECT
    joined,
    age,
     player_count,
    SUM(retained) AS totalRetained,
     player_count - SUM (Retained) AS NotRetained
FROM(
    SELECT
        id,
        joined,
        age,
        player_count,
        lastPlayed,
        CASE
            WHEN lastPlayed - joined >= 30 THEN 1
            ELSE 0
        END AS retained
    FROM(
        SELECT
            player.player_id AS id,
            joined,
            age,
            COUNT(*) OVER(PARTITION BY joined, age) AS  player_count,
            --get the last day the player played
            MAX(match.day) AS lastPlayed,
        FROM `project-1-st.gamecompanydata.player_info` AS player
        JOIN `project-1-st.gamecompanydata.matches_info` AS match
            ON player.player_id = match.player_id
        GROUP BY player.player_id, joined, age
    )
)
GROUP BY age,  player_count, joined
    HAVING totalRetained > 0
```


## Q3: For the players who played a match, does a player’s rate of winning (win percentage) affect their rolling 30-day retention rate?

```
SELECT
   player_id,
   retained,
   win_percentage,
   CASE
       WHEN win_percentage BETWEEN 0 AND 9.99 THEN "0-9.99%"
       WHEN win_percentage BETWEEN 10 AND 19.99 THEN "10-19.99%"
       WHEN win_percentage BETWEEN 20 AND 29.99 THEN "20-29.99%"
       WHEN win_percentage BETWEEN 30 AND 39.99 THEN "30-39.99%"
       WHEN win_percentage BETWEEN 40 AND 49.99 THEN "40-49.99%"
       WHEN win_percentage BETWEEN 50 AND 59.99 THEN "50-59.99%"
       WHEN win_percentage BETWEEN 60 AND 69.99 THEN "60-69.99%"
       WHEN win_percentage BETWEEN 70 AND 79.99 THEN "70-79.99%"
       WHEN win_percentage BETWEEN 80 AND 89.99 THEN "80-89.99%"
       WHEN win_percentage BETWEEN 90 AND 100.00 THEN "90-100%"
       ELSE NULL
       END AS win_percent_buckets,
   NTILE(10) OVER (ORDER BY win_percentage) AS win_percentile
FROM (
   SELECT
       p.player_id,
       CASE
           WHEN MAX(m.day) - p.joined >= 30 THEN 1
           ELSE 0
           END AS retained, -- MAX(m.day) = most recent game, 1 = retained, 0 = not retained
       ROUND((COUNTIF(outcome = "win") / COUNT(outcome))*100, 2) AS win_percentage -- COUNTIF(outcome = "win") = total wins, COUNT(outcome) = total matches
   FROM
       `project-test-329514.gamecompanydata.player_info` p
   JOIN
       `project-test-329514.gamecompanydata.matches_info` m
   ON
       p.player_id = m.player_id
   WHERE
       p.joined <= 335
   GROUP BY
       p.player_id,
       p.joined) -- table with player_id, day joined, most recent game, and win percentage
ORDER BY
   win_percentage
```
