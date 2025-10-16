
Committee ID: 62950
Committee Name: RAZE
##### c3 VR Data
```SQL
CREATE OR REPLACE TABLE `prod-organize-arizon-4e1c0a83.2024_program_summary.RAZE_VR_2024_PDC` AS(
SELECT 
'Rural Arizona Engagement' AS OrganizationName
, vr.Voter_File_VANID
, vr.voterbase_id
, 'RAZE' AS CommitteeName
, '62950' AS CommitteeID
, 'Voter Registration' AS ContactTypeName
, null AS ContactResultName
, cc.DateCanvassed
, 'AZ' AS State

FROM `prod-organize-arizon-4e1c0a83.2024_program_summary.RAZE_VR_2024` AS vr

LEFT JOIN `prod-organize-arizon-4e1c0a83.raze_ngpvan_data.TSM_OneAZ_ContactsContacts_VR` AS cc
  ON vr.Voter_File_VANID = cc.VanID
)
```
![[bquxjob_1bccd953_1961b2c196a.csv]]

##### c3 VAN all contact
```SQL
CREATE OR REPLACE TABLE `prod-organize-arizon-4e1c0a83.2024_program_summary.RAZE_VAN_ALL_2024_PDC` AS(
SELECT 
'Rural Arizona Engagement' AS OrganizationName
, cc.VanID
, vb.voterbase_id
, 'RAZE' AS CommitteeName
, '62950' AS CommitteeID
, ct.ContactTypeName
, ri.ResultShortName
, cc.DateCanvassed
, 'AZ' AS State
FROM `prod-organize-arizon-4e1c0a83.raze_ngpvan_data.TSM_OneAZ_ContactsContacts_VF` AS cc

LEFT JOIN `prod-organize-arizon-4e1c0a83.targetsmart_AZ.voter_base_latest` AS vb
  ON cc.VanID = vb.vb_smartvan_id

LEFT JOIN `prod-organize-arizon-4e1c0a83.viewers_dataset.contacttypeid` AS ct
  ON cc.ContactTypeID = ct.ContactTypeID

LEFT JOIN `prod-organize-arizon-4e1c0a83.viewers_dataset.resultid` AS ri
  ON cc.ResultID = ri.ResultID

WHERE EXTRACT(YEAR FROM cc.DateCanvassed) = 2024
)
![[bquxjob_5b02a5dd_1961b3a25c0.csv]]```
![[bquxjob_5b02a5dd_1961b3a25c0 1.csv]]