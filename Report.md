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

A project to determine what, if anything, influenced the graduation success of the San Francisco class of 2014.

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
def sf_cross_tab(df, column, normalize="index"):
    return pd.crosstab(
        df[df.site == "San Francisco"].high_school_class,
        df[df.site == "San Francisco"][column],
        normalize=normalize,
        margins=True,
    )
```

##  General Distrobutions

SF Class of 2014 is on track to have the highest 6 year grad rate, with almost 70% of students already graduating, but that number isn't significantly higher than the class of 2013. Though we do see a reasonably big jump from 2012 to 2013. 




#### Table 1. San Francisco 6 Year Graduation Rate by High School Class 

```python
# Grad Rate Less than 6 years
sf_cross_tab(df, "graduated_4_year_degree_less_6_years")

```

#### Table 2. San Francisco 6 Year Graduation Count by High School Class 


```python
sf_cross_tab(df, "graduated_4_year_degree_less_6_years", normalize=False)

```

#### Table 3. San Francisco 5 Year Graduation Rate by High School Class

To be more accurate, we can look at the 5 year grad rate, but this tells essentially the same story.

```python
sf_cross_tab(df, "graduated_4_year_degree_less_5_years")

```

### Statistical Test

If we run an independent t-test on the 5 year graduation rates, we see that the class of 2014 is not statistically higher than the class of 2013, or 2012. 


#### P Value from t-test comparing 2014 -> 2013

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

#### P Value from t-test comparing 2014 -> 2012 

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
sm.stats.ttest_ind(population1, population2)[1]

```

#### P Value from t-test comparing 2014 -> 2011
Note, this value is < 0.5

```python
population1 = (
    df[(df.site == "San Francisco") & (df.high_school_class == 2011)][
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
sm.stats.ttest_ind(population1, population2)[1]

```

### Other Distributions

Based on the above results, I don't believe we can say that the class of 2014 was notably higher that previous classes. It does however appear there is an upswring in graduation rate that has been increasing since 2011, with a decent jump from 2012 -> 2013 (though not a statistically significant one).

With that in mind, here are other notable differences in the high school class distributions which might indicatate changes that are influencing the graduation rate 


#### 11th Grade College Eligibility GPA

This is the most notable distrobution change, with the class of 2014 having by far the highest 11th grade GPAs, with almost 75% of that student group having over a 3.0. Compared to 53% from the class of 2013 (the next highest)

```python
sf_cross_tab(df, "gpa_bucket")
```

```python

```
