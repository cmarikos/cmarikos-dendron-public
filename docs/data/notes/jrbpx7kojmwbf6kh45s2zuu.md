I have the RAZA IDs in long format and need to convert to wide format for QGIS, so I can get one row per precinct.

```python
import pandas as pd

# Read the CSV
df = pd.read_csv('Candidate_Issue_IDS - Extract 1 (1).csv') 

# Pivot the data so that each precinct becomes one row and IDs become columns
wide_df = df.pivot_table(
    index=['GEOMETRY', 'COUNTY', 'PRECINCTNA','precinctname', 'PCTNUM', 'LEGISLATIV', 'CONGRESSIO'],
    columns=['SurveyQuestionName', 'SurveyResponseName'],
    values='response_count',
    aggfunc='sum'
).reset_index()

# Optionally, flatten the MultiIndex columns if desired
wide_df.columns = ['_'.join([str(c) for c in col if c]).strip('_') for col in wide_df.columns.values]

# Save the wide format to a new CSV file
wide_df.to_csv('wide_ids2.csv', index=False)

```