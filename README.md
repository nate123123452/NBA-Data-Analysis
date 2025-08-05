# NBA-Data-Analysis

## Player Efficiency by Position (Query 1)
Calculate a simple custom efficiency score for NBA players based on:
Efficiency = (PTS + TRB + AST) - (FGA - FG + TOV)
(Points + Rebounds + Assists, minus Missed FG Attempts and Turnovers)

### SQL Query
```sql
WITH ranked_players AS (
  SELECT *,
         ROW_NUMBER() OVER (PARTITION BY "Pos" ORDER BY ("PTS" + "TRB" + "AST") - (("FGA" - "FG") + "TOV") DESC) AS pos_rank
  FROM nba_staging_full
)
SELECT 
  "Pos",
  AVG(("PTS" + "TRB" + "AST") - (("FGA" - "FG") + "TOV")) AS avg_efficiency
FROM 
  ranked_players
WHERE 
  pos_rank <= 50
GROUP BY 
  "Pos"
ORDER BY 
  avg_efficiency DESC;
```

### Results
| Position | Avg Efficiency | 
|----------|----------------|
| C        | 1152.90        |           
| PG       | 1023.64        |             
| SG       | 1008.68        |             
| PF       | 918.94         |           
| SF       | 865.86         |      

## Top 10% Players by Defensive Impact (Steals + Blocks per Game) by Position (Query 2)

### SQL Query
```sql
WITH DefensiveStats AS (
  SELECT
    "Player",
    "Pos",
    "STL",
    "BLK",
    "G",
    (CAST("STL" AS FLOAT) + CAST("BLK" AS FLOAT)) / NULLIF("G", 1) AS "DeflectionsPerGame",
    RANK() OVER (PARTITION BY "Pos" ORDER BY (CAST("STL" AS FLOAT) + CAST("BLK" AS FLOAT)) / NULLIF("G", 1) DESC) AS "PosRank",
    COUNT(*) OVER (PARTITION BY "Pos") AS "PosCount"
  FROM nba_staging_full
  WHERE "G" > 10
)

SELECT
  "Player",
  "Pos",
  "DeflectionsPerGame"
FROM DefensiveStats
WHERE "PosRank" <= CEIL("PosCount" * 0.10)
ORDER BY "Pos", "DeflectionsPerGame" DESC;
```
### Results
See CSV File

## Team Performance (Query 3)

### SQL Query
```sql
WITH team_stats AS (
  SELECT 
    tm AS team,
    ROUND(AVG(pts), 1) AS avg_ppg,
    ROUND(AVG(trb), 1) AS avg_rpg,
    ROUND(AVG(ast), 1) AS avg_apg,
    ROUND(AVG(blk), 1) AS avg_bpg,
    ROUND(AVG(stl), 1) AS avg_spg,
    ROUND(AVG(fg_pct)*100, 1) AS fg_pct,
    SUM(CASE WHEN pts > 20 THEN 1 ELSE 0 END) AS twenty_plus_ppg_players
  FROM nba_player_stats
  GROUP BY tm
)
SELECT 
  team,
  avg_ppg,
  avg_rpg,
  avg_apg,
  CONCAT(fg_pct, '%') AS fg_percentage,
  avg_spg || ' / ' || avg_bpg AS steals_blocks,
  twenty_plus_ppg_players,
  RANK() OVER (ORDER BY avg_ppg DESC) AS offensive_rank
FROM team_stats
ORDER BY avg_ppg DESC
LIMIT 10;
```

### Results
See CSV File
