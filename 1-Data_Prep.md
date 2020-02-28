---
jupyter:
  jupytext:
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.2'
      jupytext_version: 1.3.0
  kernelspec:
    display_name: Python 3
    language: python
    name: python3
---

## sf_class_2014

A project to determine what, if anything influenced the graduation success of the San Francisco class of 2014.

### Data Sources
- file1 : https://ctgraduates.lightning.force.com/lightning/r/Report/00O1M000007QxDwUAK/view
- file2:  https://ctgraduates.lightning.force.com/lightning/r/Report/00O1M000007QxHKUA0/view
- file3:  Link to SF Report (As Needed)

### Changes
- 02-06-2020 : Started project

```python
# General Setup
%load_ext dotenv
%dotenv
%load_ext nb_black


from salesforce_reporting import Connection, ReportParser
import pandas as pd
from pathlib import Path
from datetime import datetime
import helpers
import os

SF_PASS = os.environ.get("SF_PASS")
SF_TOKEN = os.environ.get("SF_TOKEN")
SF_USERNAME = os.environ.get("SF_USERNAME")

sf = Connection(username=SF_USERNAME, password=SF_PASS, security_token=SF_TOKEN)
```

### File Locations

```python
today = datetime.today()
in_file1 = Path.cwd() / "data" / "raw" / "sf_output_file1.csv"
in_file2 = Path.cwd() / "data" / "raw" / "sf_output_file3.csv"
# in_file3 = Path.cwd() / "data" / "raw" / "sf_output_file3.csv"

summary_file = Path.cwd() / "data" / "processed" / "processed_data.pkl"

summary_file2 = Path.cwd() / "data" / "processed" / "processed_data_file2.pkl"
```

### Load Report From Salesforce

```python
# File 1
report_id_file1 = "00O1M000007QxDwUAK"
sf_df = helpers.load_report(report_id_file1, sf)

# File 2 and 3 (As needed)
report_id_file2 = "00O1M000007QxHKUA0"
# report_id_file3 = "SF_REPORT_ID"
sf_df_file2 = helpers.load_report(report_id_file2, sf)
# sf_df = helpers.load_report(report_id_file3, sf)
```

```python
len(sf_df), len(sf_df_file2)
```

#### Save report as CSV

```python
# File 1
sf_df.to_csv(in_file1, index=False)


# File 2 and 3 (As needed)
sf_df_file2.to_csv(in_file2, index=False)
# sf_df.to_csv(in_file3, index=False)

```

### Load DF from saved CSV
* Start here if CSV already exist 

```python
# Data Frame for File 1 - if using more than one file, rename df to df_file1
df = pd.read_csv(in_file1)


# Data Frames for File 1 and 2 (As needed)

df_file2 = pd.read_csv(in_file2)
# df_file3 = pd.read_csv(in_file3)
```

### Data Manipulation

```python
def clean_column_names(df):
    
    df.columns = (
        df.columns.str
        .strip().str.lower()
        .str.replace(" ", "_")
        .str.replace("(", "")
        .str.replace(")", "")
    )
    
    return df

```

```python
df = helpers.shorten_site_names(df)
df_file2 = helpers.shorten_site_names(df_file2)
df = clean_column_names(df)
df_file2 = clean_column_names(df_file2)
```

### Save output file into processed directory

Save a file in the processed directory that is cleaned properly. It will be read in and used later for further analysis.

```python
# Save File 1 Data Frame (Or master df)
df.to_pickle(summary_file)
df_file2.to_pickle(summary_file2)
```
