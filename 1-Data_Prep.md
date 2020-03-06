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
- File 1 - Student Data : https://ctgraduates.lightning.force.com/lightning/r/Report/00O1M000007QxDwUAK/view
- File 2 - Scholarship Data:  https://ctgraduates.lightning.force.com/lightning/r/Report/00O1M000007QxHKUA0/view
- File 3 - Test Data:  https://ctgraduates.lightning.force.com/lightning/r/Report/00O1M000007R0eMUAS/view
- File 4 - Academic Term Data: https://ctgraduates.lightning.force.com/lightning/r/Report/00O1M000007R0JJUA0/view?queryScope=userFolders

### Changes
- 02-06-2020 : Started project

```python
# General Setup
# ALWAYS RUN
%load_ext dotenv
%dotenv
%load_ext nb_black


from salesforce_reporting import Connection, ReportParser
import pandas as pd
from pathlib import Path
from datetime import datetime
import helpers
import os
import re
import numpy as np


SF_PASS = os.environ.get("SF_PASS")
SF_TOKEN = os.environ.get("SF_TOKEN")
SF_USERNAME = os.environ.get("SF_USERNAME")

sf = Connection(username=SF_USERNAME, password=SF_PASS, security_token=SF_TOKEN)
```

### File Locations

```python
# ALWAYS RUN
today = datetime.today()


in_file1 = Path.cwd() / "data" / "raw" / "sf_output_file1.csv"
summary_file = Path.cwd() / "data" / "processed" / "processed_data.pkl"


in_file2 = Path.cwd() / "data" / "raw" / "sf_output_file2.csv"
summary_file2 = Path.cwd() / "data" / "processed" / "processed_data_file2.pkl"


in_file3 = Path.cwd() / "data" / "raw" / "sf_output_file3.csv"
summary_file3 = Path.cwd() / "data" / "processed" / "processed_data_file3.pkl"


in_file4 = Path.cwd() / "data" / "raw" / "sf_output_file4.csv"
summary_file4 = Path.cwd() / "data" / "processed" / "processed_data_file4.pkl"
```

### Load Report From Salesforce

```python
# File 1
report_id_file1 = "00O1M000007QxDwUAK"
sf_df = helpers.load_report(report_id_file1, sf)


# File 2
report_id_file2 = "00O1M000007QxHKUA0"
sf_df_file2 = helpers.load_report(report_id_file2, sf)


# File 3
report_id_file3 = "00O1M000007R0eMUAS"
sf_df_file3 = helpers.load_report(report_id_file3, sf)

# File 4
report_id_file4 = "00O1M000007R0JJUA0"
sf_df_file4 = helpers.load_report(report_id_file4, sf)


```

```python
# len(sf_df_file4)
```

#### Save report as CSV

```python
# File 1
sf_df.to_csv(in_file1, index=False)

# File 2
sf_df_file2.to_csv(in_file2, index=False)

# File 3
sf_df_file3.to_csv(in_file3, index=False)

# File 4
sf_df_file4.to_csv(in_file4, index=False)
```

### Load DF from saved CSV
* Start here if CSV already exist 

```python
# ALWAYS RUN
# Data Frame for File 1 - if using more than 1 df, the variable 'df' will refer to file 1
df = pd.read_csv(in_file1)

# Data Frames for File 2
df_file2 = pd.read_csv(in_file2)

# File 3
df_file3 = pd.read_csv(in_file3)

# File 4
df_file4 = pd.read_csv(in_file4)
```

### Data Manipulation


#### Functions for file cleaning

```python
def create_gpa_bucket(gpa):
    if gpa < 2.5:
        return "2.5 or less"
    elif gpa < 2.75:
        return "2.5 - 2.74"
    elif gpa < 3.0:
        return "2.75 - 2.9"
    elif gpa < 3.5:
        return "3.0 - 3.49"
    elif gpa >= 3.5:
        return "3.5 or greater"
    else:
        return "NULL"
```

```python
def clean_column_names(df):
    
    df.columns = (
        df.columns.str
        .strip().str.lower()
        .str.replace(" ", "_")
        .str.replace("(", "")
        .str.replace(")", "")
        .str.replace("-", "_")
        .str.replace(":", "")
        .str.replace("<", "less_")
        .str.replace("=", "")
        .str.replace(".", "")
        
    )
    
    return df

```

#### Cleaning applied to all files

```python
# File 1
df = helpers.shorten_site_names(df)
df = clean_column_names(df)

# File 2
df_file2 = helpers.shorten_site_names(df_file2)
df_file2 = clean_column_names(df_file2)


# File 3
df_file3 = helpers.shorten_site_names(df_file3)
df_file3 = clean_column_names(df_file3)

# File 4
df_file4 = helpers.shorten_site_names(df_file4)
df_file4 = clean_column_names(df_file4)
```

#### File 1 Specific

```python
df["gpa_bucket"] = df.apply(
    lambda x: create_gpa_bucket(x.college_elig_gpa_11th_cgpa), axis=1
)
```

#### Merging Data into File 1

```python
entrance_scores =df_file3[
    (df_file3.site == "San Francisco")
    & (df_file3.version == "Entrance into CT Diagnostic")
]

```

```python
entrace_scores = df_file3[df_file3.version == "Entrance into CT Diagnostic"]
```

```python
entrance_merge = entrace_scores[
    [
        "18_digit_id",
        "act_english",
        "act_mathematics",
        "act_english_readiness",
        "act_math_readiness",
    ]
]
```

```python
df = df.merge(entrance_merge, on="18_digit_id", how="left")
```

```python
students_with_dup_ats = df_file4[
    df_file4.duplicated(subset=["18_digit_id", "global_academic_term"], keep=False)
]
```

```python
students_with_dup_ats.sort_values(["18_digit_id", "global_academic_term"]).to_csv(
    "duplicate_at.csv"
)
```

```python
df_file_4_spring = df_file4[df_file4.global_academic_term.str.match("Spring*")]
```

```python
len(df_file_4_spring)
```

```python
df_file_4_spring = df_file_4_spring.drop_duplicates(
    subset=["18_digit_id", "global_academic_term"]
)
```

```python
grade_pivot = df_file_4_spring.pivot(
    index="18_digit_id", columns="grade_at", values="gpa_running_cumulative",
)
```

```python
df = df.merge(grade_pivot, on="18_digit_id", how="left")
```

```python
df
```

### Save output file into processed directory

Save a file in the processed directory that is cleaned properly. It will be read in and used later for further analysis.

```python
# Save File 1 Data Frame (Or master df)
df.to_pickle(summary_file)

df_file2.to_pickle(summary_file2)

df_file3.to_pickle(summary_file3)
```
