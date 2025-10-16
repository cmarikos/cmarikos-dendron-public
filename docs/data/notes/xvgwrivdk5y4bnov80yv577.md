I want to be able to do geospatial analysis to compare CVAP data to the targetsmart voterfile

This is my starting point 
```sql
SELECT
	pn.pctnum
	, vb.vb_tsmart_place
	, COUNT(DISTINCT vb.voterbase_id) as tsmart_voters
-- this is the targetsmart voterfile
FROM `prod-organize-arizon-4e1c0a83.targetsmart_AZ.voter_base_latest` AS vb

-- this is a crosswalk I made to convert the census geoid for place to match the vb_tsmart_place codes
LEFT JOIN `prod-organize-arizon-4e1c0a83.rich_christina_proj.place_crosswalk` AS pl
	ON vb.vb_tsmart_place = pl.place_id

-- this is the cvap table with place
LEFT JOIN `prod-organize-arizon-4e1c0a83.rich_christina_proj.cvap_2019_4yr_place` AS cv
	ON cv.geoid = pl.geoid

-- this is a crosswalk I made to covert AZ precinct codes into a less annoying format so I can have consistency across VAN/voterfile types
LEFT JOIN `prod-organize-arizon-4e1c0a83.rich_christina_proj.targetsmart_pctnum_crosswalk` AS pn
	ON vb.vb_tsmart_precinct_name = pn.vb_vf_precinct_name

WHERE
	vb.vb_tsmart_state = 'AZ'

GROUP BY 1, 2
```

With this I immediately notice that place boundaries cross precinct boundaries, like a lot

Next, I'm gonna do geospatial analysis with SQL. To do this I need a census place shapefile found [here](https://www.census.gov/cgi-bin/geo/shapefiles/index.php?year=2024&layergroup=Places), and converted with the process detailed [here](obsidian://open?vault=Work&file=Projects%2FMaps%20in%20Looker%20-%20File%20Conversion%20Guide)

```sql
WITH spatial_alloc AS (
	SELECT
		pg.PCTNUM,
		ARRAY_AGG(STRUCT(
			cg.GEOID,
			ST_AREA(ST_INTERSECTION(pg.GEOMETRY, cg.GEOMETRY)) AS intersect_area,
			ST_AREA(ST_INTERSECTION(pg.GEOMETRY, cg.GEOMETRY)) / ST_AREA(pg.GEOMETRY) AS overlap_fraction
		)) AS overlaps
		
	FROM `prod-organize-arizon-4e1c0a83.geofiles.az_precincts_geo` AS pg
	
	JOIN `prod-organize-arizon-4e1c0a83.geofiles.census_place_2024_geo` AS cg
		ON ST_INTERSECTS(pg.GEOMETRY, cg.GEOMETRY)
	GROUP BY pg.PCTNUM
),  

allocated_census AS (
	SELECT
		sa.PCTNUM,
		SUM(COALESCE(cv.cvap_est, 0) * ov.overlap_fraction) AS allocated_census_population
		
	FROM spatial_alloc AS sa
	
	CROSS JOIN UNNEST(sa.overlaps) AS ov
	
	-- this was a pain in the butt to figure out, casting in the join works
	LEFT JOIN `prod-organize-arizon-4e1c0a83.rich_christina_proj.place_crosswalk` AS cw
		ON CAST(ov.GEOID AS INT64) = CAST(cw.place_id AS INT64)
	
	LEFT JOIN `prod-organize-arizon-4e1c0a83.rich_christina_proj.cvap_2019_4yr_place` AS cv
		ON cw.geoid = cv.geoid
	GROUP BY 1
),

voter_counts AS (
	SELECT
		pn.pctnum,
		COUNT(DISTINCT vb.voterbase_id) AS registered_voters
	
	FROM `prod-organize-arizon-4e1c0a83.rich_christina_proj.targetsmart_pctnum_crosswalk` AS pn
	
	LEFT JOIN `prod-organize-arizon-4e1c0a83.targetsmart_AZ.voter_base_latest` AS vb
		ON pn.vb_vf_precinct_name = vb.vb_vf_precinct_name
	
	GROUP BY pn.pctnum
)

SELECT
	vc.pctnum,
	vc.registered_voters,
	ac.allocated_census_population

FROM voter_counts AS vc

LEFT JOIN allocated_census AS ac
	ON vc.pctnum = ac.PCTNUM;
```

The current issue with this is that I assumed that census place geometry areas were always smaller in area to AZ precincts, they vary wildly and this is often not the case. We can't just make place geometry the denominator of our overlap_fraction on line 7, because it's also true that census place geometries are often smaller than precinct geometries. I'm ripping my hair our on how to move forward, and I'm hungry, so this is where I stop for the evening.

Re-evaluating my choices here on day two. I think there are two possible solutions:
1. This is not a problem SQL is suited for, and I should do it in python then re-import to BigQuery (or maybe try out the BQ python notebooks?)
2. I should either forget about precincts entirely and go by block group, or try to find a geographic boundary smaller than all of the precincts in AZ. Maybe block? I'll try this one first because god forbid I learn something new when I don't have to. My issue with using place is that my TigerLine shapefile may or may not be good. I think it has way too many points for looker to manage even if I could subdivide by county (it totally doesn't have a county field to divide by ): )
3. Miriam from Columbia/DPI wants to meet, but I have to have something more substantial to show her before I go crying for help
4. Just also got a call back from the census bureau-- who despite the layoffs and political bullshit have time to answer voicemails from citizens like me because they are worth their weight in gold-- with clarification that there are no precinct based CVAP results in the ACS. Precinct/voting district based results are only collected each decennial census and are stored in the category of redistricting. Miriam also suggested checking the AZ redistricting website, so that's a thread to pull. That said, the last census was a whole presidential administration ago, it's hard to make arguments about getting funding for a program based on data that will not represent the last four cycles of program work.
	1. The nice census man also pointed out something that I overlooked. I'm only getting some of the place boundaries in my shapefile because I'm using 2024, not 2023 (the final year of my CVAP data). Not every place is surveyed every year. Gotta upload the 2023 place shapefile.


One more Hail Mary on the precinct spatial analysis though, because I just found something interesting. When I join the cvap data with the tl shapefile (the unformatted one before I hacked off all the fields I thought I didn't need in geo-formatting it), I see intplat-intplong pairs on the end that I could try to use to drop the place data into specific precincts?
```sql
SELECT
p.*,
cg.*

FROM `prod-organize-arizon-4e1c0a83.rich_christina_proj.cvap_2019_4yr_place` AS p

LEFT JOIN `prod-organize-arizon-4e1c0a83.rich_christina_proj.place_crosswalk` AS cw
	ON p.geoid = cw.geoid

INNER JOIN `prod-organize-arizon-4e1c0a83.geofiles.census_place_2024` AS cg
	ON CAST(cw.place_id as INT64) = cg.GEOID
```


Next steps:
1. Upload the 2023 shapefile, 
2. Geoformat _without_ chopping off a bunch of fields 
3. Try doing a spatial join again and see if it looks less insane

Also just gonna pop some info here for comparison:
[This breaks down national average of CVAP population vs voter registration](https://docs.google.com/spreadsheets/d/1PvZONQhsjHb8cqjhZi1DTg9RRlVR7wB7tUknlGxD3S4/edit?usp=sharing), so basically anything around 50-60% would make me unsuspicious, too far below, and especially too far above would be a sign that I'm fuckin up.

Back to GDAL with my shiny new 2023 shapefile.
```zsh
geoenvcmarikos@MacBookAir geoenv % ogr2ogr -f "CSV" census_place_2023.csv "tl_2023_04_place.shp" -lco GEOMETRY=AS_WKT
```

Now I'm uploading to BigQuery again, but this time I'm not dropping all my "unnecessary" columns like a dummy. Also gonna fix the 2024 file in case I need it again.
```sql
CREATE OR REPLACE VIEW `prod-organize-arizon-4e1c0a83.geofiles.census_place_2023_geo` AS(
	SELECT
		ST_GEOGFROMTEXT(WKT) AS GEOMETRY
		,*
	
	FROM `prod-organize-arizon-4e1c0a83.geofiles.census_place_2023`
)
```

Looking over the place map for 2023, I should really just do this analysis by place, it would be silly to do anything else. I can left join in my registered voters by place to only get places surveyed.

Hahaha! I have a new problem. Kill me.
```sql
WITH registered_voters AS(
	SELECT
		vb_tsmart_place as geoid
		,COUNT(DISTINCT voterbase_id) AS registered_voters
	
	FROM `prod-organize-arizon-4e1c0a83.targetsmart_AZ.voter_base_latest`
	
	GROUP BY 1
)

  

SELECT
	p.*
	, rv.registered_voters
	, cv.cvap_est 

FROM `prod-organize-arizon-4e1c0a83.geofiles.census_place_2023_geo` AS p

LEFT JOIN registered_voters AS rv
	ON p.GEOID = CAST(rv.geoid AS INT64)
 
-- dear reader, please know that this is a 5yr ACS report, and I just can't count, so I named it incorrectly
LEFT JOIN `prod-organize-arizon-4e1c0a83.rich_christina_proj.cvap_2019_4yr_place` as cv
	ON p.GEOIDFQ = cv.geoid
```

Dumb mistake, I forgot that there are different categories in the CVAP data, I was getting multiple rows with duplicate place results because I have demographic breakdown CVAP columns.

```sql
WITH registered_voters AS(
	SELECT DISTINCT
		vb_tsmart_place as geoid
		,COUNT(DISTINCT voterbase_id) AS registered_voters
	
	FROM `prod-organize-arizon-4e1c0a83.targetsmart_AZ.voter_base_latest`
	
	GROUP BY 1
)

SELECT
	p.*
	, rv.registered_voters
	, cv.cvap_est
	, cv.cvap_moe

FROM `prod-organize-arizon-4e1c0a83.geofiles.census_place_2023_geo` AS p  

-- switched these to inner joins to get no nulls, idk dude
INNER JOIN registered_voters AS rv
	ON p.GEOID = CAST(rv.geoid AS INT64)

INNER JOIN `prod-organize-arizon-4e1c0a83.rich_christina_proj.cvap_2019_4yr_place` as cv
	ON p.GEOIDFQ = cv.geoid 

-- cvap totals row only
WHERE cv.lnnumber = 1
```

This seems correct now, the CVAP counts seem close enough with consistency that I'm not suspicious, I am suspicious though because all my place are showing at or near 100% voter registration rates when compared with the place data even if the MoE is included in the CVAP totals. Ugh. This is 2023, is it possible we had that big of an increase in population + voter registration to make this make sense? Maybe? 

Gonna leave this one alone for a bit and return when I'm less annoyed. Might be time to bother Miriam. I also messaged Hal, who gave me the CVAP data, they said they know why my counts are off and are gonna send me something.

Jk, not dead yet on this. Gonna see if I can limit to people registered in or before 2023. 

I did a little text here to see if I can narrow my registration date at all since the date format is funky. Gonna pop the where here back into the registered voters CTE above so I can hopefully only count the people who were registered before the beginning of 2024
```sql
SELECT
	MAX(vb_vf_registration_date)
FROM `prod-organize-arizon-4e1c0a83.targetsmart_AZ.voter_base_latest`
WHERE vb_vf_registration_date < 20240000
```


My registered_voters still look way high compared to the cvap_est, but at least most of them aren't above 100% now? Idk. Now I think I actually have to wait for Hal and/or Miriam.

Spoke with Hal yesterday. Looks like the voterfile probably has tons of inactive voters in the roll still. Going to try removing them to get a closer count.

Just ran this to check how big of an impact this might have. It returned a count of 641,481. That seems like enough.
```sql
SELECT
	COUNT(vb_voterbase_registration_status)

FROM `prod-organize-arizon-4e1c0a83.targetsmart_AZ.voter_base_latest`

WHERE vb_voterbase_registration_status = "Unregistered"
```

I added status and a deceased flag back into my registered_voters CTE
```sql
WITH registered_voters AS(
	SELECT DISTINCT
		vb_tsmart_place as geoid
		,COUNT(DISTINCT voterbase_id) AS registered_voters 
	
	FROM `prod-organize-arizon-4e1c0a83.targetsmart_AZ.voter_base_latest`
	
	WHERE vb_vf_registration_date < 20240000 -- date before 2024
		AND vb_deceased_flag_date_of_death is null -- not deceased
		AND vb_voterbase_registration_status <> "Unregistered"
	
	GROUP BY 1
)
```

My results are lower than they have been, but they still seem too high. Gonna take the basic difference between the registered voters and cvap column so I can have something to look at on a map. I'm also going to remove the before 2024 line. I want to know what happened in 2024 and beyond.

**saved as cvap_place**
```sql
CREATE OR REPLACE VIEW `prod-organize-arizon-4e1c0a83.rich_christina_proj.az_unregistered_est` AS (
	WITH registered_voters AS(
		SELECT DISTINCT
			vb_tsmart_place as geoid
			,COUNT(DISTINCT voterbase_id) AS registered_voters
		
		FROM `prod-organize-arizon-4e1c0a83.targetsmart_AZ.voter_base_latest`
		WHERE vb_vf_registration_date < 20240000 -- date before 2024
			AND vb_deceased_flag_date_of_death is null -- not deceased
			AND vb_voterbase_registration_status <> "Unregistered"
		
		GROUP BY 1
	)
	
	SELECT
	p.*
	, rv.registered_voters
	, cv.cvap_est
	, cv.cvap_moe
	, CASE
		WHEN (1 - (rv.registered_voters / (cv.cvap_est + cv.cvap_moe))) < 0
			THEN NULL
		ELSE (1 - (rv.registered_voters / (cv.cvap_est + cv.cvap_moe)))
	END AS unregistered_percent
	
	FROM `prod-organize-arizon-4e1c0a83.geofiles.census_place_2023_geo` AS p  
	
	LEFT JOIN registered_voters AS rv
		ON p.GEOID = CAST(rv.geoid AS INT64)
	
	INNER JOIN `prod-organize-arizon-4e1c0a83.rich_christina_proj.cvap_2019_4yr_place` as cv
		ON p.GEOIDFQ = cv.geoid
	
	-- cvap totals row only
	WHERE cv.lnnumber = 1
)
```

The map honestly seems reasonable so I'm going to move forward with demographic breakdowns.

##### Email from Chris Brill at Targetsmart
1. Our modeled race categorical variables do not map neatly onto the CVAP race categories from the census. While our models are built using a certain amount of census data, they are not built in order to try to re-create census race categories. 
    
2. Because of that- we really don't have any categorical race variables that I would utilize to try to find 'multi-racial' individuals.  
    
3. Our most accurate race model is actually going to be the 'vb.tsmr_race' variable.  If you are able to access vb.voterbase_race than I think you can get to the vb.tsmr_race variable.  
    
4. When I work on projects comparing CVAP race data to our voter file- I tend to restrict that analysis by comparing the CVAP 'alone' categories to the tsmr_race categories- while realizing that I am excluding some percentage of the population.  
    
5. However.. There are also tsmr_race_score variables that one might try to utilize for something like this- these are the raw scores that feed into the tsmr_race categorical variable, so in theory, you could identify people who, for instance, have a higher model score for both white and asian.  
    
6. The rise of the 'multi-racial' category though is very real- but I don't think we've gotten to a place yet where we are attempting to systematically model that at the individual level.

Here is a sheet with statewide registration + CVAP by county: https://docs.google.com/spreadsheets/d/14jQsHXcgrbYyz7fyB20qacZ3kWTtbiHu9uJQairxK-Y/edit?gid=1976786004#gid=1976786004
- Current reg % is 61.77%, gonna change my map to display % registered rather than % unregistered
  
Updated the case statement to reflect the change above
```SQL
, CASE 
  WHEN (rv.registered_voters / (cv.cvap_est + cv.cvap_moe)) < 0 
    THEN NULL 
  ELSE (rv.registered_voters / (cv.cvap_est + cv.cvap_moe))
END AS unregistered_percent
```