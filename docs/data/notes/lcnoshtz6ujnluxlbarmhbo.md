
https://docs.google.com/spreadsheets/d/1TVktWZcKxL-Qjw_LxAJq70sXsfQM8TUEEn01eADM4_g/edit?usp=sharing


### Successes and Impact - "Please share at least three core ways in which your organization has had impact and met its mission over the last year. Highlights may include policy, litigation, or civic engagement outcomes; expanded or deepened constituent engagement; service delivery outcomes; or consequences of research, data collection, or coalition-building activities."

#### *Service delivery outcomes (civic engagement)*
- Voter registered - 
	- 5,352 (4,002 match to Targetsmart VoterFile)
- People canvassed/lit dropped knocked
- People who voted who we had touches with

*Consequences of research*
- Issue IDs - county demographics
- Discuss low turnout performance: 
- Where did you canvas

##### Voter Demographics
- Voter registered - 
	- 5,352 (4,002 match to Targetsmart VoterFile)

###### Race of voters registered
```SQL
SELECT 
vb.ts_tsmr_race
, COUNT(vr.voterbase_id)

FROM `prod-organize-arizon-4e1c0a83.2024_program_summary.RAZE_VR_2024` as vr

left join `prod-organize-arizon-4e1c0a83.targetsmart_AZ.voter_base_latest` as vb
  on vr.voterbase_id = vb.voterbase_id

GROUP BY 1
```

| race            | voters | percent |
| --------------- | ------ | ------- |
| Asian           | 15     | 0.4%    |
| Black           | 75     | 1.9%    |
| Native American | 13     | 0.3%    |
| White           | 1090   | 27.2%   |
| Hispanic        | 2797   | 69.9%   |
| unknown         | 12     | 0.3%    |
| unmatched       | 1350   |         |

###### Age of voters registered
```SQL
SELECT 
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
, COUNT(vr.voterbase_id)

FROM `prod-organize-arizon-4e1c0a83.2024_program_summary.RAZE_VR_2024` as vr

LEFT JOIN `prod-organize-arizon-4e1c0a83.targetsmart_AZ.voter_base_latest` as vb
  on vr.voterbase_id = vb.voterbase_id

GROUP BY 1
```

| age_bucket        | voters | percent |
| ----------------- | ------ | ------- |
| 18-24             | 1012   | 25%     |
| 25-34             | 828    | 21%     |
| 35-44             | 492    | 12%     |
| 45-54             | 444    | 11%     |
| 55-64             | 462    | 12%     |
| 65+               | 708    | 18%     |
| age not specified | 56     | 1%      |
| unmatched         | 1350   |         |
|                   |        |         |

###### Gender of registered voters
```SQL
SELECT 
vb.vb_voterbase_gender
, COUNT(vr.voterbase_id)

FROM `prod-organize-arizon-4e1c0a83.2024_program_summary.RAZE_VR_2024` AS vr

LEFT JOIN `prod-organize-arizon-4e1c0a83.targetsmart_AZ.voter_base_latest` AS vb
  on vr.voterbase_id = vb.voterbase_id

LEFT JOIN `prod-organize-arizon-4e1c0a83.targetsmart_AZ.vote_history_synthetics_latest` AS vh
  ON vb.voterbase_id = vh.voterbase_id

GROUP BY 1
```

| gender    | voters | percent |
| --------- | ------ | ------- |
| Female    | 1609   | 40.2%   |
| Male      | 1696   | 42.4%   |
| Unknown   | 697    | 17.4%   |
| Unmatched | 1350   |         |

###### Estimated income of voters
```SQL
SELECT 
ib.intp_estimated_income_range
, COUNT(vr.voterbase_id)

FROM `prod-organize-arizon-4e1c0a83.2024_program_summary.RAZE_VR_2024` AS vr

LEFT JOIN `prod-organize-arizon-4e1c0a83.targetsmart_AZ.voter_base_latest` AS vb
  ON vr.voterbase_id = vb.voterbase_id

LEFT JOIN `prod-organize-arizon-4e1c0a83.targetsmart_AZ.intellibase_platinum_latest` AS ib
  ON vb.voterbase_id = ib.voterbase_id

GROUP BY 1
ORDER BY 1
```

| estimated income bucket | voters | peercent |
| ----------------------- | ------ | -------- |
| Less than $20,000       | 355    | 19.6%    |
| $20,000-$29,999         | 295    | 16.3%    |
| C$30,000-$39,999        | 232    | 12.8%    |
| $40,000-$49,999         | 196    | 10.8%    |
| $50,000-$59,999         | 163    | 9.0%     |
| $60,000-$74,999         | 184    | 10.1%    |
| $75,000-$99,999         | 193    | 10.6%    |
| $100,000-$124,999       | 99     | 5.5%     |
| $125,000-$149,999       | 50     | 2.8%     |
| J$150,000-$199,999      | 35     | 1.9%     |
| $200,000-$249,999       | 3      | 0.2%     |
| $250,000-$499,999       | 7      | 0.4%     |
| $500,000+               | 1      | 0.1%     |
| Unmatched/unknown       | 3539   |          |
###### Urbanicity of voters registered
```SQL
SELECT 
vb.ts_tsmart_urbanicity
, COUNT(vr.voterbase_id)

FROM `prod-organize-arizon-4e1c0a83.2024_program_summary.RAZE_VR_2024` AS vr

left join `prod-organize-arizon-4e1c0a83.targetsmart_AZ.voter_base_latest` AS vb
  on vr.voterbase_id = vb.voterbase_id

GROUP BY 1
```

| urbanicity            | voters | percent |
| --------------------- | ------ | ------- |
| Rural 1 (least dense) | 358    | 8.95%   |
| Rural 2               | 1113   | 27.81%  |
| Suburban 3            | 1492   | 37.28%  |
| Suburban 4            | 924    | 23.09%  |
| Urban 5               | 112    | 2.80%   |
| Urban 6 (most dense)  | 3      | 0.07%   |
| Unmatched/unknown     | 1350   |         |

###### Gender

###### Were they first time registrants?
```SQL
SELECT 
vb.vb_vf_earliest_registration_date
, COUNT(vr.voterbase_id)

FROM `prod-organize-arizon-4e1c0a83.2024_program_summary.RAZE_VR_2024` AS vr

LEFT JOIN `prod-organize-arizon-4e1c0a83.targetsmart_AZ.voter_base_latest` AS vb
  on vr.voterbase_id = vb.voterbase_id

LEFT JOIN `prod-organize-arizon-4e1c0a83.targetsmart_AZ.vote_history_synthetics_latest` AS vh
  ON vb.voterbase_id = vh.voterbase_id

WHERE vb.vb_vf_earliest_registration_date >= 20240000

GROUP BY 1
```

Used whether they were first registered in 2024 A n indicator, there may be a more correct way to do this. 1,054 of 4,002 were first time .

Newly registered voters who voted in the general election
```SQL
SELECT 
COUNT(vr.voterbase_id)

FROM `prod-organize-arizon-4e1c0a83.2024_program_summary.RAZE_VR_2024` AS vr

LEFT JOIN `prod-organize-arizon-4e1c0a83.targetsmart_AZ.voter_base_latest` AS vb
  on vr.voterbase_id = vb.voterbase_id

LEFT JOIN `prod-organize-arizon-4e1c0a83.targetsmart_AZ.vote_history_synthetics_latest` AS vh
  ON vb.voterbase_id = vh.voterbase_id

WHERE vb.vb_vf_earliest_registration_date >= 20240000
AND vb.vb_vf_g2024 IS NOT NULL
```
###### Did they vote* 
```SQL
SELECT 
vb.vb_vf_g2024
, COUNT(vr.voterbase_id)

FROM `prod-organize-arizon-4e1c0a83.2024_program_summary.RAZE_VR_2024` AS vr

LEFT JOIN `prod-organize-arizon-4e1c0a83.targetsmart_AZ.voter_base_latest` AS vb
  on vr.voterbase_id = vb.voterbase_id

LEFT JOIN `prod-organize-arizon-4e1c0a83.targetsmart_AZ.vote_history_synthetics_latest` AS vh
  ON vb.voterbase_id = vh.voterbase_id

GROUP BY 1
```
 1,916 of 5,352 voted, roughly 48%

###### Did we do outreach to them in GOTV
We attempted 3,608 of the 5,352 voters registered by RAZE
We contacted to 240 of the 5,352 voters registered by RAZE
```SQL
SELECT
COUNT(vc.VanID) AS vr_outreach
FROM `prod-organize-arizon-4e1c0a83.2024_program_summary.RAZE_VR_2024` AS vr

LEFT JOIN `prod-organize-arizon-4e1c0a83.raze_ngpvan_data.TSM_OneAZ_ContactsContacts_VF` AS vc
  ON vr.Voter_File_VANID = vc.VanID

LEFT JOIN `prod-organize-arizon-4e1c0a83.viewers_dataset.ResultsID` AS ri
   ON vc.ResultID = ri.ResultID

WHERE ri.ResultShortName = 'Canvassed'
  AND vc.ContactTypeID = 2
  AND vc.DateCanvassed > '2024-09-01'
```

#### GOTV Outreach
###### Attempts and Canvassed
```SQL
SELECT
COUNT(vc.VanID) AS vr_outreach
FROM `prod-organize-arizon-4e1c0a83.raze_ngpvan_data.TSM_OneAZ_ContactsContacts_VF` AS vc

LEFT JOIN `prod-organize-arizon-4e1c0a83.viewers_dataset.ResultsID` AS ri
   ON vc.ResultID = ri.ResultID

WHERE vc.ContactTypeID = 2
  AND vc.DateCanvassed > '2024-09-01' 
  AND vc.DateCanvassed < '2025-01-01'
  --AND ri.ResultShortName = 'Canvassed'
```

###### County Breakdown
```SQL
SELECT
vb.vb_tsmart_county_name
, COUNT(vc.VanID) 
FROM `prod-organize-arizon-4e1c0a83.raze_ngpvan_data.TSM_OneAZ_ContactsContacts_VF` AS vc

LEFT JOIN `prod-organize-arizon-4e1c0a83.viewers_dataset.ResultsID` AS ri
   ON vc.ResultID = ri.ResultID
   
LEFT JOIN `prod-organize-arizon-4e1c0a83.targetsmart_AZ.voter_base_latest` AS vb
  ON vc.VanID = vb.vb_smartvan_id

WHERE vc.ContactTypeID = 2
  AND vc.DateCanvassed > '2024-09-01' 
  AND vc.DateCanvassed < '2025-01-01'
  --AND ri.ResultShortName = 'Canvassed'
  AND vb.vb_tsmart_state = 'AZ'

GROUP BY 1
ORDER BY 1
```

| county     | voters | percent |
| ---------- | ------ | ------- |
| ungeocoded | 183    | 0.13%   |
| COCHISE    | 236    | 0.16%   |
| COCONINO   | 11     | 0.01%   |
| GILA       | 3      | 0.00%   |
| GRAHAM     | 4      | 0.00%   |
| LA PAZ     | 1      | 0.00%   |
| MARICOPA   | 434    | 0.30%   |
| MOHAVE     | 6181   | 4.28%   |
| NAVAJO     | 6      | 0.00%   |
| PIMA       | 152    | 0.11%   |
| PINAL      | 5012   | 3.47%   |
| SANTA CRUZ | 12284  | 8.50%   |
| YAVAPAI    | 17     | 0.01%   |
| YUMA       | 119982 | 83.03%  |
###### GOTV contacts who voted

```SQL
SELECT
vb.vb_vf_g2024
, COUNT(vc.VanID) 
FROM `prod-organize-arizon-4e1c0a83.raze_ngpvan_data.TSM_OneAZ_ContactsContacts_VF` AS vc

LEFT JOIN `prod-organize-arizon-4e1c0a83.viewers_dataset.ResultsID` AS ri
   ON vc.ResultID = ri.ResultID
   
LEFT JOIN `prod-organize-arizon-4e1c0a83.targetsmart_AZ.voter_base_latest` AS vb
  ON vc.VanID = vb.vb_smartvan_id

WHERE vc.ContactTypeID = 2
  AND vc.DateCanvassed > '2024-09-01' 
  AND vc.DateCanvassed < '2025-01-01'
  AND ri.ResultShortName = 'Canvassed'
  --AND vb.vb_tsmart_state = 'AZ'

GROUP BY 1
ORDER BY 1
```

```SQL
SELECT
vb.vb_vf_g2024
, COUNT(vc.VanID) 
FROM `prod-organize-arizon-4e1c0a83.raze_ngpvan_data.TSM_OneAZ_ContactsContacts_VF` AS vc

LEFT JOIN `prod-organize-arizon-4e1c0a83.viewers_dataset.ResultsID` AS ri
   ON vc.ResultID = ri.ResultID
   
LEFT JOIN `prod-organize-arizon-4e1c0a83.targetsmart_AZ.voter_base_latest` AS vb
  ON vc.VanID = vb.vb_smartvan_id

WHERE vc.ContactTypeID = 2
  AND vc.DateCanvassed > '2024-09-01' 
  AND vc.DateCanvassed < '2025-01-01'
  --AND ri.ResultShortName = 'Canvassed'
  --AND vb.vb_tsmart_state = 'AZ'

GROUP BY 1
ORDER BY 1
```

Compared the contacts who voted to the full attempts universe and saw a much high rate in the contacts
-  62.3% contacts vs 45.8% attempts

Didn't vote in the past two general elections
```SQL
SELECT
COUNT(vc.VanID) 
FROM `prod-organize-arizon-4e1c0a83.raze_ngpvan_data.TSM_OneAZ_ContactsContacts_VF` AS vc

LEFT JOIN `prod-organize-arizon-4e1c0a83.viewers_dataset.ResultsID` AS ri
   ON vc.ResultID = ri.ResultID
   
LEFT JOIN `prod-organize-arizon-4e1c0a83.targetsmart_AZ.voter_base_latest` AS vb
  ON vc.VanID = vb.vb_smartvan_id

WHERE vc.ContactTypeID = 2
  AND vc.DateCanvassed > '2024-09-01' 
  AND vc.DateCanvassed < '2025-01-01'
  AND ri.ResultShortName = 'Canvassed'
  AND vb.vb_vf_g2024 IS NOT NULL
  AND vb.vb_vf_g2020 IS NULL
  AND vb.vb_vf_g2016 IS NULL
```

```SQL
SELECT

COUNT(vc.VanID)

FROM `prod-organize-arizon-4e1c0a83.raze_ngpvan_data.TSM_OneAZ_ContactsContacts_VF` AS vc

  

LEFT JOIN `prod-organize-arizon-4e1c0a83.viewers_dataset.ResultsID` AS ri

ON vc.ResultID = ri.ResultID

LEFT JOIN `prod-organize-arizon-4e1c0a83.targetsmart_AZ.voter_base_latest` AS vb

ON vc.VanID = vb.vb_smartvan_id

  

WHERE vc.ContactTypeID = 2

AND vc.DateCanvassed > '2024-09-01'

AND vc.DateCanvassed < '2025-01-01'

AND ri.ResultShortName = 'Canvassed'

AND vb.vb_vf_g2024 IS NOT NULL

AND vb_voterbase_general_votes = 1
```
 ##### Issue IDs
```SQL
SELECT DISTINCT
id.`2024_Issue_ID`
, COUNT(vb.voterbase_id)
FROM `prod-organize-arizon-4e1c0a83.rich_christina_proj.c3IssueIDs_2024` AS id

LEFT JOIN `prod-organize-arizon-4e1c0a83.targetsmart_AZ.voter_base_latest` AS vb
  ON id.`Voter File VANID` = vb.vb_smartvan_id

GROUP BY 1
```

| 2024_Issue_ID        | voters | percent |
| -------------------- | ------ | ------- |
| Paid Leave           | 53     | 0.6%    |
| Workers Rights       | 90     | 1.0%    |
| Gun Safety/GunViolnc | 187    | 2.0%    |
| Voting Rights        | 193    | 2.1%    |
| Environment          | 227    | 2.4%    |
| Housing              | 772    | 8.2%    |
| Immigration          | 775    | 8.3%    |
| Reproductive Freedom | 914    | 9.8%    |
| Education            | 948    | 10.1%   |
| Healthcare           | 1124   | 12.0%   |
| Economy/Jobs/Inflatn | 4089   | 43.6%   |
###### Find race and age demographics for Econ issue ID
```SQL
SELECT DISTINCT
vb.ts_tsmr_race
, COUNT(vb.voterbase_id)
FROM `prod-organize-arizon-4e1c0a83.rich_christina_proj.c3IssueIDs_2024` AS id

LEFT JOIN `prod-organize-arizon-4e1c0a83.targetsmart_AZ.voter_base_latest` AS vb
  ON id.`Voter File VANID` = vb.vb_smartvan_id

WHERE id.`2024_Issue_ID` = 'Economy/Jobs/Inflatn'

GROUP BY 1
```

| Economy/Jobs/Inflatn |        |         |
| -------------------- | ------ | ------- |
| ts_tsmr_race         | voters | percent |
| A                    | 162    | 4.0%    |
| N                    | 56     | 1.4%    |
| W                    | 1055   | 25.8%   |
| B                    | 196    | 4.8%    |
| H                    | 2600   | 63.6%   |
| U                    | 20     | 0.5%    |
|                      |        |         |
| age_bucket           | voters | percent |
| 18-24                | 387    | 9.5%    |
| 25-34                | 1276   | 31.2%   |
| 35-44                | 765    | 18.7%   |
| 45-54                | 531    | 13.0%   |
| 55-64                | 462    | 11.3%   |
| 65+                  | 620    | 15.2%   |
| age not specified    | 48     | 1.2%    |
###### Find age demographics for Healthcare IDs
```SQL
SELECT
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
, COUNT(vb.voterbase_id)
FROM `prod-organize-arizon-4e1c0a83.rich_christina_proj.c3IssueIDs_2024` AS id

LEFT JOIN `prod-organize-arizon-4e1c0a83.targetsmart_AZ.voter_base_latest` AS vb
  ON id.`Voter File VANID` = vb.vb_smartvan_id

WHERE id.`2024_Issue_ID` = 'Healthcare'

GROUP BY 1
ORDER BY 1
```

| Healthcare        |        |         |
| ----------------- | ------ | ------- |
| age_bucket        | voters | percent |
| 18-24             | 72     | 6.41%   |
| 25-34             | 197    | 17.53%  |
| 35-44             | 134    | 11.92%  |
| 45-54             | 131    | 11.65%  |
| 55-64             | 177    | 15.75%  |
| 65+               | 400    | 35.59%  |
| age not specified | 13     | 1.16%   |


##### Voters Contacted Demographics
```SQL
SELECT
vb.ts_tsmr_race
, COUNT(vb.voterbase_id) as voters
FROM `prod-organize-arizon-4e1c0a83.raze_ngpvan_data.TSM_OneAZ_ContactsContacts_VF` AS vc

LEFT JOIN `prod-organize-arizon-4e1c0a83.viewers_dataset.ResultsID` AS ri
   ON vc.ResultID = ri.ResultID

LEFT JOIN `prod-organize-arizon-4e1c0a83.targetsmart_AZ.voter_base_latest` AS vb
  ON vc.VanID = vb.vb_smartvan_id

WHERE vc.ContactTypeID = 2
  AND vc.DateCanvassed > '2024-09-01' 
  AND vc.DateCanvassed < '2025-01-01'
  AND ri.ResultShortName = 'Canvassed'

GROUP BY 1 
ORDER BY 1
```

| race | voters contacted | percent |
| ---- | ---------------- | ------- |
| A    | 50               | 0.49%   |
| B    | 113              | 1.10%   |
| H    | 8737             | 84.85%  |
| N    | 45               | 0.44%   |
| U    | 20               | 0.19%   |
| W    | 1231             | 11.95%  |


#### Electorate

https://censusreporter.org/profiles/04000US04-arizona/