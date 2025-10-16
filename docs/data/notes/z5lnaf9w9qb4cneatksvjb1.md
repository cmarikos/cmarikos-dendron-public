AZ Precinct names suck, here's a script to fix it

```python
import pandas as pd

  

precincts = pd.read_csv('/RAZA_c4_2024_Door_Attempts - bq-results-20250107-174015-1736271644789.csv')

  

# County to prefix dictionary, may consider swapping with FIPS

county_codes = {

'YUMA':'YU'

,'MARICOPA':'MC'

,'SANTA CRUZ':'SC'

,'GILA':'GI'

,'PIMA':'PM'

,'PINAL':'PN'

,'APACHE':'AP'

,'GRAHAM':'GM'

,'LA PAZ':'LP'

,'MOHAVE':'MO'

,'NAVAJO':'NA'

,'COCHISE':'CH'

,'YAVAPAI':'YA'

,'COCONINO':'CN'

,'GREENLEE':'GN'

}

  

# Function to extract pctnum based on county and precinct code

def extract_pctnum(countyname, precinctcode, county_codes):

  

try:

precinct_int = int(float(precinctcode)) # Handles cases like 25.0 → 25

except ValueError:

return 'ERROR'

  

# Ensure precinctcode is a string and zero-pad to 4 digits

num_part = str(precinct_int).zfill(4)

  

# Get the county code from the dictionary

county_codes = county_codes.get(countyname)

  

if county_codes:

return county_codes + num_part # Concatenate county code and zero-padded precinct code

else:

return 'ERROR'

  
  

#Apply to dataframe, swap out row['__'] with column names

precincts['pctnum'] = precincts.apply(lambda row: extract_pctnum(row['countyname'], row['precinctcode'], county_codes), axis=1)

  

#Export the new dataframe to csv

precincts.to_csv('/content/modified_c4precincts.csv', index=False)

  

#Display head to check result

precincts.head()
```


merge files based on pctnum
```python
import pandas as pd

df1 = pd.read_csv('Map Files - everything but 2024.csv')
df2 = pd.read_csv('results_2024_modified_precincts.csv')


pctnum = 'pctnum'

merged_df = pd.merge(df1, df2, on=pctnum, how='outer')

#convert any numeric values from strings for qgis
merged_df = merged_df.apply(lambda col: pd.to_numeric(col, errors='ignore'))

# Replace '%' with 'perc' in all column names
merged_df.columns = merged_df.columns.str.replace('%', 'perc')

# Save the final merged DataFrame to a new CSV file
merged_df.to_csv('merged_mapfiles_raza.csv', index=False)

print(merged_df.head())

```