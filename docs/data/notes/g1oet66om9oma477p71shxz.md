
<iframe width="600" height="450" src="https://lookerstudio.google.com/embed/reporting/a3f3129f-dc36-47b1-91a7-a04d938ec1c1/page/zLjcE" frameborder="0" style="border:0" allowfullscreen sandbox="allow-storage-access-by-user-activation allow-scripts allow-same-origin allow-popups allow-popups-to-escape-sandbox"></iframe>

[Connected sheet categorizing events by organizer - Extract 3](https://docs.google.com/spreadsheets/d/1VJ9Q_i2ehaiS3U7g_1ElCt-cZfryS9I0eccgqRl5XJg/edit?gid=242540621#gid=242540621)


###### Issue tracker

| 03/03/2025     | Engaged volunteers + super volunteers aren't displaying as they should |
| -------------- | ---------------------------------------------------------------------- |
|                | - Resolution date                                                      |
| 03/03/2025     | No date control filter                                                 |
| **03/12/2025** | - Resolution date                                                      |

- engaged volunteers + super volunteers aren't grabbing people as they should (03/02/2025)


**volunteershifts02182025**_run1st -- volunteer shift meters, engaged and super volunteer lists
This has to run before other tables
____________________
``` SQL
`CREATE OR REPLACE VIEW prod-organize-arizon-4e1c0a83.organizing_view.volunteershifts02182025 AS (`

  `-- Select the necessary columns from the event data and related tables`
  `SELECT` 
    `e.eventid,                         -- Event ID`
    `es.vanid,                          -- VAN ID 
    `e.event_name,                      -- Event name`
    `e.eventsignupid,                   -- Event signup ID`
    `e.role,                            -- Role associated with the event` attendee
    `e.status,                          -- Status of the event` attendee
    `e.start_date,                      -- Start date of the event`
    `e.end_date,                        -- End date of the event`
    `EXTRACT(YEAR FROM e.end_date) AS year,  -- Extract the year from the end date`
    `ou.organizer                       -- Organizer of the event`

  `-- From the enhanced events table`
  `FROM proj-tmc-mem-mvp.everyaction_enhanced.enh_everyaction__events AS e`

  `-- Left join with event signups table based on the eventsignupid`
  `LEFT JOIN proj-tmc-mem-mvp.everyaction_cleaned.cln_everyaction__eventsignups AS es`
    `ON e.eventsignupid = es.eventsignupid`

  `-- Left join with contacts table based on vanid
  `LEFT JOIN proj-tmc-mem-mvp.everyaction_enhanced.enh_everyaction__contacts AS ec`
    `ON e.vanid = ec.vanid`

  `-- Left join with the organizing user events table a connected sheet I made based on event name`
  `LEFT JOIN prod-organize-arizon-4e1c0a83.organizing_view.organizing_user_events AS ou`
    `ON e.event_name = ou.event_name`

  `-- Filter the results based on specific conditions`
  `WHERE e.start_date >= '2025-01-01'  -- Include events starting from January 1, 2025`
    `AND e.status = 'Completed'        -- Only include event attendees with a 'Completed' status`

`);`
```

**organizinggoals02182025_run2nd** -- creates ptg meters
_______________________________________________________________
``` SQL
CREATE OR REPLACE VIEW `prod-organize-arizon-4e1c0a83.organizing_view.organizing_goals02182025` AS (
  WITH a AS (
    SELECT
       organizer_user
      ,EXTRACT(MONTH FROM utc_datecanvassed) AS month
      ,COUNT(contacttypename) AS phone_attempts
    FROM `prod-organize-arizon-4e1c0a83.organizing_view.contacts_011325`
    WHERE contacttypename = 'Phone'
      AND utc_datecanvassed >= '2025-01-01 00:00:00 UTC'
    GROUP BY organizer_user, month
  ),
  b AS (
    SELECT
       organizer_user
      ,EXTRACT(MONTH FROM utc_datecanvassed) AS month
      ,COUNT(contacttypename) AS phone_conversations
    FROM `prod-organize-arizon-4e1c0a83.organizing_view.contacts_011325`
    WHERE contacttypename = 'Phone'
      AND utc_datecanvassed >= '2025-01-01 00:00:00 UTC'
      AND resultshortname = 'Canvassed'
    GROUP BY organizer_user, month
  ),
  c AS (
    SELECT
       organizer_user
      ,EXTRACT(MONTH FROM utc_datecanvassed) AS month
      ,COUNT(contacttypename) AS one_on_ones
    FROM `prod-organize-arizon-4e1c0a83.organizing_view.contacts_011325`
    WHERE contacttypename = 'One on One'
      AND utc_datecanvassed >= '2025-01-01 00:00:00 UTC'
      AND resultshortname = 'Canvassed'
    GROUP BY organizer_user, month
  ),
  d AS (
    SELECT
       organizer
      ,EXTRACT(MONTH FROM start_date) AS month
      ,COUNT(vanid) AS volunteer_shifts
    FROM `prod-organize-arizon-4e1c0a83.organizing_view.volunteershifts02182025`
    WHERE role = 'Volunteer'
    GROUP BY organizer, month
  ),
  e AS (
    SELECT
       organizer
      ,EXTRACT(MONTH FROM start_date) AS month
      ,COUNT(DISTINCT vanid) AS volunteers
    FROM `prod-organize-arizon-4e1c0a83.organizing_view.volunteershifts02182025`
    WHERE role = 'Volunteer'
    GROUP BY organizer, month
  )
  SELECT DISTINCT
       a.organizer_user
      ,a.month
      ,a.phone_attempts
      ,b.phone_conversations
      ,c.one_on_ones
      ,d.volunteer_shifts
      ,e.volunteers
  FROM a
  LEFT JOIN b ON a.organizer_user = b.organizer_user AND a.month = b.month
  LEFT JOIN c ON a.organizer_user = c.organizer_user AND a.month = c.month
  LEFT JOIN d ON a.organizer_user = d.organizer AND a.month = d.month
  LEFT JOIN e ON a.organizer_user = e.organizer AND a.month = e.month
  ORDER BY a.organizer_user, a.month
)
```

**ea_hotlist_2025** -- creates tables for hotlists engagement
____________________________________________
```SQL
`-- Create or replace the view for EA Hotlists in 2025`
`CREATE OR REPLACE VIEW prod-organize-arizon-4e1c0a83.organizing_view.ea_hotlists_2025 AS (`

  `-- Subquery to count the number of phone conversations for each VAN ID`
  `WITH a AS (`
    `SELECT` 
      `vanid,                             -- The VAN ID (unique identifier for a contact)`
      `COUNT(contactscontactid) AS phone_conversations  -- Count the number of phone conversations`
    `FROM proj-tmc-mem-mvp.everyaction_cleaned.cln_everyaction__contactscontacts`
    `WHERE contacttypename = 'Phone'        -- Filter for phone contacts`
      `AND resultshortname = 'Canvassed'    -- Filter for 'Canvassed' result`
      `AND utc_datecanvassed >= '2025-01-01 00:00:00 UTC'  -- Only include records from 2025 onward`
    `GROUP BY vanid                         -- Group by VAN ID (each contact)`
    `ORDER BY phone_conversations DESC      -- Order by number of phone conversations in descending order`
  `),`

  `-- Subquery to count the number of events attended for each VAN ID`
  `b AS (`
    `SELECT` 
      `vanid,                             -- The VAN ID (unique identifier for a contact)`
      `COUNT(contactscontactid) AS events_attended  -- Count the number of events attended`
    `FROM proj-tmc-mem-mvp.everyaction_cleaned.cln_everyaction__contactscontacts`
    `WHERE contacttypename = 'Event'       -- Filter for event contacts`
      `AND resultshortname = 'Canvassed'    -- Filter for 'Canvassed' result`
      `AND utc_datecanvassed >= '2025-01-01 00:00:00 UTC'  -- Only include records from 2025 onward`
    `GROUP BY vanid                         -- Group by VAN ID (each contact)`
    `ORDER BY events_attended DESC          -- Order by number of events attended in descending order`
  `),`

  `-- Subquery to count the number of one-on-one meetings for each VAN ID`
  `c AS (`
    `SELECT` 
      `vanid,                             -- The VAN ID (unique identifier for a contact)`
      `COUNT(contactscontactid) AS one_on_ones  -- Count the number of one-on-one meetings`
    `FROM proj-tmc-mem-mvp.everyaction_cleaned.cln_everyaction__contactscontacts`
    `WHERE contacttypename = 'One on One'  -- Filter for one-on-one contacts`
      `AND resultshortname = 'Canvassed'   -- Filter for 'Canvassed' result`
      `AND utc_datecanvassed >= '2025-01-01 00:00:00 UTC'  -- Only include records from 2025 onward`
    `GROUP BY vanid                         -- Group by VAN ID (each contact)`
    `ORDER BY one_on_ones DESC              -- Order by number of one-on-ones in descending order`
  `),`

  `-- Subquery to count the number of volunteer shifts for each VAN ID`
  `d AS (`
    `SELECT DISTINCT`
      `vanid,                             -- The VAN ID (unique identifier for a contact)`
      `COUNT(role) AS volunteer_shifts    -- Count the number of volunteer shifts`
    `FROM prod-organize-arizon-4e1c0a83.organizing_view.volunteershifts02182025`
    `WHERE role = 'Volunteer'             -- Filter for volunteer roles`
    `GROUP BY vanid                        -- Group by VAN ID (each contact)`
  `)`

  `-- Final select combining all subqueries and additional information`
  `SELECT DISTINCT` 
    `cc.vanid,                            -- The VAN ID from contacts table`
    `ec.first_name,                       -- The first name from contacts table`
    `ec.last_name,                        -- The last name from contacts table`
    `sr.surveyresponsename,               -- Survey response name (e.g., 'Yes', 'No')`
    `a.phone_conversations,               -- Number of phone conversations from subquery a`
    `b.events_attended,                   -- Number of events attended from subquery b`
    `c.one_on_ones,                       -- Number of one-on-one meetings from subquery c`
    `d.volunteer_shifts                   -- Number of volunteer shifts from subquery d`
  `FROM proj-tmc-mem-mvp.everyaction_cleaned.cln_everyaction__contactscontacts AS cc`

  `-- Left join with survey responses to get survey data`
  `LEFT JOIN proj-tmc-mem-mvp.everyaction_enhanced.enh_everyaction__survey_responses AS sr`
    `ON cc.contactscontactid = sr.contactscontactid`

  `-- Left join with contacts to get first and last name for each VAN ID`
  `LEFT JOIN proj-tmc-mem-mvp.everyaction_enhanced.enh_everyaction__contacts AS ec`
    `ON cc.vanid = ec.vanid`

  `-- Left joins with the subqueries (a, b, c, d) to get the various counts`
  `LEFT JOIN a ON cc.vanid = a.vanid`
  `LEFT JOIN b ON cc.vanid = b.vanid`
  `LEFT JOIN c ON cc.vanid = c.vanid`
  `LEFT JOIN d ON cc.vanid = d.vanid`

  `-- Filtering the final dataset to include only relevant survey responses`
  `WHERE` 
    `sr.surveyquestionname = 'Recruiting Organizer'  -- Only include responses for 'Recruiting Organizer'`
    `AND sr.surveyresponsename NOT IN ('No Organizer', 'Remove')  -- Exclude 'No Organizer' and 'Remove' responses`

`);`

```

Organizing view contacts
```SQL
create or replace view `prod-organize-arizon-4e1c0a83.organizing_view.contacts_011325` as (

  

select

cc.vanid,

cc.contacttypename,

cc.resultshortname,

cc.utc_datecanvassed,

extract(year from cc.utc_datecanvassed) as year,

cc.canvassedby,

u.organizer_user

  
  

from `proj-tmc-mem-mvp.everyaction_cleaned.cln_everyaction__contactscontacts` as cc

  

left join `proj-tmc-mem-mvp.everyaction_enhanced.enh_everyaction__contacts` as ec

on cc.vanid = ec.vanid

  

left join `prod-organize-arizon-4e1c0a83.organizing_view.ea_publicuser_organizers` as u

on cc.canvassedby = u.canvassedby

  

where cc.contacttypename <> 'No Actual Contact'

)
```


Event tables + totals
Query name: organizing_event_attendees
```SQL
CREATE OR REPLACE VIEW `prod-organize-arizon-4e1c0a83.organizing_view.distinctevents011025` AS (
  WITH event_attendees AS (
    SELECT DISTINCT
    e.event_name||e.start_date as event_id
    , e.role
    , COUNT(e.role) as role_count
    FROM `proj-tmc-mem-mvp.everyaction_enhanced.enh_everyaction__events` AS e
    GROUP BY 1,2
    ORDER BY 1
  )
  SELECT DISTINCT
    
    ou.event_name
    , ou.start_date
    , ou.organizer
    , CASE
        WHEN ou.organizer = 'Elyanna Juarez' THEN 'Pinal'
        WHEN ou.organizer = 'Hector Castellanos' THEN 'Pinal'
        WHEN ou.organizer = 'Tara Clayton' THEN 'Cochise'
        WHEN ou.organizer = 'Jhanitzel Bogarin' THEN 'Yuma'
        WHEN ou.organizer IS NULL THEN 'Virtual'
        ELSE NULL
      END AS event_location
    , ae.role
    , ae.role_count

  FROM `prod-organize-arizon-4e1c0a83.organizing_view.organizing_user_events` AS ou

  LEFT JOIN event_attendees AS ae
    on ou.event_name||ou.start_date = ae.event_id

  WHERE ou.start_date > '2024-12-31'
)
```

Creates a table of event attendees, phone, event_role, events attended, count of events, and affiliated organizers
```SQL
CREATE OR REPLACE VIEW `prod-organize-arizon-4e1c0a83.organizing_view.unique_attendee_volunteers` AS (
  WITH main_query AS (
    WITH unique_attendees AS (
      SELECT DISTINCT
        vanid
        , 'event_attendee' as role
      FROM `proj-tmc-mem-mvp.everyaction_enhanced.enh_everyaction__events` 
      WHERE start_date > '2024-12-31'
    )


    SELECT 
      c.first_name||" "||c.last_name AS full_name
      , c.phone_number
      , e.vanid
      ,a.role
      , STRING_AGG(CAST(e.event_name AS STRING),', ') as agg_events
      , ARRAY_LENGTH(SPLIT(STRING_AGG(CAST(e.event_name AS STRING), ', '), ', ')) AS event_count

    FROM `proj-tmc-mem-mvp.everyaction_enhanced.enh_everyaction__events`  AS e

    LEFT JOIN unique_attendees AS a
      ON e.vanid = a.vanid

    LEFT JOIN `proj-tmc-mem-mvp.everyaction_enhanced.enh_everyaction__contacts` AS c
      ON e.vanid = c.vanid

    WHERE e.start_date > '2024-12-31'

    GROUP BY 1, 2, 3, 4
  )

  SELECT 
    m.full_name
    , m.phone_number
    , m.vanid
    , m.role
    , m.agg_events
    , m.event_count
    , (
        SELECT STRING_AGG(DISTINCT oe.organizer, ', ')
        FROM `prod-organize-arizon-4e1c0a83.organizing_view.organizing_user_events` AS oe
        WHERE oe.event_name IN UNNEST(SPLIT(m.agg_events, ', '))
      ) AS agg_organizers
  FROM main_query AS m
)
```

Added month for date filters to the two event queries above

unique_attendee_volunteers
```sql
CREATE OR REPLACE VIEW `prod-organize-arizon-4e1c0a83.organizing_view.unique_attendee_volunteers` AS (
  WITH main_query AS (
    WITH unique_attendees AS (
      SELECT DISTINCT
        vanid
        , 'event_attendee' as role
      FROM `proj-tmc-mem-mvp.everyaction_enhanced.enh_everyaction__events` 
      WHERE start_date > '2024-12-31'
    )


    SELECT 
      c.first_name||" "||c.last_name AS full_name
      , c.phone_number
      , e.vanid
      , a.role
      , EXTRACT(MONTH FROM e.start_date) AS month
      , STRING_AGG(CAST(e.event_name AS STRING),', ') as agg_events
      , ARRAY_LENGTH(SPLIT(STRING_AGG(CAST(e.event_name AS STRING), ', '), ', ')) AS event_count

    FROM `proj-tmc-mem-mvp.everyaction_enhanced.enh_everyaction__events`  AS e

    LEFT JOIN unique_attendees AS a
      ON e.vanid = a.vanid

    LEFT JOIN `proj-tmc-mem-mvp.everyaction_enhanced.enh_everyaction__contacts` AS c
      ON e.vanid = c.vanid

    WHERE e.start_date > '2024-12-31'

    GROUP BY 1, 2, 3, 4, 5
  )

  SELECT 
    m.month
    , m.full_name
    , m.phone_number
    , m.vanid
    , m.role
    , m.agg_events
    , m.event_count
    , (
        SELECT STRING_AGG(DISTINCT oe.organizer, ', ')
        FROM `prod-organize-arizon-4e1c0a83.organizing_view.organizing_user_events` AS oe
        WHERE oe.event_name IN UNNEST(SPLIT(m.agg_events, ', '))
      ) AS agg_organizers
  FROM main_query AS m
)

```

organizing event attendees
```sql
CREATE OR REPLACE VIEW `prod-organize-arizon-4e1c0a83.organizing_view.organizing_event_attendees` AS (
  WITH event_attendees AS (
    SELECT DISTINCT
    e.event_name||e.start_date as event_id
    , e.role
    , COUNT(e.role) as role_count
    FROM `proj-tmc-mem-mvp.everyaction_enhanced.enh_everyaction__events` AS e
    GROUP BY 1,2
    ORDER BY 1
  )
  SELECT DISTINCT
    
    ou.event_name
    , ou.start_date
    , EXTRACT(MONTH FROM ou.start_date) AS month
    , ou.organizer
    , CASE
        WHEN ou.organizer = 'Elyanna Juarez' THEN 'Pinal'
        WHEN ou.organizer = 'Hector Castellanos' THEN 'Pinal'
        WHEN ou.organizer = 'Tara Clayton' THEN 'Cochise'
        WHEN ou.organizer = 'Jhanitzel Bogarin' THEN 'Yuma'
        WHEN ou.organizer IS NULL THEN 'Virtual'
        ELSE NULL
      END AS event_location
    , ae.role
    , ae.role_count

  FROM `prod-organize-arizon-4e1c0a83.organizing_view.organizing_user_events` AS ou

  LEFT JOIN event_attendees AS ae
    on ou.event_name||ou.start_date = ae.event_id

  WHERE ou.start_date > '2024-12-31'
)
```


hotlist code - using activist codes in EA
```SQL
WITH hotlists AS(
  SELECT 
    c.firstname
    , c.lastname
    , a.activistcodename
  FROM `proj-tmc-mem-mvp.everyaction_enhanced.enh_everyaction__activist_codes` AS a

  LEFT JOIN `proj-tmc-mem-mvp.everyaction_cleaned.cln_everyaction__contacts` AS c
    ON a.vanid = c.vanid

  WHERE activistcodename LIKE '%Hotlist'
)

, role_counts AS (
  SELECT
    vanid,
    role,
    COUNT(*) AS role_count
  FROM `proj-tmc-mem-mvp.everyaction_enhanced.enh_everyaction__events` 
  WHERE start_date > '2024-12-31'
    AND final_status = 'Completed'
  GROUP BY vanid, role
)
SELECT

FROM role_counts AS r

LEFT JOIN hotlist AS h
```