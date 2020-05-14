---
layout: post
title:      "The Best in Baseball"
date:       2020-05-14 09:15:42 -0400
permalink:  the_best_in_baseball
---

## Exploring, creating, visualizing data.

---


I'm always curious when people talk about *the best player of all time* in a sport. It’s an impossible comparison and, yes – that’s the fun part. That didn’t stop me from wanting to quantify it as best I could. I wanted to come up with a metric which could compare players across eras.

Baseball is my favorite sport, so I decided to make a stat for pitchers.

I decided that a player’s season could be quantified by their **dominance in the era they played in** (compared to other players in the same era).

The overall goal is simple: Create a metric which compares a pitcher’s overall season against the next-best pitcher’s season.

---

* The first step was to get some data. From Kaggle ([Kaggle - The History of Baseball](https://www.kaggle.com/seanlahman/the-history-of-baseball)) I downloaded a dataset of pitching stats from decades of the regular season. I decided to only focus on the years 1955-2015.

* Next, I had to make a single, numerical stat which could quantify the entire season. I decided, very unimaginatively, to call it **TPS** (The Pitching Stat). It factors in strikeouts, baserunners, earned runs, and outs recorded. Having never made a stat before, there was some guesswork involved. I also do not claim that this stat is perfect, but I do admire the way it came out.

* After that, we implement the *real* stat (also unimaginatively): **DPS** (Dominant Pitching Stat). This would compare the top TPS in each season to the second-best. The higher a player’s DPS in a season, the better they were than *every* other pitcher in the game. 

(Note that unlike WAR (Wins Against Replacement), we aren’t comparing to the league average, we’re comparing to the other great players in the league to see who really stood out the most.)

---

Let's code some visualizations:

```python
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np
import seaborn as sns
%matplotlib inline
```

To start, let's just have a look at the breakdown year-by-year of our **TPS** stat.

```python
# Boxplot for each year in the dataset.

fig, ax = plt.subplots(figsize=(12, 8))

sns.boxplot(x='year', y='tps', data=df, ax=ax)

ax.yaxis.set_visible(False)

ax.xaxis.set_ticks([])

ax.set(title='"The Pitching Stat"', xlabel='Years (1955-2015)')
```

<img src="https://raw.githubusercontent.com/cwf231/dominant_pitcher/master/images/boxtps.png">

Each year, the majority of the league is in the same cluster. What's most interesting about this stat will be the outlires.

---

What I wanted to look at next was the **league-leader** and the second-finisher in TPS.

For this, I concatinated a list of dataframes - each one with the top two TPS per year.
```python
# Create df of top two TPS per year

df_lst = [df[df['year'] == year].sort_values('tps', ascending=False).head(2) 
          for year in df.year.unique()]

top2_df = pd.concat(df_lst)
top2_df.head()
```

And now we can have a look at `top2_df`.

```python
# Plot the difference between the top two TPS per year.

fig, ax = plt.subplots(figsize=(12, 8))

sns.scatterplot(x='year', y='tps', alpha=1, data=top2_df, ax=ax)

for year in top2_df.year.unique():
    y1 = top2_df[top2_df['year'] == year]['tps'].iloc[0]
    y2 = top2_df[top2_df['year'] == year]['tps'].iloc[1]
    plt.plot([year, year], [y1, y2], c='black', alpha=0.3)
    
ax.get_yaxis().set_visible(False)
ax.set(title='Difference In TPS per Year')
```

<img src="https://raw.githubusercontent.com/cwf231/dominant_pitcher/master/images/tpsdiff.png">

This is starting to get interesting. This shows the metric I wanted to quantify.

---

Now, let's put these lines up against each other to compare **DPS** (how dominant a pitcher was compared to the next-best).

```python
fig, ax = plt.subplots(figsize=(12, 8))

sns.scatterplot(x='year', y='dps', data=top2_df, ax=ax)

ax.set_title('DPS')
for year in top2_df.year.unique():
    y1 = top2_df[top2_df['year'] == year]['dps'].iloc[0]
    y2 = top2_df[top2_df['year'] == year]['dps'].iloc[1]
    plt.plot([year, year], [y1, y2], c='black', alpha=0.3)

avg = top2_df[top2_df.dps > 0].dps.mean()
avg_line = plt.plot([1955, 2015], [avg, avg], alpha=0.5, ls=':')

ax.yaxis.set_visible(False)
ax.set_xlabel('Year')

for spine in ax.spines:
    ax.spines[spine].set_visible(False)

ax.legend(avg_line, ['Mean DPS'])
```

<img src="https://raw.githubusercontent.com/cwf231/dominant_pitcher/master/images/dpslollipop.png">

---

Now, let's get a sense of the top performers of DPS since 1955.

```python
# Sample plot of the top 10 DPS.

fig, ax = plt.subplots(figsize=(12,8))

dps_sorted_df = df[df['dps'] > 0].head(10)

ax.bar(x='playeryear', height='dps', width=.5, data=dps_sorted_df)

ax.set(title='Most Dominant Pitching Seasons', xlabel='Year')

ax.get_yaxis().set_visible(False)

fig.autofmt_xdate()
```

<img src="https://raw.githubusercontent.com/cwf231/dominant_pitcher/master/images/top10dps.png">

Interesting. Sandy Koufax appears several times...

---

If we look at the top 10 **DPS** and the top 10 **TPS**, we can see how good Koufax was in his career. 

```python
# Sorting tps / dps with tail because barh charts blot bottom-to-top.

top_tps = df.sort_values('tps').tail(10)
top_dps = df.sort_values('dps').tail(10)

# Setting colors to highlight Koufax

colors1 = ['#3366ff' if player.startswith('koufa') else 'lightgrey' 
           for player in top_tps.playeryear]
colors2 = ['#3366ff' if player.startswith('koufa') else 'lightgrey' 
           for player in top_dps.playeryear]

def label_bars(ax):
    """Add value labels to the bars (instead of an x-axis)."""
    
    # Iterate over the rectangle objects that we've charted.
		
    for rect in ax.patches:
        # Determine the placement of the labels.
				
        x_val = rect.get_width()
        y_val = rect.get_y() + rect.get_height() / 2
        label = round(rect.get_width() / 10000, 2)
        
        # Create annotation.
				
        ax.annotate(label, (x_val, y_val), xytext=(0, 0), 
                     textcoords="offset points", 
                     va='center', ha='right')

# Plot the figure.

fig, (ax1, ax2) = plt.subplots(ncols=2, figsize=(12, 8))

ax1.barh(y='playeryear', width='tps', color=colors1, data=top_tps)
ax1.set_title('Best Overall Seasons')

ax2.barh(y='playeryear', width='dps', color=colors2, data=top_dps)
ax2.set_title('Most Dominant Seasons')

for ax in (ax1, ax2):
    ax.xaxis.set_visible(False)
    
    for spine in ax1.spines:
        ax.spines[spine].set_visible(False)
    
    label_bars(ax)

plt.tight_layout()
```

<img src="https://raw.githubusercontent.com/cwf231/dominant_pitcher/master/images/koufax.png">

---

If you're a baseball fan, it may not *shock* you. It did put into perspective how good Sandy Koufax was in his career, especially compared to the other pitchers of his time.
