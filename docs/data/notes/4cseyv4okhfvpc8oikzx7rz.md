Star date 04/14/2025
- Currently Rich cannot use the giant merged file I gave him because it (predictably) has way too many columns to open anywhere. Going to make a merge script that gives me all of my column names in an output csv so I can work with all 500+ of them manually in a google sheet where I can see best which ones I wanna delete. Then, I'll reconvert that to a column list and write script to keep only the columns in that list.
  
###### merge + column list script
```python
import pandas as pd
import csv

df1 = pd.read_csv('filtered_mapfiles.csv')
df2 = pd.read_csv('cleaned_results_2024_modified_precincts.csv')


pctnum = 'pctnum'

merged_df = pd.merge(df1, df2, on=pctnum, how='outer')

#convert any numeric values from strings for qgis
merged_df = merged_df.apply(lambda col: pd.to_numeric(col, errors='ignore'))

# Save the final merged DataFrame to a new CSV file
merged_df.to_csv('merged_mapfiles_raza.csv', index=False)

column_list = merged_df.columns.tolist()

# Comment out all of this to just merge the files
# Open a new CSV file for writing
with open("output.csv", "w", newline="") as csvfile:
    writer = csv.writer(csvfile)
    
    # Write header (optional)
    writer.writerow(["Column Name"])
    
    # Write each item as a new row (each in one column)
    for item in column_list:
        writer.writerow([item])
        
print("CSV file created successfully as output.csv")
```

[Here is the column list google sheet](https://docs.google.com/spreadsheets/d/1sj8dgKm0C6LsUs7qqTR18htDsBVoEko9CNIw8uo4vX8/edit?gid=1954200069#gid=1954200069)
- There are 565 columns, kill me

Here is the shortened list
- I still have 336 columns, that's too many probably but let's try it
![[merge file column list - filtered_columns 1.csv]]

Now I have to write a script that will use this list to keep only these columns in my merged file