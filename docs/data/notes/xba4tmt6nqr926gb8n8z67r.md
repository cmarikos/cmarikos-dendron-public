Maps registered voters by precinct, county, ld, and cd by party

```sql
  

CREATE OR REPLACE VIEW `prod-organize-arizon-4e1c0a83.rich_christina_proj.registered_voters_by_pctnum_2024` AS (
	WITH a AS(
		SELECT DISTINCT
			cw.pctnum
			, vb.vb_vf_party as party
			, COUNT(vb.voterbase_id) registered_voters
		
		FROM `prod-organize-arizon-4e1c0a83.targetsmart_AZ.voter_base_latest` AS vb
		
		LEFT JOIN `prod-organize-arizon-4e1c0a83.rich_christina_proj.targetsmart_pctnum_crosswalk` as cw
			ON vb.vb_vf_precinct_name = cw.vb_vf_precinct_name
		
		WHERE vb.vb_tsmart_state = 'AZ'
		
		GROUP BY 1,2
		ORDER BY 1
	)
	
	SELECT
		pg.GEOMETRY
		, pg.COUNTY
		, pg.PRECINCTNA
		, pg.LEGISLATIV
		, pg.CONGRESSIO
		, a.party
		, a.registered_voters
	
	FROM `prod-organize-arizon-4e1c0a83.geofiles.az_precincts_geo` AS pg
	
	LEFT JOIN a
		ON pg.PCTNUM = a.pctnum
)
```

Age buckets place holder
```sql
SELECT distinct

CASE

WHEN vb.vb_voterbase_age BETWEEN 18 AND 24 THEN '18-24'

WHEN vb.vb_voterbase_age BETWEEN 25 AND 34 THEN '25-34'

WHEN vb.vb_voterbase_age BETWEEN 25 AND 34 THEN '25-34'

WHEN vb.vb_voterbase_age BETWEEN 35 AND 44 THEN '35-44'

WHEN vb.vb_voterbase_age BETWEEN 45 AND 54 THEN '45-54'

WHEN vb.vb_voterbase_age BETWEEN 55 AND 64 THEN '55-64'

WHEN vb.vb_voterbase_age > 65 THEN '65+'

ELSE 'age not specified'

END AS age_bucket

, COUNT(vb.voterbase_id) registered_voters

FROM `prod-organize-arizon-4e1c0a83.targetsmart_AZ.voter_base_latest` AS vb

  
  

WHERE vb.vb_tsmart_state = 'AZ'

  

GROUP BY 1

ORDER BY 1
```


#### LD17
```sql
  

CREATE OR REPLACE VIEW `prod-organize-arizon-4e1c0a83.rich_christina_proj.ld17_by_pctnum_2024` AS (

WITH a AS(

SELECT DISTINCT

cw.pctnum

, vb.vb_vf_party as party

,CASE

WHEN vb.vb_voterbase_age BETWEEN 18 AND 24 THEN '18-24'

WHEN vb.vb_voterbase_age BETWEEN 25 AND 34 THEN '25-34'

WHEN vb.vb_voterbase_age BETWEEN 25 AND 34 THEN '25-34'

WHEN vb.vb_voterbase_age BETWEEN 35 AND 44 THEN '35-44'

WHEN vb.vb_voterbase_age BETWEEN 45 AND 54 THEN '45-54'

WHEN vb.vb_voterbase_age BETWEEN 55 AND 64 THEN '55-64'

WHEN vb.vb_voterbase_age > 65 THEN '65+'

ELSE 'age not specified'

END AS age_bucket

, vb.vb_voterbase_gender

, vb.vb_voterbase_race as modeled_race

, COUNT(vb.vb_vf_early_voter_status) AS early_voters

, COUNT(vb.voterbase_id) AS registered_voters

FROM `prod-organize-arizon-4e1c0a83.targetsmart_AZ.voter_base_latest` AS vb

  

LEFT JOIN `prod-organize-arizon-4e1c0a83.rich_christina_proj.targetsmart_pctnum_crosswalk` as cw

ON vb.vb_vf_precinct_name = cw.vb_vf_precinct_name

  

WHERE vb.vb_tsmart_state = 'AZ'

  

GROUP BY 1,2,3,4,5

ORDER BY 1

)

  
  

SELECT

pg.GEOMETRY

, pg.COUNTY

, pg.PRECINCTNA

, pg.LEGISLATIV

, pg.CONGRESSIO

, a.party

, a.age_bucket

, a.vb_voterbase_gender

, a.modeled_race

, a.early_voters

, a.registered_voters

FROM `prod-organize-arizon-4e1c0a83.geofiles.az_precincts_geo` AS pg

  

LEFT JOIN a

ON pg.PCTNUM = a.pctnum

  

WHERE pg.LEGISLATIV = 17

  

)
```