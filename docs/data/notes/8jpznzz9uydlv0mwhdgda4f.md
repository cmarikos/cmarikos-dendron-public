<iframe width="600" height="450" src="https://lookerstudio.google.com/embed/reporting/2c90014e-a1a2-4f7e-a06c-2729e118b2e8/page/p_gl68xct6pd" frameborder="0" style="border:0" allowfullscreen sandbox="allow-storage-access-by-user-activation allow-scripts allow-same-origin allow-popups allow-popups-to-escape-sandbox"></iframe>



Survey responses by county + precinct (pctnum)
```SQL
SELECT
	mc.countyname
	, mc.pctnum
	, sq.SurveyQuestionName
	, sr.SurveyResponseName
	, COUNT(cs.VanID) as response_count 

FROM `prod-organize-arizon-4e1c0a83.ngpvan.CTARAA_ContactsSurveyResponses_VF` AS cs 

LEFT JOIN `prod-organize-arizon-4e1c0a83.ngpvan.CTARAA_SurveyQuestions` AS sq
	ON cs.SurveyQuestionID = sq.SurveyQuestionID

LEFT JOIN `prod-organize-arizon-4e1c0a83.ngpvan.CTARAA_SurveyResponses` AS sr
	ON cs.SurveyResponseID = sr.SurveyResponseID

FULL OUTER JOIN `prod-organize-arizon-4e1c0a83.rich_christina_proj.modified_c4_precincts_2024` as mc
	ON cs.VanID = mc.VanID

WHERE sq.Cycle ='2024'
	AND mc.countyname in ('COCONINO','COCHISE','MOHAVE','PINAL','PIMA','YAVAPAI','YUMA')

GROUP BY 1,2,3,4
ORDER BY 1,2,3,4
```



create a view and join to geo table
```SQL
CREATE OR REPLACE VIEW `prod-organize-arizon-4e1c0a83.rich_christina_proj.sr_by_pctnum_c4_2024` AS (

	WITH a AS(
		SELECT
			mc.pctnum
			, sq.SurveyQuestionName
			, sr.SurveyResponseName
			, COUNT(cs.VanID) as response_count
		
		FROM `prod-organize-arizon-4e1c0a83.ngpvan.CTARAA_ContactsSurveyResponses_VF` AS cs
		
		LEFT JOIN `prod-organize-arizon-4e1c0a83.ngpvan.CTARAA_SurveyQuestions` AS sq
			ON cs.SurveyQuestionID = sq.SurveyQuestionID 
		
		LEFT JOIN `prod-organize-arizon-4e1c0a83.ngpvan.CTARAA_SurveyResponses` AS sr
			ON cs.SurveyResponseID = sr.SurveyResponseID
		
		FULL OUTER JOIN `prod-organize-arizon-4e1c0a83.rich_christina_proj.modified_c4_precincts_2024` as mc
			ON cs.VanID = mc.VanID 
		
		WHERE sq.Cycle ='2024'
			AND mc.countyname IN ('COCONINO','COCHISE','MOHAVE','PINAL','PIMA','YAVAPAI','YUMA')
		
		GROUP BY 1,2,3
		ORDER BY 1,2,3
	)  
	
	SELECT
		pg.GEOMETRY
		, pg.COUNTY
		, pg.PRECINCTNA
	    , pg.PCTNUM
		, pg.LEGISLATIV
		, pg.CONGRESSIO
		, a.SurveyQuestionName
		, a.SurveyResponseName
		, a.response_count
	
	FROM `prod-organize-arizon-4e1c0a83.geofiles.az_precincts_geo` AS pg
	
	LEFT JOIN a
		ON pg.PCTNUM = a.pctnum
	  
	WHERE a.response_count IS NOT NULL
)
```

