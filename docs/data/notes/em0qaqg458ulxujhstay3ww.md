
###### A longer conversation about membership looms on the horizon

 [Climate Equity People Data 2023](https://docs.google.com/spreadsheets/d/1jWbStsSmk9t9J5q1WTgTMLICRifn6wp8UnOGkJDgmwk/edit?gid=0#gid=0)
  - the numbers in here are crazy and represent all of RAZA + RAZE outreach attempts, not even contacts
 Previously defined as:
- Base -  anyone we have ever attempted ever - this year we changed this to everyone we have had any conversation with (VR, campaign conversation, event attendance, organizing outreach)
- Membership - anyone who has come to an event, volunteered, raze leader/raza advocate, donor on actblue, basically any engagement
- Member leader - raze leader/raza advocate, unique volunteers, donors


Fernando meeting: https://docs.google.com/document/d/1SKRjyaD6c1XUOracWwQVBOCOCGc2oBP10UBl-ktb3wI/edit?tab=t.0

Membership definition notes: https://docs.google.com/document/d/1Nv7EjtA7e49ihFrXEingGsf2KWvJ8TtwndHVdHlCLi4/edit?tab=t.0


Climate Equity 2025 RFP https://docs.google.com/document/d/1rddMAtRmKmG6b5amvLCQ2_4C8q8J9xWttyrB03Yw1As/edit?tab=t.0

2025 people data for last year's 2024 c3 campaign cycle

|                   |                                                                                                                                                                                                                                  |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Member leaders    | People who have formal roles, titles, and/or clear responsibilities. They are involved in decision making about the activities of the organization. These people are unpaid, not counting stipends or meals, etc.                |
| Members           | People who volunteer time, attend meetings, take action with the organization (e.g., make phone calls, door knocking/canvassing), or pay dues to the organization. These people are unpaid, not counting stipends or meals, etc. |
| Base/constituency | People who receive regular, direct communication from the organization via doorknocking, phone calls, text messages or online                                                                                                    |
|                   |                                                                                                                                                                                                                                  |

From here down is the application
--------------------------------
[People Data 07212025 Climate Equity Fund](https://docs.google.com/spreadsheets/d/1Q1iNkE03Xuajpv51c8hoJu0nWxO4r3Wt58eDBSsm-2s/edit?gid=0#gid=0)

#### Base
 SmartVAN Arizona - RAZE Committee
![[Screenshot 2025-07-25 at 3.58.14 PM.png]]

Search saved in Climate Equity Folder as search titled - Climate Equity Base Full
+
##### Voter Demographics
- Voter registered - 
	- 5,352 (4,002 match to Targetsmart VoterFile)
-RACE + GENDER of Registered Voters
```SQL
SELECT 
vb.vb_voterbase_gender
, vb.ts_tsmr_race
, COUNT(vr.voterbase_id)

FROM `prod-organize-arizon-4e1c0a83.2024_program_summary.RAZE_VR_2024` as vr

left join `prod-organize-arizon-4e1c0a83.targetsmart_AZ.voter_base_latest` as vb
  on vr.voterbase_id = vb.voterbase_id

GROUP BY 1,2
ORDER BY 1,2
```

|                     |                 |                  |
| ------------------- | --------------- | ---------------- |
| vb_voterbase_gender | ts_tsmr_race    | voter_registered |
|                     |                 | 40               |
| Female              | AAPI            | 2                |
| Female              | Black           | 15               |
| Female              | Hispanic        | 1120             |
| Female              | Native American | 5                |
| Female              | Uncoded         | 6                |
| Female              | White           | 448              |
| Male                | AAPI            | 6                |
| Male                | Black           | 15               |
| Male                | Hispanic        | 1183             |
| Male                | Native American | 6                |
| Male                | Uncoded         | 3                |
| Male                | White           | 457              |
| Unknown             | AAPI            | 7                |
| Unknown             | Black           | 45               |
| Unknown             | Hispanic        | 468              |
| Unknown             | Native American | 2                |
| Unknown             | Uncoded         | 5                |
| Unknown             | White           | 169              |

#### Base EJ
![[Screenshot 2025-07-25 at 3.58.14 PM.png]]

Plus 
![[Screenshot 2025-07-31 at 8.33.23 AM.png]]

The list is 20 people :/ and is saved in the Climate Equity folder with Climate Equity Base 2024 EJ IDs

#### Members

Volunteers and event attendees between 06/01/2024 and 11/30/2024 logged in EA Movement Cooperative + Education roster

###### 1. Ran SQL query - saved in [organizing-dashboard](https://github.com/cmarikos/organizing-dashboard/blob/main/unique_attendee_volunteers_legacy.sql)
```SQL
WITH main_query AS (
  WITH unique_attendees AS (
    SELECT DISTINCT
      vanid,
      'Volunteer' AS role
    FROM `proj-tmc-mem-mvp.everyaction_enhanced.enh_everyaction__events` 
    WHERE start_date > '2024-06-01'
      AND start_date < '2024-11-30'
      AND status = 'Completed'
      AND role = 'Volunteer'
      AND current_status = 1
  )

  SELECT 
    c.first_name || " " || c.last_name AS full_name,
    e.vanid,
    a.role,
    STRING_AGG(DISTINCT e.event_name, ', ') AS agg_events,
    ARRAY_LENGTH(ARRAY_AGG(DISTINCT e.event_name)) AS event_count,

    -- NEW: how many DISTINCT events where this person was a Volunteer & Completed & current_status=1
    COUNT(DISTINCT IF(e.role = 'Volunteer'
                      AND e.status = 'Completed'
                      AND e.current_status = 1,
                      e.event_name, NULL)) AS volunteer_event_count

  FROM `proj-tmc-mem-mvp.everyaction_enhanced.enh_everyaction__events` AS e
  LEFT JOIN unique_attendees AS a
    ON e.vanid = a.vanid
  LEFT JOIN `proj-tmc-mem-mvp.everyaction_enhanced.enh_everyaction__contacts` AS c
    ON e.vanid = c.vanid
  WHERE e.start_date > '2024-06-01'
    AND e.start_date < '2024-11-30'
    AND status = 'Completed'
  GROUP BY 1, 2, 3
)

SELECT 
  CAST(c.utc_datecreated AS DATE) date_of_first_engagement,
  m.full_name,
  m.vanid,
  CASE WHEN m.role IS NULL THEN "" ELSE m.role END AS is_volunteer,
  m.agg_events,
  m.event_count,
  IFNULL(m.volunteer_event_count, 0) AS volunteer_event_count,   -- ← just return it
  (
    SELECT STRING_AGG(DISTINCT oe.organizer, ', ')
    FROM `prod-organize-arizon-4e1c0a83.organizing_view.organizing_user_events` AS oe
    WHERE oe.event_name IN UNNEST(SPLIT(m.agg_events, ', '))
  ) AS agg_organizers
FROM main_query AS m
LEFT JOIN `proj-tmc-mem-mvp.everyaction_enhanced.enh_everyaction__contacts` AS c
  ON m.vanid = c.vanid

```

Create sheet [demographics-climate action-2024](https://docs.google.com/spreadsheets/d/1DlwEJJtWSzHm8ZH1BM-HEMXQZV3NJJJoxg43hMq4W6g/edit?gid=214969885#gid=214969885)
- Insert race and gender drop downs for organizing team to manually fill out
- Made a pivot table to get gender and race counts

|   |
|---|
|volunteers|
|18|

|                                   |        |        |                                              |        |                 |
| --------------------------------- | ------ | ------ | -------------------------------------------- | ------ | --------------- |
| race                              |        | Man    | Transgender/Gender Nonconforming/Genderqueer | Woman  | **Grand Total** |
|                                   | 21     | 1      |                                              | 2      | **24**          |
| Asian American / Pacific Islander |        | 1      |                                              |        | **1**           |
| Black                             |        | 1      |                                              |        | **1**           |
| Latinx                            |        | 11     |                                              | 25     | **36**          |
| Multiracial                       |        |        |                                              | 2      | **2**           |
| Native American                   |        |        |                                              | 1      | **1**           |
| White                             |        | 9      | 2                                            | 21     | **32**          |
| **Grand Total**                   | **21** | **23** | **2**                                        | **51** | **97**          |
|                                   |        |        |                                              |        |                 |
|                                   |        |        |                                              |        |                 |
+
****

[Full Education Roster](https://docs.google.com/spreadsheets/d/127WCJ7rpOJzxfYClv5zFb4khy5D-MJWN0GTzVzYXzx4/edit?gid=0#gid=0)
- include grads from Pinal cohorts 5,6,7,8 (42) & Yuma 6,7,8,9 (85) = 127

={'Full Roster'!A35:X76; 'Full Roster'!A149:X244; 'Full Roster'!A291:X307}

Education Full roster filter to anyone who has a program in 2024 

|                 |     |        |      |             |
| --------------- | --- | ------ | ---- | ----------- |
| race            |     | Female | Male | Grand Total |
|                 | 0   |        |      | 0           |
| Black           |     | 2      | 4    | 6           |
| Hispanic        |     | 75     | 55   | 130         |
| Multiracial     |     | 1      |      | 1           |
| Multiracial     |     | 1      |      | 1           |
| Native American |     | 1      | 1    | 2           |
| Native American |     | 2      | 2    | 4           |
| White           |     | 6      | 5    | 11          |
| Grand Total     | 0   | 88     | 67   | 155         |