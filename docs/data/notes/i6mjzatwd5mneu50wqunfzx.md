#### c3 voter scores
```SQL
CREATE OR REPLACE VIEW `prod-organize-arizon-4e1c0a83.rich_christina_proj.c3_2024_voterpropensity` AS(
SELECT
vc.VanID
, cw.pctnum
, cw.vb_vf_county_name
, vb.vb_voterbase_voter_score
FROM `prod-organize-arizon-4e1c0a83.raze_ngpvan_data.TSM_OneAZ_ContactsContacts_VF` AS vc

LEFT JOIN `prod-organize-arizon-4e1c0a83.viewers_dataset.ResultsID` AS ri
   ON vc.ResultID = ri.ResultID

LEFT JOIN `prod-organize-arizon-4e1c0a83.targetsmart_AZ.voter_base_latest` AS vb
  ON vc.VanID = vb.vb_smartvan_id

LEFT JOIN `prod-organize-arizon-4e1c0a83.rich_christina_proj.targetsmart_pctnum_crosswalk` AS cw
  ON vb.vb_vf_national_precinct_code = cw.vb_vf_national_precinct_code


WHERE vc.ContactTypeID = 2
  AND vc.DateCanvassed > '2024-09-01' 
  AND vc.DateCanvassed < '2025-01-01'
  --AND ri.ResultShortName = 'Canvassed'
  AND vb.vb_voterbase_voter_score is not null
)
```

#### vote propensity
```SQL
CREATE OR REPLACE VIEW `prod-organize-arizon-4e1c0a83.rich_christina_proj.c4_2024_voterpropensity` AS(
SELECT
vc.VanID
, cw.pctnum
, cw.county
, m.catalistmodel_voteprop2024
FROM `prod-organize-arizon-4e1c0a83.ngpvan.CTARAA_ContactsContacts_VF` AS vc

LEFT JOIN `prod-organize-arizon-4e1c0a83.viewers_dataset.ResultsID` AS ri
   ON vc.ResultID = ri.ResultID

LEFT JOIN `prod-organize-arizon-4e1c0a83.raze_raza_dwid_to_van_id_crosswalk.dwid_vanid_crosswalk` AS dw
  ON vc.VanID = dw.VanID
  
LEFT JOIN `proj-tmc-mem-mvp.catalist_cleaned.cln_catalist__models` AS m
  ON dw.dwid = m.dwid

LEFT JOIN `proj-tmc-mem-mvp.catalist_cleaned.cln_catalist__district` AS d
  ON dw.dwid = d.dwid

LEFT JOIN `prod-organize-arizon-4e1c0a83.rich_christina_proj.catalist_pctnum_crosswalk_native` AS cw
  ON d.uniqueprecinctcode = cw.uniqueprecinctcode
  

WHERE vc.ContactTypeID = 2
  AND vc.DateCanvassed > '2024-01-01' 
  AND vc.DateCanvassed < '2025-01-01'
  --AND ri.ResultShortName = 'Canvassed'
)
```