###### GOTV IDs
```SQL
SELECT
  c.pctnum
  , c.county
  , d.congressionaldistrict
  , d.statehousedistrict
  , sq.SurveyQuestionName
	, sr.SurveyResponseName
  , COUNT(cw.DWID) AS response_count

FROM `prod-organize-arizon-4e1c0a83.ngpvan.CTARAA_ContactsSurveyResponses_VF` AS cs 

LEFT JOIN `prod-organize-arizon-4e1c0a83.ngpvan.CTARAA_SurveyQuestions` AS sq
	ON cs.SurveyQuestionID = sq.SurveyQuestionID

LEFT JOIN `prod-organize-arizon-4e1c0a83.ngpvan.CTARAA_SurveyResponses` AS sr
	ON cs.SurveyResponseID = sr.SurveyResponseID

LEFT JOIN `prod-organize-arizon-4e1c0a83.raze_raza_dwid_to_van_id_crosswalk.dwid_vanid_crosswalk` AS cw
  ON cs.VanID = cw.VanID

LEFT JOIN `proj-tmc-mem-mvp.catalist_cleaned.cln_catalist__district` AS d
  ON cw.DWID = d.dwid

INNER JOIN `prod-organize-arizon-4e1c0a83.rich_christina_proj.catalist_pctnum_crosswalk_native` AS c
  ON d.uniqueprecinctcode = c.uniqueprecinctcode

WHERE sq.Cycle ='2024'
	AND c.county in ('COCONINO','COCHISE','MOHAVE','PINAL','PIMA','YAVAPAI','YUMA')

GROUP BY 1,2,3,4,5,6
ORDER BY 1

```
##### race
```SQL
SELECT 
  p.pctnum
  , d.countyname
  , n.race
  , d.congressionaldistrict
  , d.statehousedistrict
  , COUNT(DISTINCT n.dwid) as people_count

FROM `proj-tmc-mem-mvp.catalist_enhanced.enh_catalist__ntl_current` AS n

LEFT JOIN `proj-tmc-mem-mvp.catalist_cleaned.cln_catalist__district` AS d
  ON n.dwid = d.dwid

LEFT JOIN `prod-organize-arizon-4e1c0a83.rich_christina_proj.catalist_pctnum_crosswalk_native` AS p
  ON d.uniqueprecinctcode = p.uniqueprecinctcode

WHERE n.state = 'AZ'
  AND p.pctnum IS NOT NULL

GROUP BY 1,2,3,4,5
```

file name: race_precinct_district.csv

##### age
```SQL
SELECT
  c.county
, c.pctnum
, b.congressionaldistrict
, b.statehousedistrict
, CASE
    WHEN a.age < 18 THEN "under 18"
    WHEN a.age BETWEEN 18 AND 24 THEN "18-24"
    WHEN a.age BETWEEN 25 AND 34 THEN "25-34"
    WHEN a.age BETWEEN 35 AND 44 THEN "35-44"
    WHEN a.age BETWEEN 45 AND 54 THEN "45-54"
    WHEN a.age BETWEEN 55 AND 64 THEN "55-64"
    WHEN a.age >= 65 THEN "65+"
    ELSE NULL
  END AS Age_Buckets
, COUNT(a.dwid) AS DWIDS
FROM `proj-tmc-mem-mvp.catalist_enhanced.enh_catalist__ntl_current` AS a
LEFT JOIN `proj-tmc-mem-mvp.catalist_cleaned.cln_catalist__district` AS b
  ON a.dwid = b.dwid
INNER JOIN `prod-organize-arizon-4e1c0a83.rich_christina_proj.catalist_pctnum_crosswalk_native` AS c
  ON b.uniqueprecinctcode = c.uniqueprecinctcode
WHERE a.state = "AZ"
GROUP BY 1, 2, 3, 4, 5

```

file name: age_precinct_district.csv

##### income
```sql
SELECT
  c.county
, c.pctnum
, b.congressionaldistrict
, b.statehousedistrict
, a.catalistmodel_income_bin
, COUNT(a.dwid) AS DWIDS
FROM `proj-tmc-mem-mvp.catalist_cleaned.cln_catalist__models` as a
LEFT JOIN `proj-tmc-mem-mvp.catalist_cleaned.cln_catalist__district` as b
  ON a.dwid = b.dwid
INNER JOIN `prod-organize-arizon-4e1c0a83.rich_christina_proj.catalist_pctnum_crosswalk_native` as c
  ON b.uniqueprecinctcode = c.uniqueprecinctcode
WHERE a.state = "AZ"
GROUP BY 1, 2, 3, 4, 5

```

file name: income_precinct_district.csv

##### volunteers
```SQL
SELECT
p.pctnum
, p.county
, cd.congressionaldistrict
, cd.statehousedistrict
, count(b.VanID) as potential_volunteers
FROM `prod-organize-arizon-4e1c0a83.ngpvan.CTARAA_ContactsSurveyResponses_VF` as b
LEFT JOIN `prod-organize-arizon-4e1c0a83.ngpvan.CTARAA_SurveyQuestions` as a
  on b.SurveyQuestionID = a.SurveyQuestionID
LEFT JOIN `prod-organize-arizon-4e1c0a83.ngpvan.CTARAA_SurveyResponses` as c
  on b.SurveyResponseID = c.SurveyResponseID
RIGHT JOIN `prod-organize-arizon-4e1c0a83.raze_raza_dwid_to_van_id_crosswalk.dwid_vanid_crosswalk` as cr
  on b.VanID = cr.VanID
LEFT JOIN `proj-tmc-mem-mvp.catalist_cleaned.cln_catalist__district` as cd
  on cr.DWID = cd.dwid
LEFT JOIN `prod-organize-arizon-4e1c0a83.rich_christina_proj.catalist_pctnum_crosswalk_native` as p
  on cd.uniqueprecinctcode = p.uniqueprecinctcode
WHERE extract(year from b.DateCanvassed) = 2024
  and a.SurveyQuestionName = 'RAZA Volunteer'
  and c.SurveyResponseName in ('Yes', 'Maybe Later')
  and cd.state = 'AZ'
  and p.pctnum IS NOT NULL
GROUP BY 1,2,3,4
ORDER BY 1
```


##### canvasser performance vote share
```sql
CREATE OR REPLACE TABLE `prod-organize-arizon-4e1c0a83.rich_christina_proj.vote_share_performance` AS (
WITH door_ratio AS(
SELECT 
r.pctnum
, (cr.Attempts /r.Registered_Total) AS door_ratio
, (r.PresidentDem_Harris /r.Registered_Dem) AS vote_share -- swap out with race
FROM `prod-organize-arizon-4e1c0a83.rich_christina_proj.results_2024` as r
LEFT JOIN `prod-organize-arizon-4e1c0a83.rich_christina_proj.canvassresults2024` as cr
  on r.pctnum = cr.pctnum
  WHERE cr.Attempts is not null
    AND cr.Attempts > 50
)

SELECT
p.COUNTY
, r.pctnum
, r.Congressional
, r.Legislative
, (dr.vote_share / dr.door_ratio) AS canvass_performance -- higher = better pres performance
, p.GEOMETRY
FROM `prod-organize-arizon-4e1c0a83.rich_christina_proj.results_2024` AS r

INNER JOIN door_ratio as dr
  ON r.pctnum = dr.pctnum

LEFT JOIN `prod-organize-arizon-4e1c0a83.geofiles.az_precincts_geo` AS p
  ON r.pctnum = p.PCTNUM

WHERE p.County IN ('Coconino', 'Cochise', 'Mohave', 'Pinal', 'Pima', 'Yavapai', 'Yuma')
)
```

https://lookerstudio.google.com/s/qFEMNUM2W_c


##### education
```SQL
SELECT
c.pctnum
, c.county
, b.congressionaldistrict
, b.statehousedistrict
,avg(a.catalistmodel_education_3_0) as avg_edscore
FROM `proj-tmc-mem-mvp.catalist_cleaned.cln_catalist__models` as a
left join `proj-tmc-mem-mvp.catalist_cleaned.cln_catalist__district` as b
on a.dwid=b.dwid
inner join `prod-organize-arizon-4e1c0a83.rich_christina_proj.catalist_pctnum_crosswalk_native` as c
on b.uniqueprecinctcode = c.uniqueprecinctcode
WHERE b.state = "AZ"
GROUP BY 1,2,3,4
ORDER BY 1
```