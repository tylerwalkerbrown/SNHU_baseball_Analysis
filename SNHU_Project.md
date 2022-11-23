```python
import requests
import pandas as pd
from lxml import html
import csv 
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import matplotlib
#Scatter matrix
import plotly.express as px
#H Test
get_ipython().run_line_magic('matplotlib', 'inline')
import scipy.stats as stats 
import math
#Decision Tree
from sklearn import tree

#Encoding categorical data 
#from sklearn.preprocession import LabelEncoder, OneHotEncoder
#labellencoder = LabelEncoder()

```


```python
# snhu_22, snhu_21, snhu_19,snhu_18 - all years taken into consideration (hitting)
#snhu_22_p,snhu_21_p, snhu_19_p,snhu_18_p - all years taken into consideration (pitching)
# frames = List used to concat years frames_p = pitchers concat
#df - combination of all years stored in a pandas df
#correlation - corraltion matrix that will be used to analyze r-squared values visually
#sample_mean- sample mean 
#h_mean- pop mean 
#N - sample size 
#dev -standard dev of population
#variables = subset of highly correlated variables
```


```python
#Problems with data 
####PO are innacurate so cannot accurately find the amount of SO per game
####totals at the end of each year need to be dropped or data will be skewed 
####Year 2020 was excluded from analysis (covid-year)
####
```


```python
#Reading in data
snhu_22 = pd.read_html('https://snhupenmen.com/sports/baseball/stats/2022', header = 0)
snhu_21 = pd.read_html('https://snhupenmen.com/sports/baseball/stats/2021', header = 0)
snhu_19 = pd.read_html('https://snhupenmen.com/sports/baseball/stats/2019', header = 0)
snhu_18 = pd.read_html('https://snhupenmen.com/sports/baseball/stats/2018', header = 0)
snhu_17 = pd.read_html('https://snhupenmen.com/sports/baseball/stats/2017', header = 0)
snhu_16 = pd.read_html('https://snhupenmen.com/sports/baseball/stats/2016', header = 0)
snhu_15 = pd.read_html('https://snhupenmen.com/sports/baseball/stats/2015', header = 0)
```

# Pitchers Cleaning/Formatting


```python
#Converting to a dataframe instead of a list (hitting)
snhu_15_p = pd.DataFrame(snhu_15[7])
snhu_16_p = pd.DataFrame(snhu_16[7])
snhu_17_p = pd.DataFrame(snhu_17[7])
snhu_18_p = pd.DataFrame(snhu_18[7])
snhu_19_p = pd.DataFrame(snhu_19[7])
snhu_21_p = pd.DataFrame(snhu_21[7])
snhu_22_p = pd.DataFrame(snhu_22[7])

#Converting to a dataframe instead of a list (hitting)
snhu_15 = pd.DataFrame(snhu_15[6])
snhu_16 = pd.DataFrame(snhu_16[6])
snhu_17 = pd.DataFrame(snhu_17[6])
snhu_18 = pd.DataFrame(snhu_18[6])
snhu_19 = pd.DataFrame(snhu_19[6])
snhu_21 = pd.DataFrame(snhu_21[6])
snhu_22 = pd.DataFrame(snhu_22[6])
```


```python
#Setting the indexes to drop the Total Columns 
#Total columns will skew means because of large values
snhu_15_p = snhu_15_p.set_index('Date')
snhu_16_p = snhu_16_p.set_index('Date')
snhu_17_p = snhu_17_p.set_index('Date')
snhu_18_p = snhu_18_p.set_index('Date')
snhu_19_p = snhu_19_p.set_index('Date')
snhu_21_p = snhu_21_p.set_index('Date')
snhu_22_p = snhu_22_p.set_index('Date')

#Setting the indexes to drop the Total Columns 
#Total columns will skew means because of large values
snhu_15 = snhu_15.set_index('Date')
snhu_16 = snhu_16.set_index('Date')
snhu_17 = snhu_17.set_index('Date')
snhu_18 = snhu_18.set_index('Date')
snhu_19 = snhu_19.set_index('Date')
snhu_21 = snhu_21.set_index('Date')
snhu_22 = snhu_22.set_index('Date')
```


```python
#Dropping all year end sum columns 
snhu_15_p = snhu_15_p.drop(index = ('Total'))
snhu_16_p = snhu_16_p.drop(index = ('Total'))
snhu_17_p = snhu_17_p.drop(index = ('Total'))
snhu_18_p = snhu_18_p.drop(index = ('Total'))
snhu_19_p = snhu_19_p.drop(index = ('Total'))
snhu_21_p = snhu_21_p.drop(index = ('Total'))
snhu_22_p = snhu_22_p.drop(index = ('Total'))

#Dropping all year end sum columns 
snhu_15 = snhu_15.drop(index = ('Total'))
snhu_16 = snhu_16.drop(index = ('Total'))
snhu_17 = snhu_17.drop(index = ('Total'))
snhu_18 = snhu_18.drop(index = ('Total'))
snhu_19 = snhu_19.drop(index = ('Total'))
snhu_21 = snhu_21.drop(index = ('Total'))
snhu_22 = snhu_22.drop(index = ('Total'))
```


```python
#Combining all dataframes that we are analyzing 
frames_p = [snhu_15_p,snhu_16_p,snhu_17_p,snhu_18_p,snhu_19_p,snhu_21_p,snhu_22_p]
```


```python
#Concating pitchers data  
frames_p = pd.concat(frames_p)
```


```python
#Dropping repeat columns for the merge
#Data will be captured in the hitting data
frames_p = frames_p.drop(labels = ['Loc','Opponent','W/L', 'Score','Score.1'], axis = 1)

```


```python
#Combining all dataframes that we are analyzing 
frames_h = [snhu_15,snhu_16,snhu_17,snhu_18,snhu_19,snhu_21,snhu_22]
```


```python
#Combining all frames together
frames_h = pd.concat(frames_h)
```


```python
#Merging pitcher data and hitting data
df = pd.concat([frames_h,frames_p],axis = 1)
```


```python
#Final dataframe
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
      <th>Loc</th>
      <th>Opponent</th>
      <th>W/L</th>
      <th>Score</th>
      <th>AB</th>
      <th>R</th>
      <th>H</th>
      <th>RBI</th>
      <th>2B</th>
      <th>3B</th>
      <th>...</th>
      <th>3B</th>
      <th>HR</th>
      <th>WP</th>
      <th>BK</th>
      <th>HBP</th>
      <th>IBB</th>
      <th>W</th>
      <th>L</th>
      <th>SV</th>
      <th>ERA</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2/13/2015</th>
      <td>at</td>
      <td>Nova Southeastern</td>
      <td>L</td>
      <td>3-5</td>
      <td>35</td>
      <td>3</td>
      <td>8</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>4.50</td>
    </tr>
    <tr>
      <th>2/14/2015</th>
      <td>at</td>
      <td>Nova Southeastern</td>
      <td>L</td>
      <td>3-10</td>
      <td>35</td>
      <td>3</td>
      <td>8</td>
      <td>3</td>
      <td>2</td>
      <td>0</td>
      <td>...</td>
      <td>1</td>
      <td>2</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>6.75</td>
    </tr>
    <tr>
      <th>2/15/2015</th>
      <td>at</td>
      <td>Nova Southeastern</td>
      <td>L</td>
      <td>3-6</td>
      <td>32</td>
      <td>3</td>
      <td>5</td>
      <td>3</td>
      <td>1</td>
      <td>0</td>
      <td>...</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>3</td>
      <td>0</td>
      <td>4.50</td>
    </tr>
    <tr>
      <th>2/20/2015</th>
      <td>vs</td>
      <td>U. of Bridgeport</td>
      <td>W</td>
      <td>16-1</td>
      <td>35</td>
      <td>16</td>
      <td>12</td>
      <td>13</td>
      <td>0</td>
      <td>2</td>
      <td>...</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>3</td>
      <td>0</td>
      <td>1.00</td>
    </tr>
    <tr>
      <th>2/20/2015</th>
      <td>vs</td>
      <td>Felician</td>
      <td>W</td>
      <td>6-4</td>
      <td>36</td>
      <td>6</td>
      <td>9</td>
      <td>4</td>
      <td>2</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>3</td>
      <td>0</td>
      <td>3.00</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 41 columns</p>
</div>



Renaming the columns by index number. We have repeat coulmns that have the same title but different measures of team vs opponent.


```python
#Renaming the data from the pitchers table to have o = Opponent
df.columns.values[25] = 'oH'
df.columns.values[26] = "oR"
df.columns.values[28] ="oBB"
df.columns.values[30] ="o2B"
df.columns.values[31] = "o3B"
df.columns.values[32] ="oHR"
df.columns.values[35] ="oHBP"
df.columns.values[36] = "oIBB" 
```

# End of Collection and Concating


```python
#Splitting the score column 
df[['penman','opps']] = df['Score'].str.split('-', expand = True)
```


```python
#Changing type from Object to int
df.penman = df.penman.astype(int)
df.opps = df.opps.astype(int)
```


```python
#We will be analyzing 212 total games
df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Index: 364 entries, 2/13/2015 to 6/7/2022
    Data columns (total 43 columns):
     #   Column    Non-Null Count  Dtype  
    ---  ------    --------------  -----  
     0   Loc       240 non-null    object 
     1   Opponent  364 non-null    object 
     2   W/L       364 non-null    object 
     3   Score     364 non-null    object 
     4   AB        364 non-null    int64  
     5   R         364 non-null    int64  
     6   H         364 non-null    int64  
     7   RBI       364 non-null    int64  
     8   2B        364 non-null    int64  
     9   3B        364 non-null    int64  
     10  HR        364 non-null    int64  
     11  BB        364 non-null    int64  
     12  IBB       364 non-null    int64  
     13  SB        364 non-null    int64  
     14  CS        364 non-null    int64  
     15  HBP       364 non-null    int64  
     16  SH        364 non-null    int64  
     17  SF        364 non-null    int64  
     18  GDP       364 non-null    int64  
     19  K         364 non-null    int64  
     20  PO        364 non-null    int64  
     21  A         364 non-null    int64  
     22  E         364 non-null    int64  
     23  AVG       364 non-null    float64
     24  IP        364 non-null    float64
     25  oH        364 non-null    int64  
     26  oR        364 non-null    int64  
     27  ER        364 non-null    int64  
     28  oBB       364 non-null    int64  
     29  SO        364 non-null    int64  
     30  o2B       364 non-null    int64  
     31  o3B       364 non-null    int64  
     32  oHR       364 non-null    int64  
     33  WP        364 non-null    int64  
     34  BK        364 non-null    int64  
     35  oHBP      364 non-null    int64  
     36  oIBB      364 non-null    int64  
     37  W         364 non-null    int64  
     38  L         364 non-null    int64  
     39  SV        364 non-null    int64  
     40  ERA       364 non-null    float64
     41  penman    364 non-null    int64  
     42  opps      364 non-null    int64  
    dtypes: float64(3), int64(36), object(4)
    memory usage: 125.1+ KB



```python
#Top 5 obervations in df
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
      <th>Loc</th>
      <th>Opponent</th>
      <th>W/L</th>
      <th>Score</th>
      <th>AB</th>
      <th>R</th>
      <th>H</th>
      <th>RBI</th>
      <th>2B</th>
      <th>3B</th>
      <th>...</th>
      <th>WP</th>
      <th>BK</th>
      <th>oHBP</th>
      <th>oIBB</th>
      <th>W</th>
      <th>L</th>
      <th>SV</th>
      <th>ERA</th>
      <th>penman</th>
      <th>opps</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2/13/2015</th>
      <td>at</td>
      <td>Nova Southeastern</td>
      <td>L</td>
      <td>3-5</td>
      <td>35</td>
      <td>3</td>
      <td>8</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>4.50</td>
      <td>3</td>
      <td>5</td>
    </tr>
    <tr>
      <th>2/14/2015</th>
      <td>at</td>
      <td>Nova Southeastern</td>
      <td>L</td>
      <td>3-10</td>
      <td>35</td>
      <td>3</td>
      <td>8</td>
      <td>3</td>
      <td>2</td>
      <td>0</td>
      <td>...</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>6.75</td>
      <td>3</td>
      <td>10</td>
    </tr>
    <tr>
      <th>2/15/2015</th>
      <td>at</td>
      <td>Nova Southeastern</td>
      <td>L</td>
      <td>3-6</td>
      <td>32</td>
      <td>3</td>
      <td>5</td>
      <td>3</td>
      <td>1</td>
      <td>0</td>
      <td>...</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>3</td>
      <td>0</td>
      <td>4.50</td>
      <td>3</td>
      <td>6</td>
    </tr>
    <tr>
      <th>2/20/2015</th>
      <td>vs</td>
      <td>U. of Bridgeport</td>
      <td>W</td>
      <td>16-1</td>
      <td>35</td>
      <td>16</td>
      <td>12</td>
      <td>13</td>
      <td>0</td>
      <td>2</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>3</td>
      <td>0</td>
      <td>1.00</td>
      <td>16</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2/20/2015</th>
      <td>vs</td>
      <td>Felician</td>
      <td>W</td>
      <td>6-4</td>
      <td>36</td>
      <td>6</td>
      <td>9</td>
      <td>4</td>
      <td>2</td>
      <td>0</td>
      <td>...</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>3</td>
      <td>0</td>
      <td>3.00</td>
      <td>6</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 43 columns</p>
</div>




```python
#Bottom 5 observations in df
df.tail()
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
      <th>Loc</th>
      <th>Opponent</th>
      <th>W/L</th>
      <th>Score</th>
      <th>AB</th>
      <th>R</th>
      <th>H</th>
      <th>RBI</th>
      <th>2B</th>
      <th>3B</th>
      <th>...</th>
      <th>WP</th>
      <th>BK</th>
      <th>oHBP</th>
      <th>oIBB</th>
      <th>W</th>
      <th>L</th>
      <th>SV</th>
      <th>ERA</th>
      <th>penman</th>
      <th>opps</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>5/27/2022</th>
      <td>NaN</td>
      <td>Molloy</td>
      <td>L</td>
      <td>5-6</td>
      <td>33</td>
      <td>5</td>
      <td>6</td>
      <td>5</td>
      <td>2</td>
      <td>0</td>
      <td>...</td>
      <td>1</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>44</td>
      <td>10</td>
      <td>0</td>
      <td>4.00</td>
      <td>5</td>
      <td>6</td>
    </tr>
    <tr>
      <th>5/27/2022</th>
      <td>at</td>
      <td>Molloy</td>
      <td>W</td>
      <td>7-5</td>
      <td>33</td>
      <td>7</td>
      <td>10</td>
      <td>6</td>
      <td>4</td>
      <td>1</td>
      <td>...</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>45</td>
      <td>10</td>
      <td>0</td>
      <td>2.00</td>
      <td>7</td>
      <td>5</td>
    </tr>
    <tr>
      <th>5/29/2022</th>
      <td>vs</td>
      <td>Molloy</td>
      <td>W</td>
      <td>7-3</td>
      <td>35</td>
      <td>7</td>
      <td>9</td>
      <td>6</td>
      <td>3</td>
      <td>1</td>
      <td>...</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>46</td>
      <td>10</td>
      <td>0</td>
      <td>2.00</td>
      <td>7</td>
      <td>3</td>
    </tr>
    <tr>
      <th>6/5/2022</th>
      <td>vs</td>
      <td>Angelo State</td>
      <td>L</td>
      <td>4-7</td>
      <td>34</td>
      <td>4</td>
      <td>7</td>
      <td>4</td>
      <td>1</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>46</td>
      <td>11</td>
      <td>0</td>
      <td>5.00</td>
      <td>4</td>
      <td>7</td>
    </tr>
    <tr>
      <th>6/7/2022</th>
      <td>vs</td>
      <td>West Chester</td>
      <td>L</td>
      <td>3-7</td>
      <td>34</td>
      <td>3</td>
      <td>10</td>
      <td>3</td>
      <td>0</td>
      <td>1</td>
      <td>...</td>
      <td>1</td>
      <td>1</td>
      <td>6</td>
      <td>0</td>
      <td>46</td>
      <td>12</td>
      <td>0</td>
      <td>7.88</td>
      <td>3</td>
      <td>7</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 43 columns</p>
</div>



# EDA 


```python
#Changing the box plot size and layout
plt.rcParams["figure.figsize"] = [12, 6]
plt.rcParams["figure.autolayout"] = True
```


```python
#Looking at the central tendency
df.describe()
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
      <th>AB</th>
      <th>R</th>
      <th>H</th>
      <th>RBI</th>
      <th>2B</th>
      <th>3B</th>
      <th>HR</th>
      <th>BB</th>
      <th>IBB</th>
      <th>SB</th>
      <th>...</th>
      <th>WP</th>
      <th>BK</th>
      <th>oHBP</th>
      <th>oIBB</th>
      <th>W</th>
      <th>L</th>
      <th>SV</th>
      <th>ERA</th>
      <th>penman</th>
      <th>opps</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>364.000000</td>
      <td>364.000000</td>
      <td>364.000000</td>
      <td>364.000000</td>
      <td>364.000000</td>
      <td>364.000000</td>
      <td>364.000000</td>
      <td>364.000000</td>
      <td>364.000000</td>
      <td>364.000000</td>
      <td>...</td>
      <td>364.000000</td>
      <td>364.000000</td>
      <td>364.000000</td>
      <td>364.000000</td>
      <td>364.000000</td>
      <td>364.000000</td>
      <td>364.000000</td>
      <td>364.000000</td>
      <td>364.000000</td>
      <td>364.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>34.219780</td>
      <td>7.365385</td>
      <td>9.917582</td>
      <td>6.373626</td>
      <td>1.824176</td>
      <td>0.387363</td>
      <td>0.923077</td>
      <td>4.519231</td>
      <td>0.065934</td>
      <td>2.217033</td>
      <td>...</td>
      <td>0.881868</td>
      <td>0.115385</td>
      <td>0.689560</td>
      <td>0.104396</td>
      <td>21.206044</td>
      <td>5.821429</td>
      <td>0.087912</td>
      <td>3.077582</td>
      <td>7.365385</td>
      <td>3.725275</td>
    </tr>
    <tr>
      <th>std</th>
      <td>5.724246</td>
      <td>4.836843</td>
      <td>4.248976</td>
      <td>4.419836</td>
      <td>1.608130</td>
      <td>0.701006</td>
      <td>1.103271</td>
      <td>2.813224</td>
      <td>0.259357</td>
      <td>2.104602</td>
      <td>...</td>
      <td>1.194478</td>
      <td>0.396798</td>
      <td>0.950453</td>
      <td>0.348286</td>
      <td>12.899009</td>
      <td>3.954769</td>
      <td>0.283557</td>
      <td>2.951806</td>
      <td>4.836843</td>
      <td>3.160765</td>
    </tr>
    <tr>
      <th>min</th>
      <td>13.000000</td>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>...</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>31.000000</td>
      <td>3.000000</td>
      <td>7.000000</td>
      <td>3.000000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>3.000000</td>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>...</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>10.000000</td>
      <td>3.000000</td>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>3.000000</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>34.000000</td>
      <td>7.000000</td>
      <td>10.000000</td>
      <td>5.000000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>4.000000</td>
      <td>0.000000</td>
      <td>2.000000</td>
      <td>...</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>20.000000</td>
      <td>6.000000</td>
      <td>0.000000</td>
      <td>2.410000</td>
      <td>7.000000</td>
      <td>3.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>38.000000</td>
      <td>10.000000</td>
      <td>13.000000</td>
      <td>9.000000</td>
      <td>3.000000</td>
      <td>1.000000</td>
      <td>2.000000</td>
      <td>6.000000</td>
      <td>0.000000</td>
      <td>3.000000</td>
      <td>...</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>31.000000</td>
      <td>8.000000</td>
      <td>0.000000</td>
      <td>4.500000</td>
      <td>10.000000</td>
      <td>5.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>56.000000</td>
      <td>26.000000</td>
      <td>24.000000</td>
      <td>25.000000</td>
      <td>8.000000</td>
      <td>5.000000</td>
      <td>5.000000</td>
      <td>18.000000</td>
      <td>2.000000</td>
      <td>12.000000</td>
      <td>...</td>
      <td>9.000000</td>
      <td>3.000000</td>
      <td>6.000000</td>
      <td>3.000000</td>
      <td>50.000000</td>
      <td>17.000000</td>
      <td>1.000000</td>
      <td>20.570000</td>
      <td>26.000000</td>
      <td>19.000000</td>
    </tr>
  </tbody>
</table>
<p>8 rows × 39 columns</p>
</div>




```python
#How much Penman outscore oppenant by 
print("On average the Penman outscore their opponent by ", df.penman.mean() - df.opps.mean(), "runs")
```

    On average the Penman outscore their opponent by  3.6401098901098896 runs



```python
#Box plot of pitchers data 
df.boxplot(column = ['SO','oH','oR','IP'])
plt.title('Pitchers Stats')
```




    Text(0.5, 1.0, 'Pitchers Stats')




    
![png](output_27_1.png)
    



```python
#Box plot of pitchers data 
df.boxplot(column = ['o2B','oHR','oHBP','oIBB','IP','o3B','WP'])
plt.title('Pitchers Stats')
```




    Text(0.5, 1.0, 'Pitchers Stats')




    
![png](output_28_1.png)
    



```python
#Box plot of the majority of the columns for hitters 
df.boxplot(column = ['R','H','RBI','2B','HR','BB','IBB','SB','HBP','GDP','K','penman','opps'])
plt.title("Hitting Stats")
```




    Text(0.5, 1.0, 'Hitting Stats')




    
![png](output_29_1.png)
    



```python
#Box plot of the majority of the columns for hitters 
df.boxplot(column = ['AB','K','PO','A','E'])
plt.title("Hitting Stats")
```




    Text(0.5, 1.0, 'Hitting Stats')




    
![png](output_30_1.png)
    



```python
#Seperating BA according to scale 
#Value too small to display with other data
df.boxplot(column= ['AVG']) 
plt.title("SNHU vs Opps AVG")
```




    Text(0.5, 1.0, 'SNHU vs Opps AVG')




    
![png](output_31_1.png)
    



```python
#Seperating '3B', 'SH', 'SF' according to scale 
df.boxplot(column = ['3B', 'SH', 'SF'])
plt.title('Hitters Stats')
```




    Text(0.5, 1.0, 'Hitters Stats')




    
![png](output_32_1.png)
    



```python
#Pearson correlation matrix examing r^2
matrix = df.corr(
    method = 'pearson',  # The method of correlation
    min_periods = 1      # Min number of observations required
)
```


```python
#Changing the size of the correlation matrix
sns.set(rc={'figure.figsize':(30,16)})

```


```python
#Correlation Heat Map 
matrix = df.corr().round(2)
sns.heatmap(matrix, annot=True)
plt.show()
```


    
![png](output_35_0.png)
    


# Highly Correlated Variables


```python
#Highly correlated variable subset 
variables = df[['3B','HR', 'BB', 'SB','SF','PO','E','oBB','o2B','oHR','oH','oR', 'ER','ERA', 'opps', 'penman', 'AVG', 'AB', 'R','H', 'RBI']]
```


```python
#Variables showing higher R squared values 
var = variables.corr().round(2)
sns.heatmap(var, annot=True)
plt.show()
```


    
![png](output_38_0.png)
    



```python
#Scatter matrix
pd.plotting.scatter_matrix(variables, figsize = (40,40))
```




    array([[<AxesSubplot:xlabel='3B', ylabel='3B'>,
            <AxesSubplot:xlabel='HR', ylabel='3B'>,
            <AxesSubplot:xlabel='BB', ylabel='3B'>,
            <AxesSubplot:xlabel='SB', ylabel='3B'>,
            <AxesSubplot:xlabel='SF', ylabel='3B'>,
            <AxesSubplot:xlabel='PO', ylabel='3B'>,
            <AxesSubplot:xlabel='E', ylabel='3B'>,
            <AxesSubplot:xlabel='oBB', ylabel='3B'>,
            <AxesSubplot:xlabel='o2B', ylabel='3B'>,
            <AxesSubplot:xlabel='oHR', ylabel='3B'>,
            <AxesSubplot:xlabel='oH', ylabel='3B'>,
            <AxesSubplot:xlabel='oR', ylabel='3B'>,
            <AxesSubplot:xlabel='ER', ylabel='3B'>,
            <AxesSubplot:xlabel='ERA', ylabel='3B'>,
            <AxesSubplot:xlabel='opps', ylabel='3B'>,
            <AxesSubplot:xlabel='penman', ylabel='3B'>,
            <AxesSubplot:xlabel='AVG', ylabel='3B'>,
            <AxesSubplot:xlabel='AB', ylabel='3B'>,
            <AxesSubplot:xlabel='R', ylabel='3B'>,
            <AxesSubplot:xlabel='H', ylabel='3B'>,
            <AxesSubplot:xlabel='RBI', ylabel='3B'>],
           [<AxesSubplot:xlabel='3B', ylabel='HR'>,
            <AxesSubplot:xlabel='HR', ylabel='HR'>,
            <AxesSubplot:xlabel='BB', ylabel='HR'>,
            <AxesSubplot:xlabel='SB', ylabel='HR'>,
            <AxesSubplot:xlabel='SF', ylabel='HR'>,
            <AxesSubplot:xlabel='PO', ylabel='HR'>,
            <AxesSubplot:xlabel='E', ylabel='HR'>,
            <AxesSubplot:xlabel='oBB', ylabel='HR'>,
            <AxesSubplot:xlabel='o2B', ylabel='HR'>,
            <AxesSubplot:xlabel='oHR', ylabel='HR'>,
            <AxesSubplot:xlabel='oH', ylabel='HR'>,
            <AxesSubplot:xlabel='oR', ylabel='HR'>,
            <AxesSubplot:xlabel='ER', ylabel='HR'>,
            <AxesSubplot:xlabel='ERA', ylabel='HR'>,
            <AxesSubplot:xlabel='opps', ylabel='HR'>,
            <AxesSubplot:xlabel='penman', ylabel='HR'>,
            <AxesSubplot:xlabel='AVG', ylabel='HR'>,
            <AxesSubplot:xlabel='AB', ylabel='HR'>,
            <AxesSubplot:xlabel='R', ylabel='HR'>,
            <AxesSubplot:xlabel='H', ylabel='HR'>,
            <AxesSubplot:xlabel='RBI', ylabel='HR'>],
           [<AxesSubplot:xlabel='3B', ylabel='BB'>,
            <AxesSubplot:xlabel='HR', ylabel='BB'>,
            <AxesSubplot:xlabel='BB', ylabel='BB'>,
            <AxesSubplot:xlabel='SB', ylabel='BB'>,
            <AxesSubplot:xlabel='SF', ylabel='BB'>,
            <AxesSubplot:xlabel='PO', ylabel='BB'>,
            <AxesSubplot:xlabel='E', ylabel='BB'>,
            <AxesSubplot:xlabel='oBB', ylabel='BB'>,
            <AxesSubplot:xlabel='o2B', ylabel='BB'>,
            <AxesSubplot:xlabel='oHR', ylabel='BB'>,
            <AxesSubplot:xlabel='oH', ylabel='BB'>,
            <AxesSubplot:xlabel='oR', ylabel='BB'>,
            <AxesSubplot:xlabel='ER', ylabel='BB'>,
            <AxesSubplot:xlabel='ERA', ylabel='BB'>,
            <AxesSubplot:xlabel='opps', ylabel='BB'>,
            <AxesSubplot:xlabel='penman', ylabel='BB'>,
            <AxesSubplot:xlabel='AVG', ylabel='BB'>,
            <AxesSubplot:xlabel='AB', ylabel='BB'>,
            <AxesSubplot:xlabel='R', ylabel='BB'>,
            <AxesSubplot:xlabel='H', ylabel='BB'>,
            <AxesSubplot:xlabel='RBI', ylabel='BB'>],
           [<AxesSubplot:xlabel='3B', ylabel='SB'>,
            <AxesSubplot:xlabel='HR', ylabel='SB'>,
            <AxesSubplot:xlabel='BB', ylabel='SB'>,
            <AxesSubplot:xlabel='SB', ylabel='SB'>,
            <AxesSubplot:xlabel='SF', ylabel='SB'>,
            <AxesSubplot:xlabel='PO', ylabel='SB'>,
            <AxesSubplot:xlabel='E', ylabel='SB'>,
            <AxesSubplot:xlabel='oBB', ylabel='SB'>,
            <AxesSubplot:xlabel='o2B', ylabel='SB'>,
            <AxesSubplot:xlabel='oHR', ylabel='SB'>,
            <AxesSubplot:xlabel='oH', ylabel='SB'>,
            <AxesSubplot:xlabel='oR', ylabel='SB'>,
            <AxesSubplot:xlabel='ER', ylabel='SB'>,
            <AxesSubplot:xlabel='ERA', ylabel='SB'>,
            <AxesSubplot:xlabel='opps', ylabel='SB'>,
            <AxesSubplot:xlabel='penman', ylabel='SB'>,
            <AxesSubplot:xlabel='AVG', ylabel='SB'>,
            <AxesSubplot:xlabel='AB', ylabel='SB'>,
            <AxesSubplot:xlabel='R', ylabel='SB'>,
            <AxesSubplot:xlabel='H', ylabel='SB'>,
            <AxesSubplot:xlabel='RBI', ylabel='SB'>],
           [<AxesSubplot:xlabel='3B', ylabel='SF'>,
            <AxesSubplot:xlabel='HR', ylabel='SF'>,
            <AxesSubplot:xlabel='BB', ylabel='SF'>,
            <AxesSubplot:xlabel='SB', ylabel='SF'>,
            <AxesSubplot:xlabel='SF', ylabel='SF'>,
            <AxesSubplot:xlabel='PO', ylabel='SF'>,
            <AxesSubplot:xlabel='E', ylabel='SF'>,
            <AxesSubplot:xlabel='oBB', ylabel='SF'>,
            <AxesSubplot:xlabel='o2B', ylabel='SF'>,
            <AxesSubplot:xlabel='oHR', ylabel='SF'>,
            <AxesSubplot:xlabel='oH', ylabel='SF'>,
            <AxesSubplot:xlabel='oR', ylabel='SF'>,
            <AxesSubplot:xlabel='ER', ylabel='SF'>,
            <AxesSubplot:xlabel='ERA', ylabel='SF'>,
            <AxesSubplot:xlabel='opps', ylabel='SF'>,
            <AxesSubplot:xlabel='penman', ylabel='SF'>,
            <AxesSubplot:xlabel='AVG', ylabel='SF'>,
            <AxesSubplot:xlabel='AB', ylabel='SF'>,
            <AxesSubplot:xlabel='R', ylabel='SF'>,
            <AxesSubplot:xlabel='H', ylabel='SF'>,
            <AxesSubplot:xlabel='RBI', ylabel='SF'>],
           [<AxesSubplot:xlabel='3B', ylabel='PO'>,
            <AxesSubplot:xlabel='HR', ylabel='PO'>,
            <AxesSubplot:xlabel='BB', ylabel='PO'>,
            <AxesSubplot:xlabel='SB', ylabel='PO'>,
            <AxesSubplot:xlabel='SF', ylabel='PO'>,
            <AxesSubplot:xlabel='PO', ylabel='PO'>,
            <AxesSubplot:xlabel='E', ylabel='PO'>,
            <AxesSubplot:xlabel='oBB', ylabel='PO'>,
            <AxesSubplot:xlabel='o2B', ylabel='PO'>,
            <AxesSubplot:xlabel='oHR', ylabel='PO'>,
            <AxesSubplot:xlabel='oH', ylabel='PO'>,
            <AxesSubplot:xlabel='oR', ylabel='PO'>,
            <AxesSubplot:xlabel='ER', ylabel='PO'>,
            <AxesSubplot:xlabel='ERA', ylabel='PO'>,
            <AxesSubplot:xlabel='opps', ylabel='PO'>,
            <AxesSubplot:xlabel='penman', ylabel='PO'>,
            <AxesSubplot:xlabel='AVG', ylabel='PO'>,
            <AxesSubplot:xlabel='AB', ylabel='PO'>,
            <AxesSubplot:xlabel='R', ylabel='PO'>,
            <AxesSubplot:xlabel='H', ylabel='PO'>,
            <AxesSubplot:xlabel='RBI', ylabel='PO'>],
           [<AxesSubplot:xlabel='3B', ylabel='E'>,
            <AxesSubplot:xlabel='HR', ylabel='E'>,
            <AxesSubplot:xlabel='BB', ylabel='E'>,
            <AxesSubplot:xlabel='SB', ylabel='E'>,
            <AxesSubplot:xlabel='SF', ylabel='E'>,
            <AxesSubplot:xlabel='PO', ylabel='E'>,
            <AxesSubplot:xlabel='E', ylabel='E'>,
            <AxesSubplot:xlabel='oBB', ylabel='E'>,
            <AxesSubplot:xlabel='o2B', ylabel='E'>,
            <AxesSubplot:xlabel='oHR', ylabel='E'>,
            <AxesSubplot:xlabel='oH', ylabel='E'>,
            <AxesSubplot:xlabel='oR', ylabel='E'>,
            <AxesSubplot:xlabel='ER', ylabel='E'>,
            <AxesSubplot:xlabel='ERA', ylabel='E'>,
            <AxesSubplot:xlabel='opps', ylabel='E'>,
            <AxesSubplot:xlabel='penman', ylabel='E'>,
            <AxesSubplot:xlabel='AVG', ylabel='E'>,
            <AxesSubplot:xlabel='AB', ylabel='E'>,
            <AxesSubplot:xlabel='R', ylabel='E'>,
            <AxesSubplot:xlabel='H', ylabel='E'>,
            <AxesSubplot:xlabel='RBI', ylabel='E'>],
           [<AxesSubplot:xlabel='3B', ylabel='oBB'>,
            <AxesSubplot:xlabel='HR', ylabel='oBB'>,
            <AxesSubplot:xlabel='BB', ylabel='oBB'>,
            <AxesSubplot:xlabel='SB', ylabel='oBB'>,
            <AxesSubplot:xlabel='SF', ylabel='oBB'>,
            <AxesSubplot:xlabel='PO', ylabel='oBB'>,
            <AxesSubplot:xlabel='E', ylabel='oBB'>,
            <AxesSubplot:xlabel='oBB', ylabel='oBB'>,
            <AxesSubplot:xlabel='o2B', ylabel='oBB'>,
            <AxesSubplot:xlabel='oHR', ylabel='oBB'>,
            <AxesSubplot:xlabel='oH', ylabel='oBB'>,
            <AxesSubplot:xlabel='oR', ylabel='oBB'>,
            <AxesSubplot:xlabel='ER', ylabel='oBB'>,
            <AxesSubplot:xlabel='ERA', ylabel='oBB'>,
            <AxesSubplot:xlabel='opps', ylabel='oBB'>,
            <AxesSubplot:xlabel='penman', ylabel='oBB'>,
            <AxesSubplot:xlabel='AVG', ylabel='oBB'>,
            <AxesSubplot:xlabel='AB', ylabel='oBB'>,
            <AxesSubplot:xlabel='R', ylabel='oBB'>,
            <AxesSubplot:xlabel='H', ylabel='oBB'>,
            <AxesSubplot:xlabel='RBI', ylabel='oBB'>],
           [<AxesSubplot:xlabel='3B', ylabel='o2B'>,
            <AxesSubplot:xlabel='HR', ylabel='o2B'>,
            <AxesSubplot:xlabel='BB', ylabel='o2B'>,
            <AxesSubplot:xlabel='SB', ylabel='o2B'>,
            <AxesSubplot:xlabel='SF', ylabel='o2B'>,
            <AxesSubplot:xlabel='PO', ylabel='o2B'>,
            <AxesSubplot:xlabel='E', ylabel='o2B'>,
            <AxesSubplot:xlabel='oBB', ylabel='o2B'>,
            <AxesSubplot:xlabel='o2B', ylabel='o2B'>,
            <AxesSubplot:xlabel='oHR', ylabel='o2B'>,
            <AxesSubplot:xlabel='oH', ylabel='o2B'>,
            <AxesSubplot:xlabel='oR', ylabel='o2B'>,
            <AxesSubplot:xlabel='ER', ylabel='o2B'>,
            <AxesSubplot:xlabel='ERA', ylabel='o2B'>,
            <AxesSubplot:xlabel='opps', ylabel='o2B'>,
            <AxesSubplot:xlabel='penman', ylabel='o2B'>,
            <AxesSubplot:xlabel='AVG', ylabel='o2B'>,
            <AxesSubplot:xlabel='AB', ylabel='o2B'>,
            <AxesSubplot:xlabel='R', ylabel='o2B'>,
            <AxesSubplot:xlabel='H', ylabel='o2B'>,
            <AxesSubplot:xlabel='RBI', ylabel='o2B'>],
           [<AxesSubplot:xlabel='3B', ylabel='oHR'>,
            <AxesSubplot:xlabel='HR', ylabel='oHR'>,
            <AxesSubplot:xlabel='BB', ylabel='oHR'>,
            <AxesSubplot:xlabel='SB', ylabel='oHR'>,
            <AxesSubplot:xlabel='SF', ylabel='oHR'>,
            <AxesSubplot:xlabel='PO', ylabel='oHR'>,
            <AxesSubplot:xlabel='E', ylabel='oHR'>,
            <AxesSubplot:xlabel='oBB', ylabel='oHR'>,
            <AxesSubplot:xlabel='o2B', ylabel='oHR'>,
            <AxesSubplot:xlabel='oHR', ylabel='oHR'>,
            <AxesSubplot:xlabel='oH', ylabel='oHR'>,
            <AxesSubplot:xlabel='oR', ylabel='oHR'>,
            <AxesSubplot:xlabel='ER', ylabel='oHR'>,
            <AxesSubplot:xlabel='ERA', ylabel='oHR'>,
            <AxesSubplot:xlabel='opps', ylabel='oHR'>,
            <AxesSubplot:xlabel='penman', ylabel='oHR'>,
            <AxesSubplot:xlabel='AVG', ylabel='oHR'>,
            <AxesSubplot:xlabel='AB', ylabel='oHR'>,
            <AxesSubplot:xlabel='R', ylabel='oHR'>,
            <AxesSubplot:xlabel='H', ylabel='oHR'>,
            <AxesSubplot:xlabel='RBI', ylabel='oHR'>],
           [<AxesSubplot:xlabel='3B', ylabel='oH'>,
            <AxesSubplot:xlabel='HR', ylabel='oH'>,
            <AxesSubplot:xlabel='BB', ylabel='oH'>,
            <AxesSubplot:xlabel='SB', ylabel='oH'>,
            <AxesSubplot:xlabel='SF', ylabel='oH'>,
            <AxesSubplot:xlabel='PO', ylabel='oH'>,
            <AxesSubplot:xlabel='E', ylabel='oH'>,
            <AxesSubplot:xlabel='oBB', ylabel='oH'>,
            <AxesSubplot:xlabel='o2B', ylabel='oH'>,
            <AxesSubplot:xlabel='oHR', ylabel='oH'>,
            <AxesSubplot:xlabel='oH', ylabel='oH'>,
            <AxesSubplot:xlabel='oR', ylabel='oH'>,
            <AxesSubplot:xlabel='ER', ylabel='oH'>,
            <AxesSubplot:xlabel='ERA', ylabel='oH'>,
            <AxesSubplot:xlabel='opps', ylabel='oH'>,
            <AxesSubplot:xlabel='penman', ylabel='oH'>,
            <AxesSubplot:xlabel='AVG', ylabel='oH'>,
            <AxesSubplot:xlabel='AB', ylabel='oH'>,
            <AxesSubplot:xlabel='R', ylabel='oH'>,
            <AxesSubplot:xlabel='H', ylabel='oH'>,
            <AxesSubplot:xlabel='RBI', ylabel='oH'>],
           [<AxesSubplot:xlabel='3B', ylabel='oR'>,
            <AxesSubplot:xlabel='HR', ylabel='oR'>,
            <AxesSubplot:xlabel='BB', ylabel='oR'>,
            <AxesSubplot:xlabel='SB', ylabel='oR'>,
            <AxesSubplot:xlabel='SF', ylabel='oR'>,
            <AxesSubplot:xlabel='PO', ylabel='oR'>,
            <AxesSubplot:xlabel='E', ylabel='oR'>,
            <AxesSubplot:xlabel='oBB', ylabel='oR'>,
            <AxesSubplot:xlabel='o2B', ylabel='oR'>,
            <AxesSubplot:xlabel='oHR', ylabel='oR'>,
            <AxesSubplot:xlabel='oH', ylabel='oR'>,
            <AxesSubplot:xlabel='oR', ylabel='oR'>,
            <AxesSubplot:xlabel='ER', ylabel='oR'>,
            <AxesSubplot:xlabel='ERA', ylabel='oR'>,
            <AxesSubplot:xlabel='opps', ylabel='oR'>,
            <AxesSubplot:xlabel='penman', ylabel='oR'>,
            <AxesSubplot:xlabel='AVG', ylabel='oR'>,
            <AxesSubplot:xlabel='AB', ylabel='oR'>,
            <AxesSubplot:xlabel='R', ylabel='oR'>,
            <AxesSubplot:xlabel='H', ylabel='oR'>,
            <AxesSubplot:xlabel='RBI', ylabel='oR'>],
           [<AxesSubplot:xlabel='3B', ylabel='ER'>,
            <AxesSubplot:xlabel='HR', ylabel='ER'>,
            <AxesSubplot:xlabel='BB', ylabel='ER'>,
            <AxesSubplot:xlabel='SB', ylabel='ER'>,
            <AxesSubplot:xlabel='SF', ylabel='ER'>,
            <AxesSubplot:xlabel='PO', ylabel='ER'>,
            <AxesSubplot:xlabel='E', ylabel='ER'>,
            <AxesSubplot:xlabel='oBB', ylabel='ER'>,
            <AxesSubplot:xlabel='o2B', ylabel='ER'>,
            <AxesSubplot:xlabel='oHR', ylabel='ER'>,
            <AxesSubplot:xlabel='oH', ylabel='ER'>,
            <AxesSubplot:xlabel='oR', ylabel='ER'>,
            <AxesSubplot:xlabel='ER', ylabel='ER'>,
            <AxesSubplot:xlabel='ERA', ylabel='ER'>,
            <AxesSubplot:xlabel='opps', ylabel='ER'>,
            <AxesSubplot:xlabel='penman', ylabel='ER'>,
            <AxesSubplot:xlabel='AVG', ylabel='ER'>,
            <AxesSubplot:xlabel='AB', ylabel='ER'>,
            <AxesSubplot:xlabel='R', ylabel='ER'>,
            <AxesSubplot:xlabel='H', ylabel='ER'>,
            <AxesSubplot:xlabel='RBI', ylabel='ER'>],
           [<AxesSubplot:xlabel='3B', ylabel='ERA'>,
            <AxesSubplot:xlabel='HR', ylabel='ERA'>,
            <AxesSubplot:xlabel='BB', ylabel='ERA'>,
            <AxesSubplot:xlabel='SB', ylabel='ERA'>,
            <AxesSubplot:xlabel='SF', ylabel='ERA'>,
            <AxesSubplot:xlabel='PO', ylabel='ERA'>,
            <AxesSubplot:xlabel='E', ylabel='ERA'>,
            <AxesSubplot:xlabel='oBB', ylabel='ERA'>,
            <AxesSubplot:xlabel='o2B', ylabel='ERA'>,
            <AxesSubplot:xlabel='oHR', ylabel='ERA'>,
            <AxesSubplot:xlabel='oH', ylabel='ERA'>,
            <AxesSubplot:xlabel='oR', ylabel='ERA'>,
            <AxesSubplot:xlabel='ER', ylabel='ERA'>,
            <AxesSubplot:xlabel='ERA', ylabel='ERA'>,
            <AxesSubplot:xlabel='opps', ylabel='ERA'>,
            <AxesSubplot:xlabel='penman', ylabel='ERA'>,
            <AxesSubplot:xlabel='AVG', ylabel='ERA'>,
            <AxesSubplot:xlabel='AB', ylabel='ERA'>,
            <AxesSubplot:xlabel='R', ylabel='ERA'>,
            <AxesSubplot:xlabel='H', ylabel='ERA'>,
            <AxesSubplot:xlabel='RBI', ylabel='ERA'>],
           [<AxesSubplot:xlabel='3B', ylabel='opps'>,
            <AxesSubplot:xlabel='HR', ylabel='opps'>,
            <AxesSubplot:xlabel='BB', ylabel='opps'>,
            <AxesSubplot:xlabel='SB', ylabel='opps'>,
            <AxesSubplot:xlabel='SF', ylabel='opps'>,
            <AxesSubplot:xlabel='PO', ylabel='opps'>,
            <AxesSubplot:xlabel='E', ylabel='opps'>,
            <AxesSubplot:xlabel='oBB', ylabel='opps'>,
            <AxesSubplot:xlabel='o2B', ylabel='opps'>,
            <AxesSubplot:xlabel='oHR', ylabel='opps'>,
            <AxesSubplot:xlabel='oH', ylabel='opps'>,
            <AxesSubplot:xlabel='oR', ylabel='opps'>,
            <AxesSubplot:xlabel='ER', ylabel='opps'>,
            <AxesSubplot:xlabel='ERA', ylabel='opps'>,
            <AxesSubplot:xlabel='opps', ylabel='opps'>,
            <AxesSubplot:xlabel='penman', ylabel='opps'>,
            <AxesSubplot:xlabel='AVG', ylabel='opps'>,
            <AxesSubplot:xlabel='AB', ylabel='opps'>,
            <AxesSubplot:xlabel='R', ylabel='opps'>,
            <AxesSubplot:xlabel='H', ylabel='opps'>,
            <AxesSubplot:xlabel='RBI', ylabel='opps'>],
           [<AxesSubplot:xlabel='3B', ylabel='penman'>,
            <AxesSubplot:xlabel='HR', ylabel='penman'>,
            <AxesSubplot:xlabel='BB', ylabel='penman'>,
            <AxesSubplot:xlabel='SB', ylabel='penman'>,
            <AxesSubplot:xlabel='SF', ylabel='penman'>,
            <AxesSubplot:xlabel='PO', ylabel='penman'>,
            <AxesSubplot:xlabel='E', ylabel='penman'>,
            <AxesSubplot:xlabel='oBB', ylabel='penman'>,
            <AxesSubplot:xlabel='o2B', ylabel='penman'>,
            <AxesSubplot:xlabel='oHR', ylabel='penman'>,
            <AxesSubplot:xlabel='oH', ylabel='penman'>,
            <AxesSubplot:xlabel='oR', ylabel='penman'>,
            <AxesSubplot:xlabel='ER', ylabel='penman'>,
            <AxesSubplot:xlabel='ERA', ylabel='penman'>,
            <AxesSubplot:xlabel='opps', ylabel='penman'>,
            <AxesSubplot:xlabel='penman', ylabel='penman'>,
            <AxesSubplot:xlabel='AVG', ylabel='penman'>,
            <AxesSubplot:xlabel='AB', ylabel='penman'>,
            <AxesSubplot:xlabel='R', ylabel='penman'>,
            <AxesSubplot:xlabel='H', ylabel='penman'>,
            <AxesSubplot:xlabel='RBI', ylabel='penman'>],
           [<AxesSubplot:xlabel='3B', ylabel='AVG'>,
            <AxesSubplot:xlabel='HR', ylabel='AVG'>,
            <AxesSubplot:xlabel='BB', ylabel='AVG'>,
            <AxesSubplot:xlabel='SB', ylabel='AVG'>,
            <AxesSubplot:xlabel='SF', ylabel='AVG'>,
            <AxesSubplot:xlabel='PO', ylabel='AVG'>,
            <AxesSubplot:xlabel='E', ylabel='AVG'>,
            <AxesSubplot:xlabel='oBB', ylabel='AVG'>,
            <AxesSubplot:xlabel='o2B', ylabel='AVG'>,
            <AxesSubplot:xlabel='oHR', ylabel='AVG'>,
            <AxesSubplot:xlabel='oH', ylabel='AVG'>,
            <AxesSubplot:xlabel='oR', ylabel='AVG'>,
            <AxesSubplot:xlabel='ER', ylabel='AVG'>,
            <AxesSubplot:xlabel='ERA', ylabel='AVG'>,
            <AxesSubplot:xlabel='opps', ylabel='AVG'>,
            <AxesSubplot:xlabel='penman', ylabel='AVG'>,
            <AxesSubplot:xlabel='AVG', ylabel='AVG'>,
            <AxesSubplot:xlabel='AB', ylabel='AVG'>,
            <AxesSubplot:xlabel='R', ylabel='AVG'>,
            <AxesSubplot:xlabel='H', ylabel='AVG'>,
            <AxesSubplot:xlabel='RBI', ylabel='AVG'>],
           [<AxesSubplot:xlabel='3B', ylabel='AB'>,
            <AxesSubplot:xlabel='HR', ylabel='AB'>,
            <AxesSubplot:xlabel='BB', ylabel='AB'>,
            <AxesSubplot:xlabel='SB', ylabel='AB'>,
            <AxesSubplot:xlabel='SF', ylabel='AB'>,
            <AxesSubplot:xlabel='PO', ylabel='AB'>,
            <AxesSubplot:xlabel='E', ylabel='AB'>,
            <AxesSubplot:xlabel='oBB', ylabel='AB'>,
            <AxesSubplot:xlabel='o2B', ylabel='AB'>,
            <AxesSubplot:xlabel='oHR', ylabel='AB'>,
            <AxesSubplot:xlabel='oH', ylabel='AB'>,
            <AxesSubplot:xlabel='oR', ylabel='AB'>,
            <AxesSubplot:xlabel='ER', ylabel='AB'>,
            <AxesSubplot:xlabel='ERA', ylabel='AB'>,
            <AxesSubplot:xlabel='opps', ylabel='AB'>,
            <AxesSubplot:xlabel='penman', ylabel='AB'>,
            <AxesSubplot:xlabel='AVG', ylabel='AB'>,
            <AxesSubplot:xlabel='AB', ylabel='AB'>,
            <AxesSubplot:xlabel='R', ylabel='AB'>,
            <AxesSubplot:xlabel='H', ylabel='AB'>,
            <AxesSubplot:xlabel='RBI', ylabel='AB'>],
           [<AxesSubplot:xlabel='3B', ylabel='R'>,
            <AxesSubplot:xlabel='HR', ylabel='R'>,
            <AxesSubplot:xlabel='BB', ylabel='R'>,
            <AxesSubplot:xlabel='SB', ylabel='R'>,
            <AxesSubplot:xlabel='SF', ylabel='R'>,
            <AxesSubplot:xlabel='PO', ylabel='R'>,
            <AxesSubplot:xlabel='E', ylabel='R'>,
            <AxesSubplot:xlabel='oBB', ylabel='R'>,
            <AxesSubplot:xlabel='o2B', ylabel='R'>,
            <AxesSubplot:xlabel='oHR', ylabel='R'>,
            <AxesSubplot:xlabel='oH', ylabel='R'>,
            <AxesSubplot:xlabel='oR', ylabel='R'>,
            <AxesSubplot:xlabel='ER', ylabel='R'>,
            <AxesSubplot:xlabel='ERA', ylabel='R'>,
            <AxesSubplot:xlabel='opps', ylabel='R'>,
            <AxesSubplot:xlabel='penman', ylabel='R'>,
            <AxesSubplot:xlabel='AVG', ylabel='R'>,
            <AxesSubplot:xlabel='AB', ylabel='R'>,
            <AxesSubplot:xlabel='R', ylabel='R'>,
            <AxesSubplot:xlabel='H', ylabel='R'>,
            <AxesSubplot:xlabel='RBI', ylabel='R'>],
           [<AxesSubplot:xlabel='3B', ylabel='H'>,
            <AxesSubplot:xlabel='HR', ylabel='H'>,
            <AxesSubplot:xlabel='BB', ylabel='H'>,
            <AxesSubplot:xlabel='SB', ylabel='H'>,
            <AxesSubplot:xlabel='SF', ylabel='H'>,
            <AxesSubplot:xlabel='PO', ylabel='H'>,
            <AxesSubplot:xlabel='E', ylabel='H'>,
            <AxesSubplot:xlabel='oBB', ylabel='H'>,
            <AxesSubplot:xlabel='o2B', ylabel='H'>,
            <AxesSubplot:xlabel='oHR', ylabel='H'>,
            <AxesSubplot:xlabel='oH', ylabel='H'>,
            <AxesSubplot:xlabel='oR', ylabel='H'>,
            <AxesSubplot:xlabel='ER', ylabel='H'>,
            <AxesSubplot:xlabel='ERA', ylabel='H'>,
            <AxesSubplot:xlabel='opps', ylabel='H'>,
            <AxesSubplot:xlabel='penman', ylabel='H'>,
            <AxesSubplot:xlabel='AVG', ylabel='H'>,
            <AxesSubplot:xlabel='AB', ylabel='H'>,
            <AxesSubplot:xlabel='R', ylabel='H'>,
            <AxesSubplot:xlabel='H', ylabel='H'>,
            <AxesSubplot:xlabel='RBI', ylabel='H'>],
           [<AxesSubplot:xlabel='3B', ylabel='RBI'>,
            <AxesSubplot:xlabel='HR', ylabel='RBI'>,
            <AxesSubplot:xlabel='BB', ylabel='RBI'>,
            <AxesSubplot:xlabel='SB', ylabel='RBI'>,
            <AxesSubplot:xlabel='SF', ylabel='RBI'>,
            <AxesSubplot:xlabel='PO', ylabel='RBI'>,
            <AxesSubplot:xlabel='E', ylabel='RBI'>,
            <AxesSubplot:xlabel='oBB', ylabel='RBI'>,
            <AxesSubplot:xlabel='o2B', ylabel='RBI'>,
            <AxesSubplot:xlabel='oHR', ylabel='RBI'>,
            <AxesSubplot:xlabel='oH', ylabel='RBI'>,
            <AxesSubplot:xlabel='oR', ylabel='RBI'>,
            <AxesSubplot:xlabel='ER', ylabel='RBI'>,
            <AxesSubplot:xlabel='ERA', ylabel='RBI'>,
            <AxesSubplot:xlabel='opps', ylabel='RBI'>,
            <AxesSubplot:xlabel='penman', ylabel='RBI'>,
            <AxesSubplot:xlabel='AVG', ylabel='RBI'>,
            <AxesSubplot:xlabel='AB', ylabel='RBI'>,
            <AxesSubplot:xlabel='R', ylabel='RBI'>,
            <AxesSubplot:xlabel='H', ylabel='RBI'>,
            <AxesSubplot:xlabel='RBI', ylabel='RBI'>]], dtype=object)




    
![png](output_39_1.png)
    



```python
#Creating a binary numeric target variable for our analysis 
#Function to assign 0 = L and 1 = W
def wins_losses(w_l):
    if w_l == 'W':
        return 1
    else:
        return 0

#Storing the binary variables in result 
df['result'] = df['W/L'].apply(wins_losses)
```


```python
#Storing the variable win/loss into the highly correlated variables dataframe (variables)
frames_p['result'] = df['result']
frames_h['result'] = df.result
```


```python
#Win percentage compared to varibles (hitters)
fig, ax = plt.subplots(figsize=(12,9))
fig.suptitle('Advanced Statistics Correlation with Win Percentage', x = .44, fontsize=20)
win_corr = (frames_h.corr()[['result']].sort_values(by='result',ascending=False))
hm = sns.heatmap(win_corr, annot=True, cmap ='BuGn')
hm.set_yticklabels(hm.get_yticklabels(), fontsize=15)
hm.set_xticklabels(hm.get_xticklabels(), fontsize=15);
```


    
![png](output_42_0.png)
    


Runs, RBI, Average are the most correlated variables to winning for SNHU Penman Baseball.


```python
#Win percentage compared to varibles (pitchers)
fig, ax = plt.subplots(figsize=(12,9))
fig.suptitle('Pitching Correlation to Winning', x = .44, fontsize=20)
win_corr = (frames_p.corr().abs()[['result']].sort_values(by='result',ascending=False))
hm = sns.heatmap(win_corr, annot=True, cmap ='YlGn')
hm.set_yticklabels(hm.get_yticklabels(), fontsize=15)
hm.set_xticklabels(hm.get_xticklabels(), fontsize=15);
```


    
![png](output_44_0.png)
    


Runs, ERA, ER are the most correlated variables to winning for SNHU Penman Baseball.


```python
#Creating a function to look at home,away and neutral
def home_away(h_a):   
    if h_a == 'vs':
        return 'home'
    elif h_a == 'at':
        return 'away'
    else:
        return 'neutral'
df['site'] = df['Loc'].apply(home_away)
```


```python
era_avg = pd.DataFrame(df.groupby(['site'])['ERA'].mean())
#Bar plot looking at average ERA at different sites
plt.bar(era_avg.index,era_avg.ERA, color ='gold',
        width = 0.4)
plt.xticks(ha='right', rotation=55, fontsize=35, fontname='monospace')
plt.yticks(rotation=55, fontsize=35, fontname='monospace')
plt.xlabel("Location",fontsize=35)
plt.ylabel("Average ERA",fontsize=35)
plt.title("Average ERA (H,A,N)",fontsize=35)
plt.show()
```


    
![png](output_47_0.png)
    


Looking at the average era of each of the different locations. This holds the common understanding of home field advantage showing a lower era while the Penman play at home. Neutral settings are actually holding the lowest era out of the three options. The reason that neutral is an option is becuase in college baseball you have times that fields may flood, snow (turf complex) and then playoffs are played at neutral settings if the host team is kicked out of the tournament.


```python
ba_avg = pd.DataFrame(df.groupby(['site'])['AVG'].mean())
#Bar plot looking at average runs at different sites
plt.bar(ba_avg.index,ba_avg.AVG, color ='blue',
        width = 0.4)
plt.xticks(ha='right', rotation=55, fontsize=35, fontname='monospace')
plt.yticks(rotation=55, fontsize=35, fontname='monospace')
plt.xlabel("Location",fontsize=35)
plt.ylabel("Average BA",fontsize=35)
plt.title("Average BA (H,A,N)",fontsize=35)
plt.show()
```


    
![png](output_49_0.png)
    


Home and away does not sem to have a huge impact for SNHU hitters. Neutral settings are higher on average compared to both home and away.

# Hypothesis Testing Winning Percentage (Chi^2)


```python
#Calculating the win percentage over the past 4 full seasons 
print("Over the past 4 full seasons SNHU baseball has won ", float(sum(df.result)/len(df)), "% of the time ")
print("Time to test the assumption with Hypothesis Chi-Squared")
```

    Over the past 4 full seasons SNHU baseball has won  0.771978021978022 % of the time 
    Time to test the assumption with Hypothesis Chi-Squared



```python
#Printing out the null and alternative 
print('null =',float(sum(df.result)/len(df)))
print('alternative =!', float(sum(df.result)/len(df)))
```

    null = 0.771978021978022
    alternative =! 0.771978021978022



```python
#Subsetting the total columns sample that will be examined in a hypothesis testing using 50 rows of data 
sample = df['result'][np.argsort(np.random.random(150))[:100]]
```


```python
#Storing the sample mean, pop mean, sample size and standard deviation 
sample_mean = float(sample.mean())
h_mean = float(df['result'].mean())
N = len(sample)
dev = np.std(df['result'])
print("Sample mean = ", sample_mean)
print("Hypothesis mean = ", h_mean)
print("Sample size = ", N )
print("Standard deviation = ", dev)
```

    Sample mean =  0.82
    Hypothesis mean =  0.771978021978022
    Sample size =  100
    Standard deviation =  0.4195568561719867

Sample mean =  0.81
Hypothesis mean =  0.771978021978022
Sample size =  100
Standard deviation =  0.4195568561719867

```python
# Z-score 
z = (sample_mean - h_mean)/(dev/math.sqrt(N))
print("Z-score = ", z)
```

    Z-score =  1.1445880889691038

Z-score =  0.9062413702135259

```python
# P value 
p_value = stats.norm.sf(abs(z))
if p_value > .05:
    print("Reject the null.","P-value =", p_value)
else:
    print("Null.","P-value= ", p_value)
```

    Reject the null. P-value = 0.12618991432462073

Reject the null. P-value = 0.12618991432462073
# Exporting CSV to Avoid Hardcoding 


```python
import os
```


```python
os.chdir("Desktop/Portfolio Project")
```


    ---------------------------------------------------------------------------

    FileNotFoundError                         Traceback (most recent call last)

    Input In [97], in <cell line: 1>()
    ----> 1 os.chdir("Desktop/Portfolio Project")


    FileNotFoundError: [Errno 2] No such file or directory: 'Desktop/Portfolio Project'



```python
#df = pd.read_csv("snhu_data.csv")
```


```python
df.to_csv("snhu_data.csv")
```

#                                      Statistical Modeling


```python
#Changing the box plot size and layout
plt.rcParams["figure.figsize"] = [12, 6]
plt.rcParams["figure.autolayout"] = True
```


```python
#Packages needed for random forest
from sklearn.ensemble import RandomForestRegressor
from sklearn.metrics import mean_squared_error
from mlxtend.plotting import plot_confusion_matrix
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score, confusion_matrix, classification_report
from sklearn.inspection import permutation_importance
from sklearn.linear_model import LogisticRegression
from sklearn.naive_bayes import GaussianNB
from sklearn.svm import SVC
from sklearn.ensemble import GradientBoostingClassifier
from sklearn.model_selection import GridSearchCV
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import cross_validate
from sklearn.ensemble import RandomForestClassifier
```


```python
#Subsetting all the variables used in random forest
#selection = df[['oH','oR','ER','ERA','opps','penman','AVG','AB','R','H','RBI','result']]
```


```python
#Variables that will be taken into the random forest model 
#selection.info()
```

1. Call the model

2. Fit to train dataset

3. Use fitted model

4. Evaluate model

# Subsetting Target and Variables


```python
original_variables = variables 
```


```python
#variables = df[['3B','HR', 'BB', 'SB','SF','PO','E','oBB','o2B','oHR','oH','oR', 'ER','ERA', 'opps', 'penman', 'AVG', 'AB', 'R','H', 'RBI']]
variables = df[['3B','HR', 'BB', 'SB','SF','PO','E','oBB','o2B','oHR','oH','oR', 'ER','ERA', 'AVG', 'AB', 'R','H', 'RBI']]
```


```python
#Selecting everyhting except target
x = variables.iloc[:, :-1]
#Grabbing the target variables
#y = variables.iloc[:, -1]
y = df.result
```


```python
#Splitting the sets up for train and test
#Splitting test size 25% of all observations
x_train,x_test,y_train,y_test = train_test_split(x,y, test_size = 0.15, random_state = 42)
```


```python
#Using standard scaler to improve the model performance
ss = StandardScaler()

X_sc_train = ss.fit_transform(x_train)
X_sc_test = ss.fit_transform(x_test)
```


```python
#LogisticRegression
lr = LogisticRegression()

pd.DataFrame(pd.DataFrame(cross_validate(lr, X_sc_train, y_train, return_train_score=True)).mean())
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
      <th>0</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>fit_time</th>
      <td>0.006434</td>
    </tr>
    <tr>
      <th>score_time</th>
      <td>0.000353</td>
    </tr>
    <tr>
      <th>test_score</th>
      <td>0.957959</td>
    </tr>
    <tr>
      <th>train_score</th>
      <td>0.995958</td>
    </tr>
  </tbody>
</table>
</div>




```python
#RandomForest
rfc = RandomForestClassifier(n_estimators=100)

pd.DataFrame(pd.DataFrame(cross_validate(rfc, X_sc_train, y_train, return_train_score=True)).mean())
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
      <th>0</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>fit_time</th>
      <td>0.073998</td>
    </tr>
    <tr>
      <th>score_time</th>
      <td>0.004647</td>
    </tr>
    <tr>
      <th>test_score</th>
      <td>0.970968</td>
    </tr>
    <tr>
      <th>train_score</th>
      <td>1.000000</td>
    </tr>
  </tbody>
</table>
</div>




```python
#GuassianNB
gnb = GaussianNB()

pd.DataFrame(pd.DataFrame(cross_validate(gnb, X_sc_train, y_train, return_train_score=True)).mean())
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
      <th>0</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>fit_time</th>
      <td>0.000498</td>
    </tr>
    <tr>
      <th>score_time</th>
      <td>0.000226</td>
    </tr>
    <tr>
      <th>test_score</th>
      <td>0.896404</td>
    </tr>
    <tr>
      <th>train_score</th>
      <td>0.915855</td>
    </tr>
  </tbody>
</table>
</div>




```python
#cross_validate
svc = SVC(kernel='linear')

pd.DataFrame(pd.DataFrame(cross_validate(svc, X_sc_train, y_train, return_train_score=True)).mean())
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
      <th>0</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>fit_time</th>
      <td>0.002473</td>
    </tr>
    <tr>
      <th>score_time</th>
      <td>0.000530</td>
    </tr>
    <tr>
      <th>test_score</th>
      <td>0.990323</td>
    </tr>
    <tr>
      <th>train_score</th>
      <td>0.998381</td>
    </tr>
  </tbody>
</table>
</div>




```python
#GradientBoostingClassifier
gbc = GradientBoostingClassifier()
pd.DataFrame(pd.DataFrame(cross_validate(gbc, X_sc_train, y_train, return_train_score=True)).mean())
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
      <th>0</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>fit_time</th>
      <td>0.054649</td>
    </tr>
    <tr>
      <th>score_time</th>
      <td>0.000369</td>
    </tr>
    <tr>
      <th>test_score</th>
      <td>0.961290</td>
    </tr>
    <tr>
      <th>train_score</th>
      <td>1.000000</td>
    </tr>
  </tbody>
</table>
</div>



# Fitting model

Logistic regression returned the best accuracy so we will fit the model to logistic regression.


```python
#Assigning the classifier to rf variable 
lr.fit(x_train,y_train)
```

    /Users/tylerbrown/opt/anaconda3/lib/python3.9/site-packages/sklearn/linear_model/_logistic.py:814: ConvergenceWarning: lbfgs failed to converge (status=1):
    STOP: TOTAL NO. of ITERATIONS REACHED LIMIT.
    
    Increase the number of iterations (max_iter) or scale the data as shown in:
        https://scikit-learn.org/stable/modules/preprocessing.html
    Please also refer to the documentation for alternative solver options:
        https://scikit-learn.org/stable/modules/linear_model.html#logistic-regression
      n_iter_i = _check_optimize_result(





    LogisticRegression()




```python
gnb.fit(x_train,y_train)
```




    GaussianNB()



# Predictors


```python
#Predicting the variable outputs
y_pred = lr.predict(x_test)
#Must round to the nearest whole number to be able to compare outputs
y_pred = np.rint(y_pred)
```


```python
#Variable selection importance
#plt.bar(x.columns.values,lr.feature_importances_)
#plt.xticks(rotation=55, fontsize=40)
#plt.title("Variable Importance", fontsize = 60)
```

# Assessment


```python
from sklearn import metrics
```


```python
#Accuracy score for random forest model 
accuracy_score(y_test, y_pred)
```




    0.9818181818181818




```python
#Confusion Matrix 
conf_matrix = metrics.confusion_matrix(y_test, y_pred)
conf_matrix
```




    array([[12,  0],
           [ 1, 42]])




```python
#Plotting confusion matrix 
fig, ax = plot_confusion_matrix(conf_mat=conf_matrix, figsize=(6, 6))
plt.xlabel('Predictions', fontsize=18)
plt.ylabel('Actuals', fontsize=18)
plt.title('Confusion Matrix', fontsize=18)
plt.show()
```


    
![png](output_97_0.png)
    



```python
#Classification report
print(classification_report(y_test, y_pred))
```

                  precision    recall  f1-score   support
    
               0       0.92      1.00      0.96        12
               1       1.00      0.98      0.99        43
    
        accuracy                           0.98        55
       macro avg       0.96      0.99      0.97        55
    weighted avg       0.98      0.98      0.98        55
    



```python
#Mean squared error 
mean_squared_error(y_test,y_pred)
```




    0.01818181818181818




```python
#Root mean squared error
np.sqrt(mean_squared_error(y_test,y_pred))
```




    0.13483997249264842



# Results vs Actual


```python
comparison = pd.DataFrame(y_test.astype(float))
```


```python
comparison['predict'] = y_pred
comparison.index = pd.to_datetime(comparison.index)
```


```python
#Comparing the results of the predicted vs the actual
comparison.sort_index().cumsum().plot(linewidth=4.0)
plt.xlabel('Predictions', fontsize=30)
plt.ylabel('Actuals', fontsize=30)
plt.xticks(ha='right', rotation=55, fontsize=35, fontname='monospace')
plt.yticks(rotation=55, fontsize=35, fontname='monospace')
plt.title('Predicted vs Actual Logistic Regression', fontsize=30)
plt.legend(loc=2,prop={'size': 30})
plt.show()
```


    
![png](output_104_0.png)
    


# ROC Curve


```python
from sklearn.metrics import roc_curve, roc_auc_score
```


```python
#Inputs for plots
fpr1, tpr1, _a = metrics.roc_curve(y_test,  y_pred)
```


```python
#Auc score calculation
auc_score=[metrics.roc_auc_score(y_test,i) for i in [y_pred] ]
auc_score
```




    [0.9883720930232558]




```python
#ROC Curve
plt.plot(fpr1,tpr1,label="ROC = {roc}".format(roc = auc_score))
plt.legend(loc=4,fontsize=30)
plt.grid(color='b', ls = '-.', lw = 0.25)
plt.title("ROC", fontsize=60)
plt.show()
```


    
![png](output_109_0.png)
    



```python

```
