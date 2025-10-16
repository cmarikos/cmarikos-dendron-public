```SQL
SELECT DISTINCT
v.dwid
, COALESCE(vp.likely_cell_phone, vp.likely_land_phone) AS phone_number
, INITCAP(v.firstname) AS firstname
, INITCAP(v.lastname) AS lastname
, INITCAP(v.regdeliveryaddrline) AS address
, INITCAP(v.regaddrcity) AS city
, v.regaddrstate AS state
, v.mailaddrzip AS zip
FROM`proj-tmc-mem-mvp.catalist_cleaned.cln_catalist__person` AS v

LEFT JOIN `proj-tmc-mem-mvp.catalist_cleaned.cln_catalist__phones` AS vp
  ON v.DWID = vp.dwid

LEFT JOIN `proj-tmc-mem-mvp.catalist_cleaned.cln_catalist__district` AS d
  ON v.DWID = d.dwid

LEFT JOIN `proj-tmc-mem-mvp.catalist_cleaned.cln_catalist__models` AS m
  ON d.dwid = m.dwid

LEFT JOIN `proj-tmc-mem-mvp.catalist_cleaned.cln_catalist__vote_history` AS vh
  ON d.dwid = vh.dwid

WHERE COALESCE(vp.likely_cell_phone, vp.likely_land_phone) IS NOT NULL
  AND v.regaddrstate = 'AZ'
  AND INITCAP(d.countyname) = 'Pinal'
  -- voted in last two primaries
  AND vh.e2024pvm IS NOT NULL
  AND vh.e2023pvm IS NOT NULL
  -- no CD7 for pinal there are only 5 voters who fit this category in CD7
  AND v.partyaffiliation = 'DEM

```