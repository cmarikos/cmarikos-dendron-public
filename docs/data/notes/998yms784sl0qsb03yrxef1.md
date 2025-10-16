###### Doing targeted re-reg and inactive registration prior to GOTV
Find bucks of inactive dems, people previously registered as dems, and high ideology scoring non-dems to get registered prior to the primary registration deadline

```SQL
WITH previous_dems AS(
  SELECT
  g.PCTNUM
  , COUNT(p.DWID) as previous_dems
  FROM `prod-organize-arizon-4e1c0a83.geofiles.az_precincts_geo` AS g

  LEFT JOIN `prod-organize-arizon-4e1c0a83.rich_christina_proj.catalist_pctnum_crosswalk_native` AS cw
    ON g.PCTNUM = cw.pctnum

  LEFT JOIN `proj-tmc-mem-mvp.catalist_cleaned.cln_catalist__district` AS d
    ON cw.uniqueprecinctcode = d.uniqueprecinctcode

  LEFT JOIN `proj-tmc-mem-mvp.catalist_cleaned.cln_catalist__person` AS p
    ON d.dwid = p.dwid

  WHERE g.CONGRESSIO = 7
    AND p.lastpartyaffiliation = 'DEM'
    AND p. partyaffiliation <> 'DEM'
    AND p.deceased <> 'Y'

  GROUP BY 1
)

, inactive_dems AS(
  SELECT
  g.PCTNUM
  , COUNT(p.DWID) as inactive_dems
  FROM `prod-organize-arizon-4e1c0a83.geofiles.az_precincts_geo` AS g

  LEFT JOIN `prod-organize-arizon-4e1c0a83.rich_christina_proj.catalist_pctnum_crosswalk_native` AS cw
    ON g.PCTNUM = cw.pctnum

  LEFT JOIN `proj-tmc-mem-mvp.catalist_cleaned.cln_catalist__district` AS d
    ON cw.uniqueprecinctcode = d.uniqueprecinctcode

  LEFT JOIN `proj-tmc-mem-mvp.catalist_cleaned.cln_catalist__person` AS p
    ON d.dwid = p.dwid
  
  WHERE g.CONGRESSIO = 7
    AND p. partyaffiliation = 'DEM'
    AND p.voterstatus = 'inactive'
    AND p.deceased <> 'Y'

  GROUP BY 1
)

, highideo_nondem AS(
  SELECT
  g.PCTNUM
  , COUNT(p.DWID) as highideo_nondems
  FROM `prod-organize-arizon-4e1c0a83.geofiles.az_precincts_geo` AS g

  LEFT JOIN `prod-organize-arizon-4e1c0a83.rich_christina_proj.catalist_pctnum_crosswalk_native` AS cw
    ON g.PCTNUM = cw.pctnum

  LEFT JOIN `proj-tmc-mem-mvp.catalist_cleaned.cln_catalist__district` AS d
    ON cw.uniqueprecinctcode = d.uniqueprecinctcode

  LEFT JOIN `proj-tmc-mem-mvp.catalist_cleaned.cln_catalist__person` AS p
    ON d.dwid = p.dwid
  
  LEFT JOIN `proj-tmc-mem-mvp.catalist_cleaned.cln_catalist__models` AS m
    ON p.dwid = m.dwid
  
  WHERE g.CONGRESSIO = 7
    AND p. partyaffiliation <> 'REP'
    AND m.catalistmodel_ideology_plus >= 80.0
    AND p.deceased <> 'Y'

  GROUP BY 1
)
SELECT
SUM(pd.previous_dems) AS previous_dems
, SUM(id.inactive_dems) AS inactive_dems
, SUM(hd.highideo_nondems) AS highideo_nondems
FROM `prod-organize-arizon-4e1c0a83.geofiles.az_precincts_geo` AS g

LEFT JOIN previous_dems AS pd
  ON g.PCTNUM = pd.PCTNUM

LEFT JOIN inactive_dems AS id
  ON g.PCTNUM = id.PCTNUM

LEFT JOIN highideo_nondem AS hd
  ON g.PCTNUM = hd.PCTNUM

WHERE g.CONGRESSIO = 7

```

###### A better way to determine previous dems -- table provided by Ashley Brennan
2020 registered dems who registered as another party for the 2024 election
```SQL
SELECT 
, COUNT(nd.dwid) AS voters
FROM `prod-organize-arizon-4e1c0a83.rich_christina_proj.dem2020notdem2024` AS nd

LEFT JOIN `proj-tmc-mem-mvp.catalist_cleaned.cln_catalist__district` AS d
  ON nd.dwid = d.dwid

LEFT JOIN `proj-tmc-mem-mvp.catalist_cleaned.cln_catalist__person` AS p
  ON nd.dwid = p.DWID

LEFT JOIN `prod-organize-arizon-4e1c0a83.rich_christina_proj.catalist_pctnum_crosswalk_native` AS cw
  ON d.uniqueprecinctcode = cw.uniqueprecinctcode

LEFT JOIN `prod-organize-arizon-4e1c0a83.geofiles.az_precincts_geo` AS g
  ON cw.pctnum = g.PCTNUM

WHERE g.CONGRESSIO = 7
```