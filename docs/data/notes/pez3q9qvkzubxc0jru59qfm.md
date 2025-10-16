
### Step 1: clean CVAP data

Begin by using CVAP ACS data to get LD
![[CVAP_2019-2023_ACS_csv_files.zip]]

Using the SLDUC file for LD since UC and LC are the same in AZ![[SLDUC.csv]]

Inspect the dataframe in python
```python
import pandas as pd

cvap_slduc = pd.read_csv('/content/SLDUC.csv')

print(cvap_slduc.head())
```
![[az_cvap_slduc.csv]]

Sample data:

| geoname                                 | lntitle                | geoid          | lnnumber | tot_est | tot_moe | adu_est | adu_moe | cit_est | cit_moe | cvap_est | cvap_moe |
| --------------------------------------- | ---------------------- | -------------- | -------- | ------- | ------- | ------- | ------- | ------- | ------- | -------- | -------- |
| State Senate District 1 (2022), Alabama | Not Hispanic or Latino | 610U800US01001 | 2        | 142975  | 1826    | 115170  | 1407    | 142130  | 1878    | 114330   | 1459     |

Base query for joining CVAP to geometry file
```sql
WITH a AS (
	SELECT
		ld_shortname
		, lntitle
		, adu_est
		, cit_est
		, cvap_est
	
	FROM `prod-organize-arizon-4e1c0a83.rich_christina_proj.az_cvap_slduc_2022`
)

SELECT
	ld.GEOMETRY
	, a.ld_shortname
	, a.lntitle
	, a.adu_est
	, a.cit_est
	, a.cvap_est

FROM `prod-organize-arizon-4e1c0a83.geofiles.az_legdistricts_geo` AS ld  

LEFT JOIN a
	ON ld.SHORTNAME = a.ld_shortname
```


Modeled race counts in Targetsmart 
```sql
SELECT DISTINCT
	tc.ld_shortname
	, vb.vb_voterbase_race
	,COUNT(vb.voterbase_id)

FROM `prod-organize-arizon-4e1c0a83.targetsmart_AZ.voter_base_latest` AS vb

LEFT JOIN `prod-organize-arizon-4e1c0a83.rich_christina_proj.tsmart_ld_crosswalk` AS tc
	ON CAST(vb.vb_tsmart_hd AS INT64) = tc.vb_tsmart_hd

WHERE vb.vb_tsmart_state = 'AZ'
	AND tc.ld_shortname IS NOT NULL

GROUP BY 1,2
```

-- current issue: CVAP race categories do not match voterbase race modeled categories. See mismatch mapped [here](https://docs.google.com/spreadsheets/d/1b0cPPyT3EGFvbTBDDb1cE3V1jL4Mr_BO_yoR7al9aOc/edit?usp=sharing).

Age ACS age buckets by target LD + precinct age maps