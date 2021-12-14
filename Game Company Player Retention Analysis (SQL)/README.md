# Game Company Data Analysis

## Background

We've been hired by a mobile gaming company to answer the following questions for their one year anniversary:

1. Is 30-day rolling retention increasing or decreasing over the lifecycle of the game?
2. Does a player’s age affect their rolling 30-day retention rate?
3. For the players who played at least one match, did a player’s rate of winning (win percentage) affect their rolling 30-day retention rate?

We have used BigQuery and Sheets for analysis and visualization, and have been provided with the following data tables:

* Match information, including the players who matched against each other, and the outcome
* Player information, including information like the player's age and when they joined

## Q1: Is 30-day rolling retention increasing or decreasing over the lifecycle of the game?

### Query:
To find the total number of players who joined per day (regardless of whether or not they ever played a match), we first used the player_info table to count the player_id, grouped by the day they joined.

We then joined the players_info table with the matches_info table on player_id, and returned the player_id, day joined, and the player’s most recently played game day (using a `MAX` function). We then nested this statement and used a `CASE` statement to determine player retention: if the last match a player played was 30 or more days since the day they joined, we assigned a retained value of 1. Otherwise, we assigned a value of 0, meaning the player was not retained.

Finally, we returned a table with the join date, player count, retention status, and fractional retention (by dividing the retained player count by the total player count).

### Visualization:
We created two visualizations to analyse our data:

1. An area chart that shows the count of total players (grey) compared to the retained players (green) over the days they initially joined. In this chart view, there appear to be a few noticeable peaks in the retained player count (e.g. day 55, 277) and a significant drop at day 127. However, these data points are quite dependent on the total player count for the day, and as we can see from the retained player trendline, overall the retention rate was quite consistent throughout the year (average of 71%).

![Player Retention (Count) vs. Day Joined Chart](Visualizations/Player-Retention-Count-vs.-Day-Joined.png)

2. A 100% stacked area chart that shows the average player retention percentage over days joined. This view clearly depicts the consistency of the retention percentage over time.

![Player Retention Percentage vs. Day Joined Chart](Visualizations/Player-Retention-Percentage-vs.-Day-Joined.png)

Note: As our retention rate criteria is based on knowing if a player played a game 30 days after their join date, we omitted data for players who joined between day 335 and 365, as these players had no opportunity to meet the criteria. We also noted that the players who joined closer to day 335 had less opportunity to meet the retention criteria, which should be considered upon chart review.

## Q2: Does a player’s age affect their rolling 30-day retention rate?

### Query:
We again joined the matches_info and player_info tables on player_id to find the player count, partitioned by the day each player joined the game and their age. To find out if the players were retained, we calculated their most recent game played by using a similar `CASE` statement as described above in our first question query, assigning a value of 1 for retained, and 0 for not retained. We have also aggregated the Retained players and subtracted that from the player count to find out the sum of the players that weren’t retained.

### Visualization:
Age has a major impact on the 30 day retention rate. We have created a column chart to find out that the players close to 20 years of age have the highest player count but the lowest retention rate. Marketing advice would be to investigate further on how to increase the retention rate for the players around the age of 20. The Game Company should also look into optimizing the features and difficulty level of the game in order to retain more of the players that are around the age group of (17-23) Looking at the data there is a huge market potential for the players in their early teens as about 100% of the players that have joined the game are being retained. Also, the game company should focus their efforts towards increasing the number of players in their early thirties to join because these players have more spending power and higher retention rate.

![Age vs. Player Retention Count Chart](Visualizations/Age-vs.-Player-Retention-Count.png)
![Age vs. Retention Rate Chart](Visualizations/Age-vs.-Retention-Rate.png)


## Q3: For the players who played at least one match, did a player’s rate of winning (win percentage) affect their rolling 30-day retention rate?

### Query:
We first joined the player_info and player_matches tables together on player_id. From these tables, we filtered out all the players who joined after day 335 as they were not able to meet the retention criteria. We then returned the player_id, and engineered two features (grouped by player_id and day_joined):

(1) Retention of each player using a `CASE` statement.
If the last match a player played (calculated with a `MAX` function) was 30 or more days since the day they joined, we assigned a retained value of 1. Otherwise, we assigned a value of 0, meaning the player was not retained.

(2) Overall win percentage for each player.

We divided the count of each “win” outcome by the count of all match outcomes (aka every match played) and converted it into a percentage and rounded to two decimal places.
We then nested this query, and in the second query we returned the player_id, retention status, win percentage, and engineered two additional features to further explore the win percentages:
* **Ten win percentage buckets** (of approximately 10% per bucket) using a `CASE` statement.
* **Distributed the win percentages into ten groups** using the `NTILE` function.

### Visualization:
When we compared the average win percentage for the retained players vs. non retained players, the results were not significant - so we created two visualizations to further analyse our win percentage data:

1. A clustered column chart comparing total retained players and total not retained players by win percent bucket, showing that the most retained players (by count) had win percentages between 50-59.99%.

![Win Average % bucketed vs. Player Count Chart](Visualizations/Win-Average-percent-bucketed-vs.-Player-Count.png)

2. A column chart that shows average player retention rate by group. This visualization was of particular interest, as the first and tenth groups (the lowest and highest win percentages, respectively) had significantly lower retention than the other groups: across all 10, the average retention rate was 71.16% (approx. 2886 players retained of a total 4055 per grouping) -- while the first group showed a rate of 58.58%, and the tenth group showed a rate of 59.31%. This insight could indicate that the two biggest deterrents for retention were being heavily under or overskilled, though it is worth noting that the total number of matches per player hasn’t been factored into this analysis -- we are including any player that played one or more matches. We recommend experimenting with dynamic difficulty adjustment (DDA) to automatically modify the game's features, behaviors, and scenarios in real-time, depending on the player's skill.

![Win Percentile vs. Retained Player Retention Rate Chart](Visualizations/Win-Percentile-vs.-Retained-Player-Retention-Rate.png)

Link to data and visualizations:
https://docs.google.com/spreadsheets/d/1tLqhKBL2LvLfDe-YP6LOFE2ouZsBOzfOdfH_KwNvEHI/edit?usp=sharing
