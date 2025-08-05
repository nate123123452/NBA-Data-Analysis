# NBA-Data-Analysis

## Player Efficiency by Position
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


Pos	avg_efficiency
C	1152.9000000000000000
PG	1023.6400000000000000
SG	1008.6800000000000000
PF	918.9400000000000000
SF	865.8600000000000000
