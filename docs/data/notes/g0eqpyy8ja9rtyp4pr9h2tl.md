```SQL
CREATE OR REPLACE VIEW `prod-organize-arizon-4e1c0a83.viewers_dataset.cd7_2025_dashboard_geo` AS(
WITH date_fix AS(
  SELECT
  contacts_contact_id
  , CAST(utc_canvassed_at AS DATE) AS date_fix
  FROM `proj-tmc-mem-mvp.ngpvan_cleaned.cln_ngpvan__contacts_contacts`
)

, canvass_result AS(
  SELECT
  contacts_contact_id
  , CASE 
    WHEN result_name = 'Canvassed' THEN 'Canvassed'
    ELSE 'Not Canvassed'
  END AS canvass_result
  FROM `proj-tmc-mem-mvp.ngpvan_cleaned.cln_ngpvan__contacts_contacts`
)

, canvass_bool AS(
  SELECT
  contacts_contact_id
  , CASE 
    WHEN result_name = 'Canvassed' THEN 1
    ELSE 0
  END AS canvass_bool
  FROM `proj-tmc-mem-mvp.ngpvan_cleaned.cln_ngpvan__contacts_contacts`
)

SELECT
cc.van_id
, p.PRECINCTNA
, p.COUNTY
, df.date_fix
, p.GEOMETRY
, p.CONGRESSIO
, cr.canvass_result
, cb.canvass_bool
FROM `proj-tmc-mem-mvp.ngpvan_cleaned.cln_ngpvan__contacts_contacts` AS cc

LEFT JOIN date_fix AS df
  ON cc.contacts_contact_id = df.contacts_contact_id

LEFT JOIN canvass_result AS cr
  on cc.contacts_contact_id = cr.contacts_contact_id

LEFT JOIN canvass_bool AS cb
  on cc.contacts_contact_id = cb.contacts_contact_id

LEFT JOIN `prod-organize-arizon-4e1c0a83.raze_raza_dwid_to_van_id_crosswalk.dwid_vanid_crosswalk` AS cw
  ON cc.van_id = cw.VanID

LEFT JOIN `proj-tmc-mem-mvp.catalist_cleaned.cln_catalist__district` AS d
  ON cw.DWID = d.dwid

INNER JOIN `prod-organize-arizon-4e1c0a83.rich_christina_proj.catalist_pctnum_crosswalk_native` AS cn
  ON d.uniqueprecinctcode = cn.uniqueprecinctcode

LEFT JOIN `prod-organize-arizon-4e1c0a83.geofiles.az_precincts_geo` AS p
  ON cn.pctnum = p.PCTNUM

WHERE contact_method_type = 'Walk'
AND df.date_fix > '2025-01-01'
AND p.COUNTY <> 'Yavapai'
)
```