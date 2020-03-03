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

This notebook contains basic statistical analysis and visualization of the data.

### Data Sources
- summary : Processed file from notebook 1-Data_Prep

### Changes
- 02-06-2020 : Started project

```python
import pandas as pd
from pathlib import Path
from datetime import datetime
```

```python
%matplotlib inline
%load_ext nb_black
```

### File Locations

```python
today = datetime.today()
in_file = Path.cwd() / "data" / "processed" / "processed_data.pkl"
report_dir = Path.cwd() / "reports"
report_file = report_dir / "Excel_Analysis_{today:%b-%d-%Y}.xlsx"

in_file2 = Path.cwd() / "data" / "processed" / "processed_data_file2.pkl"

in_file3 = Path.cwd() / "data" / "processed" / "processed_data_file3.pkl"

```

```python
df = pd.read_pickle(in_file)

df2 = pd.read_pickle(in_file2)

df3 = pd.read_pickle(in_file3)
```

### Perform Data Analysis

```python
# len(df3)
```

####  General Distrobutions

```python
df.columns
```

```python
def sf_cross_tab(df, column, normalize="index"):
    return pd.crosstab(
        df[df.site == "San Francisco"].high_school_class,
        df[df.site == "San Francisco"][column],
        normalize=normalize,
    )
```

```python
# Grad Rate Less than 6 years
sf_cross_tab(df, "graduated_4_year_degree_less_6_years")

```

```python
# Overall grad rate
sf_cross_tab(df, "graduated_4_year_degree")
```

```python
# low income
sf_cross_tab(df, "indicator_low_income")
```

```python
# GPA Bucket
sf_cross_tab(df, "gpa_bucket")
```

```python
# Ethnic Breakdown
sf_cross_tab(df, "ethnic_background")
```

```python
sf_cross_tab(df, "indicator_first_generation")
```

```python
sf_cross_tab(df, "readiness_composite_official")
```

```python
sf_cross_tab(df, "cc_advisor_4_year_degree_earned")
```

```python
sf_cross_tab(df, "act_superscore_highest_official")
```

```python
sf_cross_tab(df, "college_first_enrolled_school_type")
```

```python
sf_cross_tab(df, "#_of_best_fit_college_applications")
```

```python
sf_cross_tab(df, "#_entrance_into_ct_diagnostic")
```

```python
sf_cross_tab(df, "#_four_year_college_acceptances", normalize=False)
```

```python
grade_order = ["9th Grade", "10th Grade", "11th Grade", "12th Grade"]
```

```python
pd.crosstab(
    [
        df3[df3.site == "San Francisco"].high_school_class,
        df3[df3.site == "San Francisco"].hs_grade_level,
    ],
    df3[df3.site == "San Francisco"]["act_math_readiness"],
    normalize="index",
).reindex(index=grade_order, level=1)
```

```python
entrace_scores = df3[df3.version == "Entrance into CT Diagnostic"]
```

```python
sf_cross_tab(entrace_scores, "act_math_readiness")
```

```python
sf_cross_tab(entrace_scores, "act_english_readiness")
```

### Save Excel file into reports directory

Save an Excel file with intermediate results into the report directory

```python
writer = pd.ExcelWriter(report_file, engine='xlsxwriter')
```

```python
df.to_excel(writer, sheet_name='Report')
```

```python
writer.save()
```
