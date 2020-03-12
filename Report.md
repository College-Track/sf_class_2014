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

### Abstract

The SF Class of 2014 is currently the highest performing CT class in terms of 5 year college graduation rate. They almost certainly will be have the 6 year graduation rate at the end of FY20. This report investigated what variables might have influenced this classes success, and if any of those indicators are something which could be encouraged at other CT sites. 

While it is true the Class of 2014 will have the highest graduation rate, almost more notable is the jump in graduation rates from 2012 to 2013. In some senses, the class of 2014 was just continuing the impressive results their seniors started. However, given the small sample sizes for any given College Track cohort, it is difficult to draw conclusions on the statistical significance of these increases. In fact, looking at the t-test results indicated that the difference in graduation rates could not be considered pure randomness. 

That being said, it still is important to understand if any factors seemed different among these students that might have had a correlation with graduation rates. When considering the data available, 11th grade GPA, first year of College GPA, and, more interesting, students total bank book earnings, seemed to play an important role in graduation success. 

Other factors considered, but yield mixed or negligible results include: 
* First Gen Status
* Low Income
* Ethnic Background
* ACT scores
* High School Attended* 
* College Fit Type

\* For high school attended, there was a positive correlation with students who attended Lowell High School

Finally, the switch to site based advising occurred around this time...TO BE COMPLETED




```python
import pandas as pd
from pathlib import Path
from datetime import datetime
import statsmodels.api as sm
import numpy as np
from tabulate import tabulate
import seaborn as sns
import matplotlib.pyplot as plt
import warnings
import statsmodels.formula.api as smf

warnings.filterwarnings('ignore')


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
def sf_cross_tab(df, column, normalize="index", margins=True):
    return pd.crosstab(
        df[df.site == "San Francisco"].high_school_class,
        df[df.site == "San Francisco"][column],
        normalize=normalize,
        margins=margins,
    )
```

##  General Distributions

SF Class of 2014 is on track to have the highest 6 year grad rate, with almost 70% of students already graduating, but that number isn't significantly higher than the class of 2013. Though we do see a reasonably big jump from 2012 to 2013. 




#### Table 1. San Francisco 6 Year Graduation Rate by High School Class 

```python
# Grad Rate Less than 6 years

grad_rate_6_year = sf_cross_tab(
    df, "graduated_4_year_degree_less_6_years").round(2)

grad_rate_6_year
```

<!-- #region hide_input=true -->
#### Table 2. San Francisco 6 Year Graduation Count by High School Class 

<!-- #endregion -->

```python
grad_numbers_6_year = sf_cross_tab(df, "graduated_4_year_degree_less_6_years", normalize=False).round(2)

grad_numbers_6_year
```

#### Table 3. San Francisco 5 Year Graduation Rate by High School Class

To be more accurate, we can look at the 5 year grad rate, but this tells essentially the same story.

```python
grad_rate_5_year = sf_cross_tab(df, "graduated_4_year_degree_less_5_years").round(2)

grad_rate_5_year

```

### Statistical Test

If we run an independent t-test on the 5 year graduation rates, we see that the class of 2014 is not statistically higher than the class of 2013, or 2012. 


#### P Value from t-test comparing 2014 -> 2013

```python
population1_test_1 = (
    df[(df.site == "San Francisco") & (df.high_school_class == 2013)][
        "graduated_4_year_degree_less_5_years"
    ]
).values


population2_test_1 = (
    df[(df.site == "San Francisco") & (df.high_school_class == 2014)][
        "graduated_4_year_degree_less_5_years"
    ]
).values
```

```python
# p value of independent t-test on populations above
(round(sm.stats.ttest_ind(population1_test_1, population2_test_1)[1], 2))
```

#### P Value from t-test comparing 2014 -> 2012 

```python
population1_test_2 = (
    df[(df.site == "San Francisco") & (df.high_school_class == 2012)][
        "graduated_4_year_degree_less_5_years"
    ]
).values


population2_test_2 = (
    df[(df.site == "San Francisco") & (df.high_school_class == 2014)][
        "graduated_4_year_degree_less_5_years"
    ]
).values
```

```python
(round(sm.stats.ttest_ind(population1_test_2, population2_test_2)[1], 2))
```

#### P Value from t-test comparing 2014 -> 2011
Note, this value is < 0.5

```python
population1_test_3 = (
    df[(df.site == "San Francisco") & (df.high_school_class == 2011)][
        "graduated_4_year_degree_less_5_years"
    ]
).values


population2_test_3 = (
    df[(df.site == "San Francisco") & (df.high_school_class == 2014)][
        "graduated_4_year_degree_less_5_years"
    ]
).values
```

```python
(round(sm.stats.ttest_ind(population1_test_3, population2_test_3)[1], 2))
```

### Other Distributions

Based on the above results, I don't believe we can say that the class of 2014 was notably higher that previous classes. It does however appear there is an upswing in graduation rate that has been increasing since 2011, with a decent jump from 2012 -> 2013 (though not a statistically significant one).

With that in mind, here are other notable differences in the high school class distributions which might indicate changes that are influencing the graduation rate 


#### 11th Grade College Eligibility GPA

This is the most notable distribution change, with the class of 2014 having by far the highest 11th grade GPAs, with almost 75% of that student group having over a 3.0. Compared to 53% from the class of 2013 (the next highest)

```python
gpa_buckets = sf_cross_tab(df, "gpa_bucket", margins=False).round(2)



gpa_buckets
```

#### CT Entrance Diagnostics Math Scores
If we look at each high school classes incoming CT math diagnostics scores, we also see that the class of 2014 had the highest percent of students either "Ready" or "Near Ready." Noteworthy though, is the class of 2013 did not have any appreciable differences from the class of 2012. 

```python
entrace_scores = df3[df3.version == "Entrance into CT Diagnostic"]
```

```python
entrance_scores = sf_cross_tab(entrace_scores, "act_math_readiness", margins=False).round(2)

entrance_scores
```

#### Pearson Correlation Coefficient for key continuous data points (SF - all classes)

Below is the correlation matrix for key continuous variables that might influence college graduation. For the purposes here, only the first column is displays relevant information. 

We can see that GPA in general plays the largest factor, with 11th grade and the first year of college particularly important. Somewhat surprising, was the role bank book earnings played, second only to 11th grade gpa. 

Note - there are no notable differences between the Pearson Coefficient and the Spearman Coefficient


```python
# cleaning up column types (needs to be moved to data prep)

# changing act_math to an int
df['act_mathematics'] = pd.to_numeric(df['act_mathematics'], errors='coerce')

# changing bank book earnings to an int
df['total_bb_earnings_as_of_hs_grad'] = df['total_bb_earnings_as_of_hs_grad'].replace(
    '[\$,]', '', regex=True).astype(float)

# changing gpa data to int
df[['9th Grade', '10th Grade', '11th Grade',
    '12th Grade', 'Year 1', 'Year 2', 'Year 3',
    'Year 4', 'Year 5', 'Year 6']] = df[
    ['9th Grade', '10th Grade', '11th Grade',
     '12th Grade', 'Year 1', 'Year 2', 'Year 3',
     'Year 4', 'Year 5', 'Year 6']].apply(pd.to_numeric, errors='coerce')
```

```python
# creating a subset df that is just San Francisco
df_sub = df[df.site == "San Francisco"].dropna(subset=['act_mathematics'])

```

```python
corrMatrix = df_sub[['graduated_4_year_degree_less_5_years', '9th Grade', '10th Grade',
                     '11th Grade', '12th Grade', 'Year 1', 'Year 2', 'Year 3',
                     'Year 4', 'Year 5', 'indicator_first_generation', 'indicator_low_income',
                     'total_bb_earnings_as_of_hs_grad', 'act_mathematics']].corr(method='pearson')
```

```python
f, ax = plt.subplots(figsize=(12, 12))
sns.set_context('paper')


ax = sns.heatmap(corrMatrix, vmax=1, square=True, annot=True,  cmap="RdBu")

```

<!-- #region -->
#### Logistic Regression on Categorical Variables (All SF Classes)

Below is the logistic regression result on the following key categorical variables: 

* High School Attended

* Ethnic Background

* College Fit type (using the fit type of the first school attended)



All three of these test yielded mixed results with no clear indicator of strong correlation.

Only three high schools showed statistically significant values, Lowell, Balboa, and City Arts and Tech, with Lowell being the only one that had a positive influence on students. As a note, prior to this analysis I spoke with Miccaela Montague and she indicated this likely would be the case. 


For ethnic background, only Asian Americans had a small p value, but the "R squared is still quite low and the overall sample size for any individual ethnicity is low. 

Finally, fit type yielded minimal results with. The sample size for Best Fit schools was very small, with almost all the students being marked as attending a Good Fit school. 

As a note, for all of these the sample size was quite low.


<!-- #endregion -->

```python
def C1(cat):
     return pd.get_dummies(cat, drop_first=True)
```

```python
mod = smf.logit(formula= "C1(graduated_4_year_degree_less_5_years) ~ C(school)", data=df_sub).fit(method='bfgs', maxiter=100)
mod.summary()
```

```python
mod = smf.logit(formula= "C1(graduated_4_year_degree_less_5_years) ~ C(ethnic_background) - 1 ", data=df_sub).fit(method='bfgs', maxiter=100)
mod.summary()
```

```python
mod = smf.logit(formula= "C1(graduated_4_year_degree_less_5_years) ~ C(fit_type)-1", data=df_sub).fit(method='bfgs', maxiter=100)
mod.summary()
```

```python
# np.exp(mod.params)

```

```html
<script src="https://cdn.rawgit.com/parente/4c3e6936d0d7a46fd071/raw/65b816fb9bdd3c28b4ddf3af602bfd6015486383/code_toggle.js"></script>

```

```html

<style>
div.prompt {display:none}


h1, .h1 {
    font-size: 33px;
    font-family: "Trebuchet MS";
    font-size: 2.5em !important;
    color: #2a7bbd;
}

h2, .h2 {
    font-size: 10px;
    font-family: "Trebuchet MS";
    color: #2a7bbd; 
    
}


h3, .h3 {
    font-size: 10px;
    font-family: "Trebuchet MS";
    color: #5d6063; 
    
}

.rendered_html table {

    font-size: 14px;
}

.output_png {
  display: flex;
  justify-content: center;
}



</style>
```
