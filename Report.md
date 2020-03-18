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

That being said, it still is important to understand if any factors seemed different among these students that might have had a correlation with graduation rates. When considering the data available, 11th grade GPA and first year of College GPA seemed to play the most important role in graduation success.  

Other factors considered, but yield mixed or negligible results include: 
* First Gen Status
* Low Income
* Ethnic Background
* ACT scores
* High School Attended* 
* College Fit Type
* High School Attrition

\* For high school attended, there was a positive correlation with students who attended Lowell High School

Finally, the switch to site based advising occurred around this time. This is a harder indicator to effectively measure, but the results did indicate there *could* be something to this, but it is currently too difficult to pin down the extent of the significance. 




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
# original df file only included students who completed hs program. later removed that filter,
#so changing df to be the original subset, and df_master to be the whole group

df_master = df

df = df[df.indicator_completed_ct_hs_program == True]
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
# creating a subset df that is just San Francisco
df_sub = df[df.site == "San Francisco"]

df_sub = df_sub[df_sub.high_school_class<=2014]
```

<!-- #region hide_input=true -->
##  General Distributions

SF Class of 2014 is on track to have the highest 6 year grad rate, with almost 70% of students already graduating, but that number isn't significantly higher than the class of 2013. Though we do see a reasonably big jump from 2012 to 2013. 


<!-- #endregion -->

#### Table 1. San Francisco 6 Year Graduation Rate by High School Class 

```python
# Grad Rate Less than 6 years

grad_rate_6_year = sf_cross_tab(
    df, "graduated_4_year_degree_less_6_years").round(2)

grad_rate_6_year.round(2).style.format('{:.0%}')
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

grad_rate_5_year.round(2).style.format('{:.0%}')

```

### Statistical Test

If we run an independent t-test on the 5 year graduation rates, we see that the class of 2014 is not statistically higher than the class of 2013, nor is the class of 2013 statistically higher than the class of 2012.


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
print("p Value: ", (round(sm.stats.ttest_ind(population1_test_1, population2_test_1)[1], 2)))
```

#### P Value from t-test comparing 2013 -> 2012 


```python
population1_test_2 = (
    df[(df.site == "San Francisco") & (df.high_school_class == 2012)][
        "graduated_4_year_degree_less_5_years"
    ]
).values


population2_test_2 = (
    df[(df.site == "San Francisco") & (df.high_school_class == 2013)][
        "graduated_4_year_degree_less_5_years"
    ]
).values
```

```python
print(" p Value: ",(round(sm.stats.ttest_ind(population1_test_2, population2_test_2)[1], 2)))
```

### Other Distributions

Based on the above results, I don't believe we can say that the class of 2014 was notably higher that previous classes. It does however appear there is an upswing in graduation rate that has been increasing since 2011, with a decent jump from 2012 -> 2013 (though not a statistically significant one).

With that in mind, here are other notable differences in the high school class distributions which might indicate changes that are influencing the graduation rate 


#### 11th Grade College Eligibility GPA

This is the most notable distribution change, with the class of 2014 and 2013 having higher GPAs than the previous classes. The class of 2014 had almost 75% of students with a GPA above 3.0, and the class of 2013 had 54% of student above 3.0. Compared to 45% for the class of 2012.

```python
gpa_buckets = sf_cross_tab(df, "11th_grade_gpa_bucket", margins=False ).round(2)



gpa_buckets.round(2).style.format('{:.0%}')
```

#### Pearson Correlation Coefficient for key continuous data points (SF - all classes)

Below is the correlation matrix for key continuous variables that might influence college graduation. For the purposes here, only the first column is displays relevant information. 

We can see that GPA in general plays the largest factor, with 11th grade and the first year of college particularly important. Bank book earnings played the had the second highest correlation to graduation success. 

Note - there are no notable differences between the Pearson Coefficient and the Spearman Coefficient


```python
# cleaning up column types (needs to be moved to data prep)

# changing act_math to an int
df['act_mathematics'] = pd.to_numeric(df['act_mathematics'], errors='coerce')

# changing bank book earnings to an int
df['total_bb_earnings_as_of_hs_grad'] = df['total_bb_earnings_as_of_hs_grad'].replace(
    '[\$,]', '', regex=True).astype(float)

# changing gpa data to int
df[['9th_grade', '10th_grade', '11th_grade',
    '12th_grade', 'year_1', 'year_2', 'year_3',
    'year_4', 'year_5', 'year_6']] = df[
    ['9th_grade', '10th_grade', '11th_grade',
    '12th_grade', 'year_1', 'year_2', 'year_3',
    'year_4', 'year_5', 'year_6']].apply(pd.to_numeric, errors='coerce')
```

```python
corrMatrix = df_sub[['graduated_4_year_degree_less_5_years', '9th_grade', '10th_grade', '11th_grade',
                     '12th_grade', 'year_1', 'year_2', 'year_3',
                     'year_4', 'year_5', 'indicator_first_generation', 'indicator_low_income',
                     'total_bb_earnings_as_of_hs_grad', 'act_mathematics']].corr(method='pearson')
```

```python
f, ax = plt.subplots(figsize=(12, 12))
sns.set_context('paper')


ax = sns.heatmap(corrMatrix, vmax=1, square=True, annot=True,  cmap="RdBu")

```

<!-- #region -->
#### Logistic Regression on Categorical Variables (All SF Classes)

Based on the above correlations, it is important to evaluate if any categorical variables are also influencing graduation outcomes. Below are the logistic regression results for selected categorical variables with the notable continuous variables added to the model as well. 

Below is the logistic regression result on the following key categorical variables: 


* Ethnic Background

* College Fit type (using the fit type of the first school attended)

* First Generation 

* High School Attended



When performed on only San Francisco students, the results indicated that still only 11th Grade GPA and Year 1 GPA had a statistically significant influence on graduation success. None of the other categorical indicators were statistically significant. 


*For the high school test, the continuous variables had to be removed for the results to converge due to the large number of schools students attended* 



Only three high schools showed statistically significant values, Lowell, Balboa, and City Arts and Tech, with Lowell being the only one that had a positive influence on students. As a note, prior to this analysis I spoke with Miccaela Montague and she indicated this likely would be the case. 

As a note, for all of these the sample size was quite low.


<!-- #endregion -->

```python
def C1(cat):
     return pd.get_dummies(cat, drop_first=True)
```

```python
mod = smf.logit(formula= "C1(graduated_4_year_degree_less_6_years) ~ year_1  + Q('11th_grade') + C(indicator_first_generation) + total_bb_earnings_as_of_hs_grad + (C(fit_type)-1) + C(ethnic_background)", data=df_sub).fit(method='bfgs', maxiter=100)
mod.summary()
```

```python
mod = smf.logit(formula= "C1(graduated_4_year_degree_less_6_years) ~ C(school)", data=df_sub).fit(method='bfgs', maxiter=100)
mod.summary()
```

#### Logistic Regression performed on all sites and classes

The first regression from above was performed again but using a much larger data set - all sites and classes - but the results were the same. Only Year 1 and 11th Grade GPA were significant. 

```python
mod = smf.logit(formula= "C1(graduated_4_year_degree_less_6_years) ~ year_1  + Q('11th_grade') + C(indicator_first_generation) + total_bb_earnings_as_of_hs_grad + (C(fit_type)-1) + C(ethnic_background)", data=df).fit(method='bfgs', maxiter=1000)
mod.summary()
```

### San Francisco High School Attrition

Another possibility, is the classes of 2013 and 2014 has a lower percent of students who completed the High School program, which could mean only the most qualified students were included in our college graduation numbers. 

The below table list the percent of students who started college track and then completed the high school program. We do see in 2013 a smaller number, 26% of students completed the program, but that number quickly returned to 36% for the class of 2014. 

```python
sf_cross_tab(df_master, "indicator_completed_ct_hs_program").round(2).style.format('{:.0%}')
```

## SF Compared to Other Sites

San Francisco piloted site based modeling for the class of 2013, and the rest of College Track transitioned to this model in for the class of 2014. 

This is a harder metric to evaluate given that starting in 2014 all sites received site based advising, so San Francisco only has one year of only their students receiving site advising - a difficult length of time to pin down. Moreover, as we move back in time data becomes more limited or inaccurate, which makes comparing sites graduation rates prior to any advising harder to do. 

Still, from this limited analysis it does appear there *could* be a correlation with San Francisco's switch to site based advising. 




#### Class of 2013 graduation rate compared to other CT Sites

This table shows the difference in graduation rate between SF class of 2013 and all other sites - that starting in 2013 were not using site based modeling (however starting in 2014 all sites were using this model).

We can see that SF has a higher graduation rate, and the p value indicates this is a statistically significant difference. 

```python
class_cross_tab(df, "graduated_4_year_degree_less_6_years", hs_class=2013).round(2).style.format('{:.0%}')

```

```python
population1_test_4 = (
    df[(df.site == "San Francisco") & (df.high_school_class == 2013)][
        "graduated_4_year_degree_less_6_years"
    ]
).values


population2_test_4 = (
    df[(df.site != "San Francisco") & (df.high_school_class == 2013)][
        "graduated_4_year_degree_less_6_years"
    ]
).values
```

```python
print("p Value: ",(round(sm.stats.ttest_ind(population1_test_4, population2_test_4)[1], 5)))
```

#### Class of 2011 and 2012 graduation rate compared to other CT Sites

When comparing the class of 2011 and 2012 though, at the time when no sites were using site based advising, San Francisco did not have a statistically higher graduation rate. 

```python
pd.crosstab(
        df[df.high_school_class <= 2012].site,
        df[df.high_school_class <= 2012]['graduated_4_year_degree_less_6_years'],
        normalize='index',
        margins=True,
    ).round(2).style.format('{:.0%}')
```

```python
population1_test_5 = (
    df[(df.site == "San Francisco") & (df.high_school_class <= 2012)][
        "graduated_4_year_degree_less_6_years"
    ]
).values


population2_test_5 = (
    df[(df.site != "San Francisco") & (df.high_school_class <= 2012)][
        "graduated_4_year_degree_less_6_years"
    ]
).values
```

```python
print("p Value: ", (round(sm.stats.ttest_ind(population1_test_5, population2_test_5)[1], 2)))
```

#### Class of 2014 and 2015 graduation rate compared to other CT Sites

Finally when looking at the class of 2014 and 2015, when all sites were using site based advising, San Francisco still appears to have a higher graduation rate.

```python
pd.crosstab(
        df[df.high_school_class >= 2014].site,
        df[df.high_school_class >= 2014]['graduated_4_year_degree_less_6_years'],
        normalize='index',
        margins=True,
    ).round(2).style.format('{:.0%}')
```

```python
population1_test_6 = (
    df[(df.site == "San Francisco") & (df.high_school_class >= 2014)][
        "graduated_4_year_degree_less_6_years"
    ]
).values


population2_test_6 = (
    df[(df.site != "San Francisco") & (df.high_school_class >= 2014)][
        "graduated_4_year_degree_less_6_years"
    ]
).values
```

```python
print('p Value: ',(round(sm.stats.ttest_ind(population1_test_6, population2_test_6)[1], 2)))
```

```html hide_input=true

<script>
$(document).ready(function(){
    window.code_toggle = function() {
        (window.code_shown) ? $('div.input').hide(250) : $('div.input').show(250);
        window.code_shown = !window.code_shown
    }
    if($('body.nbviewer').length) {
        $('<li><a href="javascript:window.code_toggle()" title="Show/Hide Code"><span class="fa fa-code fa-2x menu-icon"></span><span class="menu-text">Show/Hide Code</span></a></li>').appendTo('.navbar-right');
        window.code_shown=false;
        $('div.input').hide();
    }
});
</script>


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

.cell {
    padding: 0px;
}


</style>
```

```python

```
