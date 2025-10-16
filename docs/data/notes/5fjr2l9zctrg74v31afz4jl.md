#education

#### RAZA Advocates Dashboard
https://lookerstudio.google.com/s/tlnRNoAMiGE 


###### RAZA Advocates Event Attendees
```SQL
CREATE OR REPLACE VIEW `prod-organize-arizon-4e1c0a83.viewers_dataset.RAZA_Advocates_attendee_volunteers` AS (
  SELECT *
  FROM (
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
    FROM main_query AS m)
  WHERE agg_organizers like '%RAZA Advocates%'
```



###### RAZA Advocate Applicants
(may need to remove applicants that never joined the program)
```SQL
CREATE OR REPLACE VIEW `prod-organize-arizon-4e1c0a83.viewers_dataset.RAZA_Advo_Apps` AS(
  SELECT DISTINCT
    cf.vanid
    , f.utc_datecreated
    , c.first_name||' '|| c.last_name AS full_name
    , a.activistcodename
  FROM `proj-tmc-mem-mvp.everyaction_cleaned.cln_everyaction__contactsonlineforms` AS cf

  LEFT JOIN `proj-tmc-mem-mvp.everyaction_cleaned.cln_everyaction__onlineforms` AS f
    ON cf.onlineformid = f.onlineformid

  LEFT JOIN `proj-tmc-mem-mvp.everyaction_cleaned.cln_everyaction__contactsactivistcodes` AS ca
    ON cf.vanid = ca.vanid

  LEFT JOIN `proj-tmc-mem-mvp.everyaction_cleaned.cln_everyaction__activistcodes` AS a
    ON ca.activistcodeid = a.activistcodeid

  LEFT JOIN `proj-tmc-mem-mvp.everyaction_enhanced.enh_everyaction__contacts` AS c
    ON cf.vanid = c.vanid

  WHERE f.onlineformname = 'RAZA Advocates Application'
    AND a.activistcodename IN ('Yuma', 'Pinal', 'Coconino')
)
```

######RAZA Advocate Event Attendee Types
```SQL
CREATE OR REPLACE VIEW `prod-organize-arizon-4e1c0a83.viewers_dataset.RAZA_Advos_Event_Attendee_Types` AS (
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
        WHEN ou.organizer = 'RAZA Advocates Yuma' THEN 'Yuma'
        WHEN ou.organizer = 'RAZA Advocates Pinal' THEN 'Pinal'
        WHEN ou.organizer = 'RAZA Advocates Coconino' THEN 'Coconino'
        ELSE NULL
      END AS event_location
    , ae.role
    , ae.role_count

  FROM `prod-organize-arizon-4e1c0a83.organizing_view.organizing_user_events` AS ou

  LEFT JOIN event_attendees AS ae
    on ou.event_name||ou.start_date = ae.event_id

  WHERE ou.start_date > '2024-12-31'
  AND ou.program LIKE '%Education%'
)
```

###### RAZA Advos Activism
```SQL
CREATE OR REPLACE VIEW `prod-organize-arizon-4e1c0a83.viewers_dataset.RAZA_Advo_Activism` AS(
WITH event_roles AS(
     SELECT DISTINCT
    e.vanid
    , COUNT(e.role) as role_count
    FROM `proj-tmc-mem-mvp.everyaction_enhanced.enh_everyaction__events` AS e
    WHERE e.role = 'Volunteer'
    GROUP BY 1
    ORDER BY 1
)

SELECT DISTINCT
ra.*
, COALESCE(er.role_count, 0) as volunteer_shifts
, COALESCE(rv.event_count, 0) as events_attended

FROM `prod-organize-arizon-4e1c0a83.viewers_dataset.RAZA_Advo_Apps` AS ra

LEFT JOIN `prod-organize-arizon-4e1c0a83.viewers_dataset.RAZA_Advocates_attendee_volunteers` AS rv
  ON ra.vanid = rv.vanid

LEFT JOIN event_roles AS er
  ON ra.vanid = er.vanid
)
```