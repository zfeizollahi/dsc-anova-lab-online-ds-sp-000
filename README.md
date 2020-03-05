# ANOVA  - Lab

## Introduction

In this lab, you'll get some brief practice generating an ANOVA table (AOV) and interpreting its output. You'll also perform some investigations to compare the method to the t-tests you previously employed to conduct hypothesis testing.

## Objectives

In this lab you will: 

- Use ANOVA for testing multiple pairwise comparisons 
- Interpret results of an ANOVA and compare them to a t-test

## Load the data

Start by loading in the data stored in the file `'ToothGrowth.csv'`: 


```python
# Your code here
import pandas as pd
import statsmodels.api as sm
from statsmodels.formula.api import ols
```


```python
df = pd.read_csv('ToothGrowth.csv')
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>len</th>
      <th>supp</th>
      <th>dose</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>4.2</td>
      <td>VC</td>
      <td>0.5</td>
    </tr>
    <tr>
      <td>1</td>
      <td>11.5</td>
      <td>VC</td>
      <td>0.5</td>
    </tr>
    <tr>
      <td>2</td>
      <td>7.3</td>
      <td>VC</td>
      <td>0.5</td>
    </tr>
    <tr>
      <td>3</td>
      <td>5.8</td>
      <td>VC</td>
      <td>0.5</td>
    </tr>
    <tr>
      <td>4</td>
      <td>6.4</td>
      <td>VC</td>
      <td>0.5</td>
    </tr>
  </tbody>
</table>
</div>




```python
df['dose'].value_counts()
```




    2.0    20
    1.0    20
    0.5    20
    Name: dose, dtype: int64



## Generate the ANOVA table

Now generate an ANOVA table in order to analyze the influence of the medication and dosage:  


```python
# Your code here
formula = 'len ~ C(supp) + C(dose)'
lm = ols(formula,df).fit() 
table = sm.stats.anova_lm(lm, typ=2)
print(table)
```

                   sum_sq    df          F        PR(>F)
    C(supp)    205.350000   1.0  14.016638  4.292793e-04
    C(dose)   2426.434333   2.0  82.810935  1.871163e-17
    Residual   820.425000  56.0        NaN           NaN


## Interpret the output

Make a brief comment regarding the statistics and the effect of supplement and dosage on tooth length: 

Dose is many times more of a contributing factor than supplement.


## Compare to t-tests

Now that you've had a chance to generate an ANOVA table, its interesting to compare the results to those from the t-tests you were working with earlier. With that, start by breaking the data into two samples: those given the OJ supplement, and those given the VC supplement. Afterward, you'll conduct a t-test to compare the tooth length of these two different samples: 


```python
# Your code here
oj = df[df['supp'] == 'OJ']
vc = df[df['supp'] == 'VC']
```

Now run a t-test between these two groups and print the associated two-sided p-value: 


```python
# Calculate the 2-sided p-value for a t-test comparing the two supplement groups
from scipy import stats
```


```python
stats.ttest_ind(oj['len'], vc['len'], equal_var=False)
```




    Ttest_indResult(statistic=1.91526826869527, pvalue=0.06063450788093387)



## A 2-Category ANOVA F-test is equivalent to a 2-tailed t-test!

Now, recalculate an ANOVA F-test with only the supplement variable. An ANOVA F-test between two categories is the same as performing a 2-tailed t-test! So, the p-value in the table should be identical to your calculation above.

> Note: there may be a small fractional difference (>0.001) between the two values due to a rounding error between implementations. 


```python
# Your code here; conduct an ANOVA F-test of the oj and vc supplement groups.
# Compare the p-value to that of the t-test above. 
# They should match (there may be a tiny fractional difference due to rounding errors in varying implementations)
formula = 'len ~ C(supp)'
lm = ols(formula,df).fit() 
table = sm.stats.anova_lm(lm, typ=2)
print(table)
```

                   sum_sq    df         F    PR(>F)
    C(supp)    205.350000   1.0  3.668253  0.060393
    Residual  3246.859333  58.0       NaN       NaN


## Run multiple t-tests

While the 2-category ANOVA test is identical to a 2-tailed t-test, performing multiple t-tests leads to the multiple comparisons problem. To investigate this, look at the various sample groups you could create from the 2 features: 


```python
for group in df.groupby(['supp', 'dose'])['len']:
    group_name = group[0]
    data = group[1]
    print(group_name)
```

    <itertools.combinations object at 0x7f87e8b5cbd8>
    ('OJ', 0.5)
    ('OJ', 1.0)
    ('OJ', 2.0)
    ('VC', 0.5)
    ('VC', 1.0)
    ('VC', 2.0)


While bad practice, examine the effects of calculating multiple t-tests with the various combinations of these. To do this, generate all combinations of the above groups. For each pairwise combination, calculate the p-value of a 2-sided t-test. Print the group combinations and their associated p-value for the two-sided t-test.


```python
# Your code here; reuse your t-test code above to calculate the p-value for a 2-sided t-test
# for all combinations of the supplement-dose groups listed above. 
# (Since there isn't a control group, compare each group to every other group.)
```


```python
from itertools import combinations
combos = combinations(df.groupby(['supp', 'dose']), 2)
combo_pvals = {}
for combo in combos:
    results = stats.ttest_ind(combo[0][1]['len'], combo[1][1]['len'])
    print('Combo:{} , {} p_val: {}'.format(combo[0][0], combo[1][0],results[1]))
```

    Combo:('OJ', 0.5) , ('OJ', 1.0) p_val: 8.357559281443774e-05
    Combo:('OJ', 0.5) , ('OJ', 2.0) p_val: 3.4018585295016214e-07
    Combo:('OJ', 0.5) , ('VC', 0.5) p_val: 0.005303661339923052
    Combo:('OJ', 0.5) , ('VC', 1.0) p_val: 0.04223992429368205
    Combo:('OJ', 0.5) , ('VC', 2.0) p_val: 7.025409196997986e-06
    Combo:('OJ', 1.0) , ('OJ', 2.0) p_val: 0.03736279585664383
    Combo:('OJ', 1.0) , ('VC', 0.5) p_val: 1.3372624230559434e-08
    Combo:('OJ', 1.0) , ('VC', 1.0) p_val: 0.0007807261651774468
    Combo:('OJ', 1.0) , ('VC', 2.0) p_val: 0.09583711277517494
    Combo:('OJ', 2.0) , ('VC', 0.5) p_val: 1.3381068810881244e-11
    Combo:('OJ', 2.0) , ('VC', 1.0) p_val: 2.3131084633597503e-07
    Combo:('OJ', 2.0) , ('VC', 2.0) p_val: 0.9637097790041267
    Combo:('VC', 0.5) , ('VC', 1.0) p_val: 6.492264598157612e-07
    Combo:('VC', 0.5) , ('VC', 2.0) p_val: 4.957285658438862e-09
    Combo:('VC', 1.0) , ('VC', 2.0) p_val: 3.397577925539582e-05


## Summary

In this lesson, you implemented the ANOVA technique to generalize testing methods to multiple groups and factors.
