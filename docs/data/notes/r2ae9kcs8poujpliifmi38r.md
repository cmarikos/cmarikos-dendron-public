```SQL
SELECT

geo_id

, total_pop

-- childcare

, families_with_young_children

, two_parent_families_with_young_children

, two_parents_in_labor_force_families_with_young_children

, children_in_single_female_hh

, children

, family_households

, father_in_labor_force_one_parent_families_with_young_children

, father_one_parent_families_with_young_children

--immigration

, not_us_citizen_pop

, hispanic_pop

FROM `cta-open-data.american_community_survey_linked.county_2018_5yr`

  

WHERE geo_id LIKE '04%'
```


AZ Census geoids: https://www2.census.gov/geo/docs/reference/codes2020/cousub/st04_az_cousub2020.txt - 04 is AZ

|   |   |   |   |   |   |   |
|---|---|---|---|---|---|---|
|vb_vf_county_name|total_pop|avg_path_to_citizen_score|var_pop_citizenship|high_citizen_support|noncitizen_perc|imm_org_need_index|
|APACHE|71522|56.95243235|1092.996344|0.5645192528|1.14%|351.6986277|
|COCHISE|126279|41.94651234|1113.14099|0.3516878333|5.00%|3664.336141|
|COCONINO|140217|59.09985199|1039.819835|0.5765856719|2.88%|1652.774981|
|GILA|53400|39.88383589|1117.371606|0.3207362028|2.12%|680.5149777|
|GRAHAM|37879|36.24576903|1001.656093|0.2630832003|2.49%|601.2023981|
|GREENLEE|9504|42.09191248|1008.016239|0.3317054112|2.14%|117.5534177|
|LA PAZ|20701|32.3061239|852.7077026|0.2157981037|7.56%|1059.409161|
|MARICOPA|4253913|53.15327255|1063.766042|0.4728759484|8.92%|177694.3219|
|MOHAVE|206064|30.65740617|743.3187532|0.1712256395|3.48%|4973.25083|
|NAVAJO|108705|43.43187683|1151.7704|0.3813553805|1.46%|894.9077085|
|PIMA|1019722|58.32537684|1113.457835|0.5607348095|6.74%|28640.46802|
|PINAL|419721|42.83636275|1020.267491|0.341984311|5.27%|12639.45183|
|SANTA CRUZ|46584|55.92384903|853.4601478|0.4983134288|13.98%|2870.679713|
|YAVAPAI|224645|37.51556935|1007.476831|0.2721998775|3.04%|4271.435679|
|YUMA|207829|48.23908232|989.658217|0.4084477176|14.95%|16079.52908|
### Immigration Organizing Need Index

```
imm_org_need_index = total_pop * noncitizen_perc * (1 - (avg_path_to_citizen_score / 100))
```

- avg_path_to_citizen_score / 100: This normalizes the support score to a 0–1 range.
- 1 - (avg_path_to_citizen_score / 100): This flips the measure so that a lower support score results in a larger value, highlighting counties that may need more organizing efforts.
- total_pop * noncitizen_perc: Multiplying by the total_pop and the noncitizen_perc emphasizes areas with larger, at-risk populations.

This adapted formula gives you a composite index where counties with a high non-citizen population and low support for the path to citizenship yield a higher score, helping you prioritize organizing efforts