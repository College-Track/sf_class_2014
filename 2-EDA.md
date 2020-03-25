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

## OP Learning Agenda: SF Class of 2014

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
import statsmodels.api as sm
import numpy as np
```

```python
%matplotlib inline

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

```python
# original df file only included students who completed hs program. later removed that filter,
#so changing df to be the original subset, and df_master to be the whole group

df_master = df

df = df[df.indicator_completed_ct_hs_program == True]
```

```python

```

```python
sf_cross_tab(df, "original_incoming_cohort?")
```

### Perform Data Analysis


####  General Distrobutions

SF Class of 2014 is on track to have the highest 6 year grad rate, with almost 70% of students already graduating, but that number isn't significantly higher than the class of 2013. Though we do see a reasonably big jump from 2012 to 2013. 

The first table represents the percentages, while the second is the raw numbers.

```python
def sf_cross_tab(df, column, normalize="index"):
    return pd.crosstab(
        df[df.site == "San Francisco"].high_school_class,
        df[df.site == "San Francisco"][column],
        normalize=normalize,
        margins=True,
    )
```

```python
# Grad Rate Less than 6 years
sf_cross_tab(df, "graduated_4_year_degree_less_6_years")

```

```python
sf_cross_tab(df, "graduated_4_year_degree_less_6_years", normalize=False)

```

We can also look at the 5 year grad rate, which is more accurate as all the data is turned in for this.

However, it tells almost the same story.

```python
sf_cross_tab(df, "graduated_4_year_degree_less_5_years")

```

If we run independent t-test on the grad rates, we see that in fact comparing 2014 to 2013, and even to 2012, results in statistically insignificant differences. 

The values displayed are the p values for first the 2014 -> 2013 comparison, and then  2014 -> 2012.

```python
population1 = (
    df[(df.site == "San Francisco") & (df.high_school_class == 2013)][
        "graduated_4_year_degree_less_5_years"
    ]
).values


population2 = (
    df[(df.site == "San Francisco") & (df.high_school_class == 2014)][
        "graduated_4_year_degree_less_5_years"
    ]
).values
```

```python
# p value of independent t-test on populations above
sm.stats.ttest_ind(population1, population2)[1]

```

```python
population1 = (
    df[(df.site == "San Francisco") & (df.high_school_class == 2012)][
        "graduated_4_year_degree_less_5_years"
    ]
).values


population2 = (
    df[(df.site == "San Francisco") & (df.high_school_class == 2014)][
        "graduated_4_year_degree_less_5_years"
    ]
).values
```

```python
# p value of independent t-test on populations above
sm.stats.ttest_ind(population1, population2)[1]

```

```python
df_sub = df[df.site == "San Francisco"]
```

```python
y = df_sub["graduated_4_year_degree_less_6_years"]
```

```python
X = df_sub["college_elig_gpa_11th_cgpa"]
X = sm.add_constant(X)
```

```python
import matplotlib.pyplot as plt

# Note the difference in argument order
model = sm.OLS(y.astype(float), X.astype(float)).fit()
predictions = model.predict(X)  # make the predictions by the model

# Print out the statistics
model.summary()

# fig, ax = plt.subplots()
# fig = sm.graphics.plot_fit(model, 0, ax=ax)

```

```python
# table_1.iloc[3]["All"]
```

```python
# n1 = table_1.iloc[0]['All']
# p1 = table_2.iloc[0][True]

# n2 = table_1.iloc[3]["All"]
# p2 = table_2.iloc[3][True]

# population1 = np.random.binomial(1, p1, n1)
# population2 = np.random.binomial(1, p2, n2)

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

### SF Compared to Other Sites

Looking at the class of 2013 compared to other sites

```python
def class_cross_tab(df, column, hs_class, normalize="index"):
    return pd.crosstab(
        df[df.high_school_class == hs_class].site,
        df[df.high_school_class == hs_class][column],
        normalize=normalize,
        margins=True,
    )
```

```python
class_cross_tab(df, "graduated_4_year_degree_less_6_years", hs_class=2013)

```

```python
class_cross_tab(df, "graduated_4_year_degree_less_6_years", hs_class=2012)

```

```python
class_cross_tab(df, "graduated_4_year_degree_less_6_years", hs_class=2011)

```

```python
df.columns
```

```python
pd.crosstab(
        df.received_site_based_advising,
        df.graduated_4_year_degree_less_4_years,
        normalize='index',
        margins=True,
    )
```

```python
# comparing if the class of 2013 / 2014 had a higher percent of students who didn't complete hs program

sf_cross_tab(df_master, "indicator_completed_ct_hs_program", normalize=False)

```

```python
sf_cross_tab(df_master, "indicator_completed_ct_hs_program")
```

```python
sf_cross_tab(df, "school")['Lowell High School']

```

```python
df_lowel = df[df.school == 'Lowell High School']
```

```python
sf_cross_tab(df_lowel, "graduated_4_year_degree_less_6_years", normalize=False)

```

```python
sf_cross_tab(df, "graduated_4_year_degree_less_6_years", normalize=False)

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
