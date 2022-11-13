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
#year variable to extract different years from the snhu website 
year = 2018
```


```python
#Reading in data
snhu_18 = pd.read_html('https://snhupenmen.com/sports/baseball/stats/{year}'.format(year = year), header = 0)
```

# Pitchers Cleaning/Formatting


```python
#Converting to a dataframe instead of a list (hitting)
snhu_18_p = pd.DataFrame(snhu_18[7])
snhu_19_p = pd.DataFrame(snhu_19[7])
snhu_21_p = pd.DataFrame(snhu_21[7])
snhu_22_p = pd.DataFrame(snhu_22[7])
```


```python
#Storing dataframes in list
frames_p = [snhu_18_p,snhu_19_p,snhu_21_p,snhu_22_p]
```


```python
#Setting the indexes to drop the Total Columns 
#Total columns will skew means because of large values
snhu_18_p = snhu_18_p.set_index('Date')
snhu_19_p = snhu_19_p.set_index('Date')
snhu_21_p = snhu_21_p.set_index('Date')
snhu_22_p = snhu_22_p.set_index('Date')
```


```python
#Dropping all year end sum columns 
snhu_18_p = snhu_18_p.drop(index = ('Total'))
snhu_19_p = snhu_19_p.drop(index = ('Total'))
snhu_21_p = snhu_21_p.drop(index = ('Total'))
snhu_22_p = snhu_22_p.drop(index = ('Total'))
```


```python
#Combining all dataframes that we are analyzing 
frames_p = [snhu_18_p,snhu_19_p,snhu_21_p,snhu_22_p]
```


```python
#Concating pitchers data 
frames_p = pd.concat(frames_p)

```


```python
#Dropping repeat columns for the merge
frames_p = frames_p.drop(labels = ['Loc','Opponent','W/L', 'Score','Score.1'], axis = 1)

```


```python
#Top 5 in pthcers data 
frames_p
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
      <th>IP</th>
      <th>H</th>
      <th>R</th>
      <th>ER</th>
      <th>BB</th>
      <th>SO</th>
      <th>2B</th>
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
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2/22/2018</th>
      <td>9.0</td>
      <td>3</td>
      <td>3</td>
      <td>1</td>
      <td>4</td>
      <td>11</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1.00</td>
    </tr>
    <tr>
      <th>2/23/2018</th>
      <td>9.0</td>
      <td>11</td>
      <td>8</td>
      <td>8</td>
      <td>1</td>
      <td>8</td>
      <td>3</td>
      <td>2</td>
      <td>1</td>
      <td>0</td>
      <td>3</td>
      <td>2</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>8.00</td>
    </tr>
    <tr>
      <th>02/24/18</th>
      <td>9.0</td>
      <td>11</td>
      <td>8</td>
      <td>6</td>
      <td>4</td>
      <td>9</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>6.00</td>
    </tr>
    <tr>
      <th>2/24/2018</th>
      <td>9.0</td>
      <td>10</td>
      <td>5</td>
      <td>5</td>
      <td>5</td>
      <td>5</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>4</td>
      <td>0</td>
      <td>0</td>
      <td>5.00</td>
    </tr>
    <tr>
      <th>02/25/18</th>
      <td>9.0</td>
      <td>13</td>
      <td>6</td>
      <td>6</td>
      <td>3</td>
      <td>15</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>5</td>
      <td>0</td>
      <td>0</td>
      <td>6.00</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>5/27/2022</th>
      <td>9.0</td>
      <td>11</td>
      <td>6</td>
      <td>4</td>
      <td>3</td>
      <td>9</td>
      <td>3</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>44</td>
      <td>10</td>
      <td>0</td>
      <td>4.00</td>
    </tr>
    <tr>
      <th>5/27/2022</th>
      <td>9.0</td>
      <td>7</td>
      <td>5</td>
      <td>2</td>
      <td>3</td>
      <td>8</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>45</td>
      <td>10</td>
      <td>0</td>
      <td>2.00</td>
    </tr>
    <tr>
      <th>5/29/2022</th>
      <td>9.0</td>
      <td>7</td>
      <td>3</td>
      <td>2</td>
      <td>3</td>
      <td>5</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>46</td>
      <td>10</td>
      <td>0</td>
      <td>2.00</td>
    </tr>
    <tr>
      <th>6/5/2022</th>
      <td>9.0</td>
      <td>11</td>
      <td>7</td>
      <td>5</td>
      <td>5</td>
      <td>8</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>46</td>
      <td>11</td>
      <td>0</td>
      <td>5.00</td>
    </tr>
    <tr>
      <th>6/7/2022</th>
      <td>8.0</td>
      <td>9</td>
      <td>7</td>
      <td>7</td>
      <td>6</td>
      <td>5</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>6</td>
      <td>0</td>
      <td>46</td>
      <td>12</td>
      <td>0</td>
      <td>7.88</td>
    </tr>
  </tbody>
</table>
<p>208 rows × 17 columns</p>
</div>



# Hitting Cleaning


```python
#Converting to a dataframe instead of a list (hitting)
snhu_18 = pd.DataFrame(snhu_18[6])
snhu_19 = pd.DataFrame(snhu_19[6])
snhu_21 = pd.DataFrame(snhu_21[6])
snhu_22 = pd.DataFrame(snhu_22[6])
```


```python
#Setting the indexes to drop the Total Columns 
#Total columns will skew means because of large values
snhu_18 = snhu_18.set_index('Date')
snhu_19 = snhu_19.set_index('Date')
snhu_21 = snhu_21.set_index('Date')
snhu_22 = snhu_22.set_index('Date')
```


```python
#Dropping all year end sum columns 
snhu_18 = snhu_18.drop(index = ('Total'))
snhu_19 = snhu_19.drop(index = ('Total'))
snhu_21 = snhu_21.drop(index = ('Total'))
snhu_22 = snhu_22.drop(index = ('Total'))
```


```python
#Combining all dataframes that we are analyzing 
frames_h = [snhu_18,snhu_19,snhu_21,snhu_22]
```


```python
#Combining all frames together
frames_h = pd.concat(frames)
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
      <th>2/22/2018</th>
      <td>vs</td>
      <td>Bridgeport</td>
      <td>W</td>
      <td>9-3</td>
      <td>36</td>
      <td>9</td>
      <td>13</td>
      <td>7</td>
      <td>3</td>
      <td>1</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>2/23/2018</th>
      <td>vs</td>
      <td>Felician</td>
      <td>W</td>
      <td>12-8</td>
      <td>31</td>
      <td>12</td>
      <td>6</td>
      <td>11</td>
      <td>1</td>
      <td>1</td>
      <td>...</td>
      <td>2</td>
      <td>1</td>
      <td>0</td>
      <td>3</td>
      <td>2</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>8.0</td>
    </tr>
    <tr>
      <th>02/24/18</th>
      <td>vs</td>
      <td>LIU Post</td>
      <td>W</td>
      <td>9-8</td>
      <td>34</td>
      <td>9</td>
      <td>10</td>
      <td>9</td>
      <td>3</td>
      <td>0</td>
      <td>...</td>
      <td>1</td>
      <td>1</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>6.0</td>
    </tr>
    <tr>
      <th>2/24/2018</th>
      <td>vs</td>
      <td>Mercy College</td>
      <td>W</td>
      <td>12-5</td>
      <td>39</td>
      <td>12</td>
      <td>16</td>
      <td>10</td>
      <td>2</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>4</td>
      <td>0</td>
      <td>0</td>
      <td>5.0</td>
    </tr>
    <tr>
      <th>02/25/18</th>
      <td>vs</td>
      <td>Le Moyne</td>
      <td>W</td>
      <td>8-6</td>
      <td>35</td>
      <td>8</td>
      <td>12</td>
      <td>8</td>
      <td>2</td>
      <td>2</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>5</td>
      <td>0</td>
      <td>0</td>
      <td>6.0</td>
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

# End Of Merges 


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
    Index: 208 entries, 2/22/2018 to 6/7/2022
    Data columns (total 43 columns):
     #   Column    Non-Null Count  Dtype  
    ---  ------    --------------  -----  
     0   Loc       132 non-null    object 
     1   Opponent  208 non-null    object 
     2   W/L       208 non-null    object 
     3   Score     208 non-null    object 
     4   AB        208 non-null    int64  
     5   R         208 non-null    int64  
     6   H         208 non-null    int64  
     7   RBI       208 non-null    int64  
     8   2B        208 non-null    int64  
     9   3B        208 non-null    int64  
     10  HR        208 non-null    int64  
     11  BB        208 non-null    int64  
     12  IBB       208 non-null    int64  
     13  SB        208 non-null    int64  
     14  CS        208 non-null    int64  
     15  HBP       208 non-null    int64  
     16  SH        208 non-null    int64  
     17  SF        208 non-null    int64  
     18  GDP       208 non-null    int64  
     19  K         208 non-null    int64  
     20  PO        208 non-null    int64  
     21  A         208 non-null    int64  
     22  E         208 non-null    int64  
     23  AVG       208 non-null    float64
     24  IP        208 non-null    float64
     25  oH        208 non-null    int64  
     26  oR        208 non-null    int64  
     27  ER        208 non-null    int64  
     28  oBB       208 non-null    int64  
     29  SO        208 non-null    int64  
     30  o2B       208 non-null    int64  
     31  o3B       208 non-null    int64  
     32  oHR       208 non-null    int64  
     33  WP        208 non-null    int64  
     34  BK        208 non-null    int64  
     35  oHBP      208 non-null    int64  
     36  oIBB      208 non-null    int64  
     37  W         208 non-null    int64  
     38  L         208 non-null    int64  
     39  SV        208 non-null    int64  
     40  ERA       208 non-null    float64
     41  penman    208 non-null    int64  
     42  opps      208 non-null    int64  
    dtypes: float64(3), int64(36), object(4)
    memory usage: 71.5+ KB



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
      <th>2/22/2018</th>
      <td>vs</td>
      <td>Bridgeport</td>
      <td>W</td>
      <td>9-3</td>
      <td>36</td>
      <td>9</td>
      <td>13</td>
      <td>7</td>
      <td>3</td>
      <td>1</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1.0</td>
      <td>9</td>
      <td>3</td>
    </tr>
    <tr>
      <th>2/23/2018</th>
      <td>vs</td>
      <td>Felician</td>
      <td>W</td>
      <td>12-8</td>
      <td>31</td>
      <td>12</td>
      <td>6</td>
      <td>11</td>
      <td>1</td>
      <td>1</td>
      <td>...</td>
      <td>0</td>
      <td>3</td>
      <td>2</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>8.0</td>
      <td>12</td>
      <td>8</td>
    </tr>
    <tr>
      <th>02/24/18</th>
      <td>vs</td>
      <td>LIU Post</td>
      <td>W</td>
      <td>9-8</td>
      <td>34</td>
      <td>9</td>
      <td>10</td>
      <td>9</td>
      <td>3</td>
      <td>0</td>
      <td>...</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>6.0</td>
      <td>9</td>
      <td>8</td>
    </tr>
    <tr>
      <th>2/24/2018</th>
      <td>vs</td>
      <td>Mercy College</td>
      <td>W</td>
      <td>12-5</td>
      <td>39</td>
      <td>12</td>
      <td>16</td>
      <td>10</td>
      <td>2</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>4</td>
      <td>0</td>
      <td>0</td>
      <td>5.0</td>
      <td>12</td>
      <td>5</td>
    </tr>
    <tr>
      <th>02/25/18</th>
      <td>vs</td>
      <td>Le Moyne</td>
      <td>W</td>
      <td>8-6</td>
      <td>35</td>
      <td>8</td>
      <td>12</td>
      <td>8</td>
      <td>2</td>
      <td>2</td>
      <td>...</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>5</td>
      <td>0</td>
      <td>0</td>
      <td>6.0</td>
      <td>8</td>
      <td>6</td>
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
      <td>208.000000</td>
      <td>208.000000</td>
      <td>208.000000</td>
      <td>208.000000</td>
      <td>208.000000</td>
      <td>208.000000</td>
      <td>208.000000</td>
      <td>208.000000</td>
      <td>208.000000</td>
      <td>208.000000</td>
      <td>...</td>
      <td>208.000000</td>
      <td>208.000000</td>
      <td>208.000000</td>
      <td>208.000000</td>
      <td>208.000000</td>
      <td>208.000000</td>
      <td>208.000000</td>
      <td>208.000000</td>
      <td>208.000000</td>
      <td>208.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>34.115385</td>
      <td>7.211538</td>
      <td>9.947115</td>
      <td>6.336538</td>
      <td>1.894231</td>
      <td>0.274038</td>
      <td>0.985577</td>
      <td>4.298077</td>
      <td>0.057692</td>
      <td>1.961538</td>
      <td>...</td>
      <td>0.879808</td>
      <td>0.139423</td>
      <td>0.740385</td>
      <td>0.139423</td>
      <td>20.572115</td>
      <td>6.663462</td>
      <td>0.067308</td>
      <td>3.158798</td>
      <td>7.211538</td>
      <td>3.783654</td>
    </tr>
    <tr>
      <th>std</th>
      <td>5.436160</td>
      <td>4.635867</td>
      <td>4.082730</td>
      <td>4.271262</td>
      <td>1.641363</td>
      <td>0.535592</td>
      <td>1.144102</td>
      <td>2.587147</td>
      <td>0.233723</td>
      <td>1.795986</td>
      <td>...</td>
      <td>1.247210</td>
      <td>0.466028</td>
      <td>1.002366</td>
      <td>0.410942</td>
      <td>12.273635</td>
      <td>4.453974</td>
      <td>0.251159</td>
      <td>3.140991</td>
      <td>4.635867</td>
      <td>3.297829</td>
    </tr>
    <tr>
      <th>min</th>
      <td>20.000000</td>
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
      <td>2.000000</td>
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
      <td>6.000000</td>
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
      <td>19.000000</td>
      <td>7.000000</td>
      <td>0.000000</td>
      <td>2.250000</td>
      <td>7.000000</td>
      <td>3.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>37.250000</td>
      <td>10.000000</td>
      <td>12.000000</td>
      <td>9.000000</td>
      <td>3.000000</td>
      <td>0.000000</td>
      <td>2.000000</td>
      <td>6.000000</td>
      <td>0.000000</td>
      <td>3.000000</td>
      <td>...</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>30.000000</td>
      <td>10.000000</td>
      <td>0.000000</td>
      <td>4.625000</td>
      <td>10.000000</td>
      <td>5.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>54.000000</td>
      <td>24.000000</td>
      <td>24.000000</td>
      <td>23.000000</td>
      <td>8.000000</td>
      <td>2.000000</td>
      <td>5.000000</td>
      <td>13.000000</td>
      <td>1.000000</td>
      <td>9.000000</td>
      <td>...</td>
      <td>9.000000</td>
      <td>3.000000</td>
      <td>6.000000</td>
      <td>3.000000</td>
      <td>46.000000</td>
      <td>17.000000</td>
      <td>1.000000</td>
      <td>20.570000</td>
      <td>24.000000</td>
      <td>18.000000</td>
    </tr>
  </tbody>
</table>
<p>8 rows × 39 columns</p>
</div>




```python
#How much Penman outscore oppenant by 
print("On average the Penman outscore their opponent by ", df.penman.mean() - df.opps.mean(), "runs")
```

    On average the Penman outscore their opponent by  3.4278846153846154 runs



```python
#Changing the box plot size and layout
plt.rcParams["figure.figsize"] = [12, 6]
plt.rcParams["figure.autolayout"] = True
```


```python
#Printing out columns for box plots 
list(df.columns)
```




    ['Loc',
     'Opponent',
     'W/L',
     'Score',
     'AB',
     'R',
     'H',
     'RBI',
     '2B',
     '3B',
     'HR',
     'BB',
     'IBB',
     'SB',
     'CS',
     'HBP',
     'SH',
     'SF',
     'GDP',
     'K',
     'PO',
     'A',
     'E',
     'AVG',
     'IP',
     'oH',
     'oR',
     'ER',
     'oBB',
     'SO',
     'o2B',
     'o3B',
     'oHR',
     'WP',
     'BK',
     'oHBP',
     'oIBB',
     'W',
     'L',
     'SV',
     'ERA',
     'penman',
     'opps']




```python
#Box plot of pitchers data 
df.boxplot(column = ['SO', 'o2B','oHR','oH','oR','oHBP','oIBB','IP','o3B','WP'])
plt.title('Pitchers Stats')
```




    Text(0.5, 1.0, 'Pitchers Stats')




    
![png](output_35_1.png)
    



```python
#Box plot of the majority of the columns for hitters 
df.boxplot(column = ['AB','R','H','RBI','2B','HR','BB','IBB','SB','HBP','GDP','K','PO','A','E','penman','opps'])
plt.title("Hitting Stats")



```




    Text(0.5, 1.0, 'Hitting Stats')




    
![png](output_36_1.png)
    



```python
df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Index: 208 entries, 2/22/2018 to 6/7/2022
    Data columns (total 43 columns):
     #   Column    Non-Null Count  Dtype  
    ---  ------    --------------  -----  
     0   Loc       132 non-null    object 
     1   Opponent  208 non-null    object 
     2   W/L       208 non-null    object 
     3   Score     208 non-null    object 
     4   AB        208 non-null    int64  
     5             208 non-null    int64  
     6   oH        208 non-null    int64  
     7   RBI       208 non-null    int64  
     8   2B        208 non-null    int64  
     9   3B        208 non-null    int64  
     10  HR        208 non-null    int64  
     11  BB        208 non-null    int64  
     12  IBB       208 non-null    int64  
     13  SB        208 non-null    int64  
     14  CS        208 non-null    int64  
     15  HBP       208 non-null    int64  
     16  SH        208 non-null    int64  
     17  SF        208 non-null    int64  
     18  GDP       208 non-null    int64  
     19  K         208 non-null    int64  
     20  PO        208 non-null    int64  
     21  A         208 non-null    int64  
     22  E         208 non-null    int64  
     23  AVG       208 non-null    float64
     24  IP        208 non-null    float64
     25  oH        208 non-null    int64  
     26  oR        208 non-null    int64  
     27  ER        208 non-null    int64  
     28  oBB       208 non-null    int64  
     29  SO        208 non-null    int64  
     30  o2B       208 non-null    int64  
     31  o3B       208 non-null    int64  
     32  oHR       208 non-null    int64  
     33  WP        208 non-null    int64  
     34  BK        208 non-null    int64  
     35  oHBP      208 non-null    int64  
     36  oIBB      208 non-null    int64  
     37  W         208 non-null    int64  
     38  L         208 non-null    int64  
     39  SV        208 non-null    int64  
     40  ERA       208 non-null    float64
     41  penman    208 non-null    int64  
     42  opps      208 non-null    int64  
    dtypes: float64(3), int64(36), object(4)
    memory usage: 71.5+ KB



```python
#Seperating BA according to scale 
df.boxplot(column= ['AVG'])
plt.title("SNHU vs Opps")
```




    Text(0.5, 1.0, 'SNHU vs Opps')




    
![png](output_38_1.png)
    



```python
#Seperating '3B', 'SH', 'SF' according to scale 
df.boxplot(column = ['3B', 'SH', 'SF'])
plt.title('Hitters Stats')
```




    Text(0.5, 1.0, 'Hitters Stats')




    
![png](output_39_1.png)
    



```python
df
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
      <th>BK</th>
      <th>oHBP</th>
      <th>oIBB</th>
      <th>W</th>
      <th>L</th>
      <th>SV</th>
      <th>ERA</th>
      <th>penman</th>
      <th>opps</th>
      <th>result</th>
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
      <th>2/22/2018</th>
      <td>vs</td>
      <td>Bridgeport</td>
      <td>W</td>
      <td>9-3</td>
      <td>36</td>
      <td>9</td>
      <td>13</td>
      <td>7</td>
      <td>3</td>
      <td>1</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1.00</td>
      <td>9</td>
      <td>3</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2/23/2018</th>
      <td>vs</td>
      <td>Felician</td>
      <td>W</td>
      <td>12-8</td>
      <td>31</td>
      <td>12</td>
      <td>6</td>
      <td>11</td>
      <td>1</td>
      <td>1</td>
      <td>...</td>
      <td>3</td>
      <td>2</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>8.00</td>
      <td>12</td>
      <td>8</td>
      <td>1</td>
    </tr>
    <tr>
      <th>02/24/18</th>
      <td>vs</td>
      <td>LIU Post</td>
      <td>W</td>
      <td>9-8</td>
      <td>34</td>
      <td>9</td>
      <td>10</td>
      <td>9</td>
      <td>3</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>6.00</td>
      <td>9</td>
      <td>8</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2/24/2018</th>
      <td>vs</td>
      <td>Mercy College</td>
      <td>W</td>
      <td>12-5</td>
      <td>39</td>
      <td>12</td>
      <td>16</td>
      <td>10</td>
      <td>2</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>4</td>
      <td>0</td>
      <td>0</td>
      <td>5.00</td>
      <td>12</td>
      <td>5</td>
      <td>1</td>
    </tr>
    <tr>
      <th>02/25/18</th>
      <td>vs</td>
      <td>Le Moyne</td>
      <td>W</td>
      <td>8-6</td>
      <td>35</td>
      <td>8</td>
      <td>12</td>
      <td>8</td>
      <td>2</td>
      <td>2</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>5</td>
      <td>0</td>
      <td>0</td>
      <td>6.00</td>
      <td>8</td>
      <td>6</td>
      <td>1</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
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
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>44</td>
      <td>10</td>
      <td>0</td>
      <td>4.00</td>
      <td>5</td>
      <td>6</td>
      <td>0</td>
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
      <td>0</td>
      <td>0</td>
      <td>45</td>
      <td>10</td>
      <td>0</td>
      <td>2.00</td>
      <td>7</td>
      <td>5</td>
      <td>1</td>
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
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>46</td>
      <td>10</td>
      <td>0</td>
      <td>2.00</td>
      <td>7</td>
      <td>3</td>
      <td>1</td>
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
      <td>1</td>
      <td>0</td>
      <td>46</td>
      <td>11</td>
      <td>0</td>
      <td>5.00</td>
      <td>4</td>
      <td>7</td>
      <td>0</td>
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
      <td>6</td>
      <td>0</td>
      <td>46</td>
      <td>12</td>
      <td>0</td>
      <td>7.88</td>
      <td>3</td>
      <td>7</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>208 rows × 44 columns</p>
</div>




```python
# Creating plot
fig = plt.figure(figsize =(10, 7))
plt.pie(df['W/L'])
 
# show plot
plt.show()
```


    ---------------------------------------------------------------------------

    ValueError                                Traceback (most recent call last)

    Input In [313], in <cell line: 3>()
          1 # Creating plot
          2 fig = plt.figure(figsize =(10, 7))
    ----> 3 plt.pie(df['W/L'])
          5 # show plot
          6 plt.show()


    File ~/opt/anaconda3/lib/python3.9/site-packages/matplotlib/pyplot.py:2744, in pie(x, explode, labels, colors, autopct, pctdistance, shadow, labeldistance, startangle, radius, counterclock, wedgeprops, textprops, center, frame, rotatelabels, normalize, data)
       2737 @_copy_docstring_and_deprecators(Axes.pie)
       2738 def pie(
       2739         x, explode=None, labels=None, colors=None, autopct=None,
       (...)
       2742         textprops=None, center=(0, 0), frame=False,
       2743         rotatelabels=False, *, normalize=True, data=None):
    -> 2744     return gca().pie(
       2745         x, explode=explode, labels=labels, colors=colors,
       2746         autopct=autopct, pctdistance=pctdistance, shadow=shadow,
       2747         labeldistance=labeldistance, startangle=startangle,
       2748         radius=radius, counterclock=counterclock,
       2749         wedgeprops=wedgeprops, textprops=textprops, center=center,
       2750         frame=frame, rotatelabels=rotatelabels, normalize=normalize,
       2751         **({"data": data} if data is not None else {}))


    File ~/opt/anaconda3/lib/python3.9/site-packages/matplotlib/__init__.py:1412, in _preprocess_data.<locals>.inner(ax, data, *args, **kwargs)
       1409 @functools.wraps(func)
       1410 def inner(ax, *args, data=None, **kwargs):
       1411     if data is None:
    -> 1412         return func(ax, *map(sanitize_sequence, args), **kwargs)
       1414     bound = new_sig.bind(ax, *args, **kwargs)
       1415     auto_label = (bound.arguments.get(label_namer)
       1416                   or bound.kwargs.get(label_namer))


    File ~/opt/anaconda3/lib/python3.9/site-packages/matplotlib/axes/_axes.py:3042, in Axes.pie(self, x, explode, labels, colors, autopct, pctdistance, shadow, labeldistance, startangle, radius, counterclock, wedgeprops, textprops, center, frame, rotatelabels, normalize)
       3039 self.set_aspect('equal')
       3040 # The use of float32 is "historical", but can't be changed without
       3041 # regenerating the test baselines.
    -> 3042 x = np.asarray(x, np.float32)
       3043 if x.ndim > 1:
       3044     raise ValueError("x must be 1D")


    File ~/opt/anaconda3/lib/python3.9/site-packages/pandas/core/series.py:872, in Series.__array__(self, dtype)
        825 def __array__(self, dtype: npt.DTypeLike | None = None) -> np.ndarray:
        826     """
        827     Return the values as a NumPy array.
        828 
       (...)
        870           dtype='datetime64[ns]')
        871     """
    --> 872     return np.asarray(self._values, dtype)


    ValueError: could not convert string to float: 'W'



    
![png](output_41_1.png)
    



```python
#Pearson correlation matrix examing r^2
matrix = df.corr(
    method = 'pearson',  # The method of correlation
    min_periods = 1      # Min number of observations required
)
```


```python
matrix.to_clipboard()
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


    
![png](output_45_0.png)
    



```python
#Highly correlated variable subset 
variables = df[['oH','oR', 'ER','ERA', 'opps', 'penman', 'AVG', 'AB', 'R','H', 'RBI']]
```

# Highly Correlated Variables

*Penman - SNHU runs 
*AVG - Batting Average
*AB - At-bats
*RBI - Runs batted in 
*H - Hits 
*R - runs
*


```python
#Scatter matrix
pd.plotting.scatter_matrix(variables, figsize = (40,40))
```




    array([[<AxesSubplot:xlabel='oH', ylabel='oH'>,
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
           [<AxesSubplot:xlabel='oH', ylabel='oR'>,
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
           [<AxesSubplot:xlabel='oH', ylabel='ER'>,
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
           [<AxesSubplot:xlabel='oH', ylabel='ERA'>,
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
           [<AxesSubplot:xlabel='oH', ylabel='opps'>,
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
           [<AxesSubplot:xlabel='oH', ylabel='penman'>,
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
           [<AxesSubplot:xlabel='oH', ylabel='AVG'>,
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
           [<AxesSubplot:xlabel='oH', ylabel='AB'>,
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
           [<AxesSubplot:xlabel='oH', ylabel='R'>,
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
           [<AxesSubplot:xlabel='oH', ylabel='H'>,
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
           [<AxesSubplot:xlabel='oH', ylabel='RBI'>,
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




    
![png](output_49_1.png)
    



```python
#Calculating the win percentage over the past 4 full seasons 
print("Over the past 4 full seasons SNHU baseball has won ", sum(df.result)/len(df), "% of the time ")
print("Time to test the assumption with Hypothesis Chi-Squared")
```

    Over the past 4 full seasons SNHU baseball has won  0.75 % of the time 
    Time to test the assumption with Hypothesis Chi-Squared


# Hypothesis Testing Winning Percentage (Chi^2)


```python
#Printing out the null and alternative 
print('null = 0.75')
print('alternative =! 0.75')
```

    null = 0.75
    alternative =! 0.75



```python
#Subsetting the total columns sample that will be examined in a hypothesis testing using 50 rows of data 
sample = df['result'][np.argsort(np.random.random(33))[:50]]
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

    Sample mean =  0.6363636363636364
    Hypothesis mean =  0.75
    Sample size =  33
    Standard deviation =  0.4330127018922193



```python
# Z-score 
z = (sample_mean - h_mean)/(dev/math.sqrt(N))
print("Z-score = ", z)
```

    Z-score =  -1.5075567228888183



```python
# P value 
p_value = stats.norm.sf(abs(z))
if p_value > .05:
    print("Reject the null.","P-value =", p_value)
else:
    print("Null.","P-value= ", p_value)
```

    Reject the null. P-value = 0.0658340080114071



```python
#Creating a binary target variable for our analysis 
#Function to assign 0 = L and 1 = W
def wins_losses(w_l):
    if w_l == 'W':
        return 1
    else:
        return 0
```


```python
#Storing the binary variables in result 
df['result'] = df['W/L'].apply(wins_losses)
```

# Decision Tree


```python
X, y = iris.data, iris.target
clf = tree.DecisionTreeClassifier()
clf = clf.fit(X, y)
```


```python

```


```python

```

# Linear Regression 


```python

```


```python

```


```python

```


```python

```
