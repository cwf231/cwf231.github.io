---
layout: post
title:      "Linear Regression Models and `variance_inflation_factor()`"
date:       2020-08-02 15:06:14 -0400
permalink:  linear_regression_models_and_variance_inflation_factor
---


Linear Regression models are great for their highly descriptive nature - once a model is built, its summary will tell you exactly how it's using the variables that it's working with.

These models are simple and strong, but have their drawbacks. I wanted to journal some of my first experiences with one of Linear Regression's assumptions: **multicolinearity**.

***

Multicolinearity occurs when multiple variables correlate with one another within a model. The presence of multicolinearity in a model will damage the interpretability by playing tricks with the coefficients attached to the affected variables. Multicolinearity does not damage the predictive power of these models, but should be carefully considered in order to fully understand what's happening on the inside.

To run through a simple example, let's build a linear regression model with the `mpg` dataset in `seaborn`. **Note:** *We are going to ignore most of the assumptions for building linear regression models and only focus on multicolinearity.*


First, import the necessary libraries:
```python
import pandas as pd
import numpy as np

import seaborn as sns
import matplotlib.pyplot as plt
plt.style.use('ggplot')

from itertools import combinations

import statsmodels.api as sm
import statsmodels.formula.api as smf
from statsmodels.stats.outliers_influence import variance_inflation_factor
```

And load the data:
```python
df = sns.load_dataset('mpg')
df.head()
```
<img src="https://raw.githubusercontent.com/cwf231/dominant_pitcher/master/blog_imgs/2df_head.PNG">

For our quick and dirty example, we'll just drop the rows with missing values...
```python
df.dropna(inplace=True)
df.reset_index(drop=True, inplace=True)
df.info()
```
<img src="https://raw.githubusercontent.com/cwf231/dominant_pitcher/master/blog_imgs/3df_info.PNG">

We can also quickly drop the categorical columns `origin, name`, as well as `cylinders` for this example.
```python
df.drop(columns=['origin', 'cylinders', 'name'], inplace=True)
df.head()
```
<img src="https://raw.githubusercontent.com/cwf231/dominant_pitcher/master/blog_imgs/4df_head.PNG">

Ok, we're ready to go.

The first thing I like to do is quickly see the relationships of all the variables. We can quickly do this with `pandas` scatter matrix function:
```python
pd.plotting.scatter_matrix(df, figsize=(12,12))
```
<img src="https://raw.githubusercontent.com/cwf231/dominant_pitcher/master/blog_imgs/5scattermatrix.PNG">

At a glance, we can see a lot of correlation with these variables just by the scatterplot. There are many ways we could transform the data to suit our needs, but for now we'll leave it alone and check out the raw correlations.

One way to do this is with a heatmap. Seaborn as a nice one:
```python
fig, ax = plt.subplots(figsize=(8,6))

corr = df.corr().abs().round(3)

# Set a 'mask' for the upper half of the heatmap.

mask = np.triu(np.ones_like(corr, dtype=np.bool))

sns.heatmap(corr, annot=True, mask=mask, cmap='Reds', ax=ax)
plt.setp(ax.get_xticklabels(), rotation=0, ha="center",)
plt.setp(ax.get_yticklabels(), rotation=0)

# Hack to fix the cutoff squares and remove empty row and column.

ax.set_ylim(len(corr), 1)
ax.set_xlim(xmax=len(corr)-1)

fig.tight_layout()
```
<img src="https://raw.githubusercontent.com/cwf231/dominant_pitcher/master/blog_imgs/6corr_matrix.PNG">

Indeed, there are some obviously high correlations here. There are many opinions of what does *high correlation* exactly mean. We can say without argument that all the relationships which are higher than `0.75` in this case are highly correlated.

We can also see this in a dataframe:
```python
# Set up the dataframe.

X = df.drop('mpg', axis=1)
y = df[['mpg']]

# Create a dataframe of correlation coefficients.

cc_df = X.corr().abs().stack().reset_index().sort_values(0, ascending=False)

# Create column with each pair or variables.

cc_df['pair'] = tuple(zip(cc_df['level_0'], cc_df['level_1']))

# Set the index to the pairs and drop the individual columns.

cc_df = cc_df.set_index('pair').drop(columns=['level_0', 'level_1'])

# Rename column.

cc_df.columns = ['correlation_coef']

# Drop rows that are comparing the variable to itself.

cc_df.drop([tup for tup in cc_df.index if tup[0] == tup[1]], inplace=True)

# Find the unique tuples. (eg: (a, b) = (b, a))

unique_tups = []
for (a, b) in cc_df.index:
    if (a, b) in unique_tups or (b, a) in unique_tups:
        continue
    unique_tups.append((a, b))
    
# Slice out the unique pairs.

cc_df = cc_df.loc[unique_tups,:]

cc_df
```
<img src="https://raw.githubusercontent.com/cwf231/dominant_pitcher/master/blog_imgs/7cc_df.PNG">

***

Now, one way to deal with multicolinearity is to adjust this matrix until nothing is highly correlated (removing variables). There *is* another way however: `variance_inflation_factor()` from `statsmodels`.

- Here is the (very thin) documentation: https://www.statsmodels.org/stable/generated/statsmodels.stats.outliers_influence.variance_inflation_factor.html

Here's the simple functionality:
- It takes a two-dimentional array as its first argument (**not** a dataframe) and a column index as its second argument.

For example:
```python
# First argument as an array.

X.values
```
<img src="https://raw.githubusercontent.com/cwf231/dominant_pitcher/master/blog_imgs/9X_values.PNG">


`variance_inflation_factor(X.values, 0)` returns `48.257254015666476` (which is pointing towards `X[0]`, or the first column of `X`: `'displacement'`)

***

My first experience with this function ended badly however...

I carried on and looped through each column to find the VIF score attached to each column.
- Keep in mind, from the documentation: 

> "The variance inflation factor is a measure for the increase of the variance of the parameter estimates if an additional variable, given by exog_idx is added to the linear regression. It is a measure for multicollinearity of the design matrix, exog.

> **One recommendation is that if VIF is greater than 5, then the explanatory variable given by exog_idx is highly collinear with the other explanatory variables**, and the parameter estimates will have large standard errors because of this."

So, as I was saying, I looped through the array to find the VIF for each variable:
```python
# Initialize dictionary.

vif_dct = {}

# Loop through each row and set the variable name to the VIF.

for i in range(len(X.columns)):
    vif = variance_inflation_factor(X.values, i)
    v = X.columns[i]
    vif_dct[v] = vif

vif_dct
```
<img src="https://raw.githubusercontent.com/cwf231/dominant_pitcher/master/blog_imgs/11vif_dctbad.PNG">

Uh oh, this is very bad. Anything more than 5 is bad, and my lowest is over 60...

Well, it turns out there is an open issue on the subject (here: https://github.com/statsmodels/statsmodels/issues/2376) stating (importantly) that there needs to be a constant included in the array in order for the function to work properly.

So, let's try again...
```python
# Very important...

const_X = sm.add_constant(X)

vif_dct = {}

for i in range(1, len(const_X.columns)): # Excluding 'const' column.
    vif = variance_inflation_factor(const_X.values, i) # Check VIF for col[i]
    v = const_X.columns[i]
    vif_dct[v] = vif

vif_dct
```
<img src="https://raw.githubusercontent.com/cwf231/dominant_pitcher/master/blog_imgs/12vif_dctgood.PNG">

That makes much more sense - especially considering the correlation coefficients we saw earlier (check the heatmap or correlation df above...)

***

To wrap up the example, let's quickly see the interactions of all the variables and their VIF with one another.
The plan is to loop through all combinations of all sizes (>1) and investigate the VIF scores of those variables.
```python
# Set up for DataFrame.

columns = ['num_variables', 'variables', 'vif_dct', 'num_high_vif']
rows = []

# Loop through number of selections per combo.

for num_variables in range(2, len(X.columns)+1):
    # Get combinations of length num_variables.
		
    combos = list(combinations(X.columns, num_variables))
    # Iterate through the combos and find the VIF scores.
		
    for combo in [list(x) for x in combos]:
        new_X = sm.add_constant(X[combo])
        vif_dct = {}
        for i in range(1, len(new_X.columns)):
            v = new_X.columns[i]
            vif = variance_inflation_factor(new_X.values, i)
            vif_dct[v] = vif
        num_high_vif = len({k: v for (k, v) in vif_dct.items() if v > 5})
        rows.append([num_variables, combo, vif_dct, num_high_vif])
        
vif_interactions = pd.DataFrame(rows, columns=columns)
vif_interactions
```
<img src="https://raw.githubusercontent.com/cwf231/dominant_pitcher/master/blog_imgs/14vif_df.PNG">

A very interesting dataframe which is worth exploring (I'll leave the playing to you).

***

Finally, *what do we do with variables with high VIF*?
There isn't an exact answer, but one way is to find out which variable, among the "troublemakers", is most highly correlated with the target and use *that*:
```python
df[correlated + ['mpg']].corr().abs()['mpg']
```
<img src="https://github.com/cwf231/dominant_pitcher/blob/master/blog_imgs/16corr_with_target.PNG">

This would point to us using `weight` for our model.

***

For conclusiveness, let's just make the model!
```python
target = 'mpg'
x_variables = ['weight', 'acceleration', 'model_year']

formula = f'{target}~{"+".join(x_variables)}'
formula #'mpg~weight+acceleration+model_year'

model = smf.ols(formula, df).fit()
model.summary()
```

<img src="https://raw.githubusercontent.com/cwf231/dominant_pitcher/master/blog_imgs/18model_summary.PNG">
