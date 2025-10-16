[MCF Application Portal](https://mcf.smartsimple.com/s_Login.jsp)
[Application Working Doc](https://docs.google.com/document/d/18VJaj5Ll-3EA4VQEQn_LAFxpXEcYkJkpht1Agn_JaSg/edit?tab=t.0)

[My working sheet](https://docs.google.com/spreadsheets/d/1kt9CvbyI0YigTP9MIpjf6ntn-7a1qyhNgR2k2cCTO0o/edit?usp=sharing)
### Data Components

#### Staff Metrics
%%
Data source: employment data
%%

##### Executive Leadership Demographic Information
Executive leadership is defined here as Executive Director(s) or CEO(s) only.

- % African American or Black: 0
- % American Indian or Alaska Native: 0
- % Asian or Asian American (including Indian Subcontinent): 0 
- % Pacific Islander or Native Hawaiian: 0
- % Hispanic or Latinx: 2
- % White: 1
- % Multiracial or Other: 0
- % Middle Eastern or North African: 0

##### Staff Demographic Information

- % African American or Black: 1
- % American Indian or Alaska Native: 1
- % Asian or Asian American (including Indian Subcontinent): 1
- % Pacific Islander or Native Hawaiian: 1
- % Hispanic or Latinx: 19
- % White 8
- % Multiracial or Other: 10
- % Middle Eastern or North African: 0
  
  
  

#### Constituency Demographic Information

> Going to be using the 2018 5 year ACS data for this and including all populations in our target counties. This is the most recent ACS that surveys AZ.
> 
> Getting my brain back into census data mode. [This breakdown of FIPS/ANSI codes and ACS geoids was helpful](https://www.census.gov/library/reference/code-lists/ansi.html#cou). 
>
>	Arizona's state ID is 04
>		[Here is the county breakdown](https://www2.census.gov/geo/docs/reference/codes2020/cousub/st04_az_cousub2020.txt)
>

```SQL
  CASE
      WHEN geo_id = '04003' THEN 'Cochise'
      WHEN geo_id = '04005' THEN 'Coconino'
      WHEN geo_id = '04015' THEN 'Mohave'
      WHEN geo_id = '04019' THEN 'Pima'
      WHEN geo_id = '04021' THEN 'Pinal'
      WHEN geo_id = '04023' THEN 'Santa Cruz'
      WHEN geo_id = '04025' THEN 'Yavapai'
      WHEN geo_id = '04027' THEN 'Yuma'
    ELSE NULL
  END AS county
```




Please tell us about the demographics of your organization's target constituency.

Constituency is defined here as the *individuals who directly benefit from the work of your organization*.


Note: 
I am defining our constituency as the entire population of the counties we’re active in. Partly because that data is easily accessible and clean. With the time we have to submit this using the broad, accessible data makes the most sense to me

###### Urbanicity

[Using this U of A Census analysis](https://crh.arizona.edu/sites/default/files/2023-06/2300601_Census-RuralUpdate-Brief.pdf) for our target counties, I got the total population of our target counties, then got a percentage of rural pop. I omitted Pima county since we don't do work in the city and it really skews the numbers.
- %Rural: 27%
- % Suburban:
- % Urban:  73%

###### Race/Ethnicity
- % African American or Black:
- % American Indian or Alaska Native:
- % Asian or Asian American (including Indian Subcontinent):
- % Pacific Islander or Native Hawaiian: - no data/possibly split between "Other", "American Indian",  and "Asian"
- % Hispanic or Latinx:
- % White:
- % Multiracial or Other:
- % Middle Eastern or North African: - no data


```SQL
SELECT 
  CASE
      WHEN geo_id = '04003' THEN 'Cochise'
      WHEN geo_id = '04005' THEN 'Coconino'
      WHEN geo_id = '04015' THEN 'Mohave'
      WHEN geo_id = '04019' THEN 'Pima'
      WHEN geo_id = '04021' THEN 'Pinal'
      WHEN geo_id = '04023' THEN 'Santa Cruz'
      WHEN geo_id = '04025' THEN 'Yavapai'
      WHEN geo_id = '04027' THEN 'Yuma'
    ELSE NULL
  END AS county
  , black_pop
  , amerindian_pop
  , asian_pop
  , hispanic_pop
  , white_pop
  , other_race_pop
FROM `cta-open-data.american_community_survey_linked.county_2018_5yr`
WHERE 
  geo_id IN ('04003','04005','04015','04019','04021','04023','04025','04027')
```


|            | black_percent | amerindian_percent | asian_percent | hispanic_percent | white_percent | other_race_percent |
| ---------- | ------------- | ------------------ | ------------- | ---------------- | ------------- | ------------------ |
| Total      | 2.72%         | 3.82%              | 2.04%         | 34.38%           | 56.93%        | 0.11%              |

###### Additional Demographics
- % Low-income: 19.27%
	- Using the [HUD income guidelines](https://www.huduser.gov/portal/datasets/home-datasets/files/HOME_IncomeLmts_State_AZ_2024.pdf) I'm selecting low income pop as those making less than 57.6k a year, which is the highest threshold for county based "low income" benefits
```SQL
SELECT 
  CASE
      WHEN geo_id = '04003' THEN 'Cochise'
      WHEN geo_id = '04005' THEN 'Coconino'
      WHEN geo_id = '04015' THEN 'Mohave'
      WHEN geo_id = '04019' THEN 'Pima'
      WHEN geo_id = '04021' THEN 'Pinal'
      WHEN geo_id = '04023' THEN 'Santa Cruz'
      WHEN geo_id = '04025' THEN 'Yavapai'
      WHEN geo_id = '04027' THEN 'Yuma'
    ELSE NULL
  END AS county
 , total_pop
 , income_10000_14999
 , income_15000_19999
 , income_20000_24999
 , income_25000_29999
 , income_30000_34999
 , income_35000_39999
 , income_40000_44999
 , income_45000_49999
 , income_50000_59999

FROM `cta-open-data.american_community_survey_linked.county_2018_5yr`
WHERE 
  geo_id IN ('04003','04005','04015','04019','04021','04023','04025','04027')


```

- % Elder/senior (65 years and older): 20.30%
	- Census doesn't have an over 65
```SQL
SELECT 
	  CASE
	      WHEN geo_id = '04003' THEN 'Cochise'
	      WHEN geo_id = '04005' THEN 'Coconino'
	      WHEN geo_id = '04015' THEN 'Mohave'
	      WHEN geo_id = '04019' THEN 'Pima'
	      WHEN geo_id = '04021' THEN 'Pinal'
	      WHEN geo_id = '04023' THEN 'Santa Cruz'
	      WHEN geo_id = '04025' THEN 'Yavapai'
	      WHEN geo_id = '04027' THEN 'Yuma'
	    ELSE NULL
	  END AS county
	 , total_pop
	 , female_65_to_66
	 , female_67_to_69
	 , female_70_to_74
	 , female_75_to_79
	 , female_80_to_84
	 , female_85_and_over
	 , male_65_to_66
	 , male_67_to_69
	 , male_70_to_74
	 , male_75_to_79
	 , male_80_to_84
	 , male_85_and_over


FROM `cta-open-data.american_community_survey_linked.county_2018_5yr`
WHERE 
  geo_id IN ('04003','04005','04015','04019','04021','04023','04025','04027')


```
- % Youth (16-24 years): 13.03%

Subtract the pop_16_over from pop_25_over to get 16-25 (not gonna fuss that this includes 25 year olds, not worth the headache)

```SQL
SELECT 
  CASE
      WHEN geo_id = '04003' THEN 'Cochise'
      WHEN geo_id = '04005' THEN 'Coconino'
      WHEN geo_id = '04015' THEN 'Mohave'
      WHEN geo_id = '04019' THEN 'Pima'
      WHEN geo_id = '04021' THEN 'Pinal'
      WHEN geo_id = '04023' THEN 'Santa Cruz'
      WHEN geo_id = '04025' THEN 'Yavapai'
      WHEN geo_id = '04027' THEN 'Yuma'
    ELSE NULL
  END AS county
 , total_pop
 , pop_16_over
 , pop_25_years_over
 , pop_25_64

FROM `cta-open-data.american_community_survey_linked.county_2018_5yr`
WHERE 
  geo_id IN ('04003','04005','04015','04019','04021','04023','04025','04027')

```
- % LGBTQ+: 4.5%
	- Using the statewide estimate from the [UCLA Williams Institute AZ Profile](https://williamsinstitute.law.ucla.edu/visualization/lgbt-stats/?topic=LGBT&area=4#about-the-data) and assuming counties are *roughly* similar because there doesn't seem to be good county data
--- the following are bad ideas:
	- gonna use modeled scores from the voter file since any "DEI" census data has been deleted. This will not include unregistered ppl, so idk if its good
	- Statewide pop is ~4.5% so if it's way off from that I'll ask follow up questions
- % Immigrant/refugee: 36.34%
	- The [U of A MAP](https://mapazdashboard.arizona.edu/article/bucking-trend-arizonas-share-foreign-born-falls#:~:text=Santa%20Cruz%20County%20reported%20the,the%20lowest%20share%20at%201.6%25.) page links to census data which I downloaded. Turns out the downloaded data is kinda junk. It's only able to give me percentages by county. 
	- Gonna use the [AZ pop estimate from the 2020 Census](https://data.census.gov/profile/Arizona?g=040XX00US04) then convert the percentages by county to a number that I can then divide by total pop
--- getting "freaky" also a bad idea:
	- For this we gotta get freaky too and use a combo of the voterfile and the census data. Gonna grab the immigrant registrants from the voterfile, then add the noncitizen pop to that -- might look at avging the two files idk


