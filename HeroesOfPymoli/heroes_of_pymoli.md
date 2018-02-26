
# Exploring Data Analsis of Purchase Data


```python
import os
from pandas import Series, DataFrame
import pandas as pd
```

## Load to DataFrame


```python
DATA_PATH = '.'
file = input("What's the input file?")
full_path = os.path.join(DATA_PATH, file)
df_purchase = pd.read_json(full_path)
```

    What's the input file?purchase_data2.json
    

## Take a look at data


```python
df_purchase.columns
```




    Index(['Age', 'Gender', 'Item ID', 'Item Name', 'Price', 'SN'], dtype='object')




```python
df_purchase.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Age</th>
      <th>Gender</th>
      <th>Item ID</th>
      <th>Item Name</th>
      <th>Price</th>
      <th>SN</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>20</td>
      <td>Male</td>
      <td>93</td>
      <td>Apocalyptic Battlescythe</td>
      <td>4.49</td>
      <td>Iloni35</td>
    </tr>
    <tr>
      <th>1</th>
      <td>21</td>
      <td>Male</td>
      <td>12</td>
      <td>Dawne</td>
      <td>3.36</td>
      <td>Aidaira26</td>
    </tr>
    <tr>
      <th>2</th>
      <td>17</td>
      <td>Male</td>
      <td>5</td>
      <td>Putrid Fan</td>
      <td>2.63</td>
      <td>Irim47</td>
    </tr>
    <tr>
      <th>3</th>
      <td>17</td>
      <td>Male</td>
      <td>123</td>
      <td>Twilight's Carver</td>
      <td>2.55</td>
      <td>Irith83</td>
    </tr>
    <tr>
      <th>4</th>
      <td>22</td>
      <td>Male</td>
      <td>154</td>
      <td>Feral Katana</td>
      <td>4.11</td>
      <td>Philodil43</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_purchase.describe()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Age</th>
      <th>Item ID</th>
      <th>Price</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>78.000000</td>
      <td>78.000000</td>
      <td>78.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>22.551282</td>
      <td>93.935897</td>
      <td>2.924359</td>
    </tr>
    <tr>
      <th>std</th>
      <td>7.339003</td>
      <td>56.352633</td>
      <td>1.134913</td>
    </tr>
    <tr>
      <th>min</th>
      <td>7.000000</td>
      <td>0.000000</td>
      <td>1.020000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>20.000000</td>
      <td>48.000000</td>
      <td>1.925000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>22.500000</td>
      <td>97.500000</td>
      <td>2.770000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>25.000000</td>
      <td>137.000000</td>
      <td>4.092500</td>
    </tr>
    <tr>
      <th>max</th>
      <td>40.000000</td>
      <td>181.000000</td>
      <td>4.810000</td>
    </tr>
  </tbody>
</table>
</div>



## Player Count

The SN looks like can uniquely identify a player so get the unique count of it.


```python
DataFrame( [ {
    'Total Playsers': len(df_purchase['SN'].unique())
} ])
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Total Playsers</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>74</td>
    </tr>
  </tbody>
</table>
</div>



## Purchasing Analysis (Total)


```python
DataFrame(
    columns = [
       'Number of Unique Items',
       'Average Price',
       'Number of Purchases',
       'Total Revenue',
    ],
    data = [
       [len(df_purchase['Item ID'].unique()),
        df_purchase.Price.mean(),
        df_purchase.Price.count(),
        df_purchase.Price.sum(),
       ],
    ], 
).style.format(
    {'Average Price': '${:,.2f}',
     'Total Revenue': '${:,.2f}',
    }
)
```




<style  type="text/css" >
</style>  
<table id="T_3ef582a6_19ec_11e8_b5ed_74e54394bcc7" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >Number of Unique Items</th> 
        <th class="col_heading level0 col1" >Average Price</th> 
        <th class="col_heading level0 col2" >Number of Purchases</th> 
        <th class="col_heading level0 col3" >Total Revenue</th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_3ef582a6_19ec_11e8_b5ed_74e54394bcc7level0_row0" class="row_heading level0 row0" >0</th> 
        <td id="T_3ef582a6_19ec_11e8_b5ed_74e54394bcc7row0_col0" class="data row0 col0" >64</td> 
        <td id="T_3ef582a6_19ec_11e8_b5ed_74e54394bcc7row0_col1" class="data row0 col1" >$2.92</td> 
        <td id="T_3ef582a6_19ec_11e8_b5ed_74e54394bcc7row0_col2" class="data row0 col2" >78</td> 
        <td id="T_3ef582a6_19ec_11e8_b5ed_74e54394bcc7row0_col3" class="data row0 col3" >$228.10</td> 
    </tr></tbody> 
</table> 



## Gender Demographics


```python
DataFrame(
    df_purchase.groupby('Gender')['SN'].unique()
).assign(
    cnt = lambda x: x['SN'].map(len)
).assign(
    perc=lambda x: x['cnt'] / len(df_purchase['SN'].unique())
).drop(
    'SN', axis=1
).rename(
    columns = {
        'cnt': 'Toatl Count',
        'perc': 'Percenage of Players'
    }
).style.format(
    {
      'Percenage of Players': '{:.2%}' 
    }
)

```




<style  type="text/css" >
</style>  
<table id="T_3fbfd1ca_19ec_11e8_9ee2_74e54394bcc7" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >Toatl Count</th> 
        <th class="col_heading level0 col1" >Percenage of Players</th> 
    </tr>    <tr> 
        <th class="index_name level0" >Gender</th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_3fbfd1ca_19ec_11e8_9ee2_74e54394bcc7level0_row0" class="row_heading level0 row0" >Female</th> 
        <td id="T_3fbfd1ca_19ec_11e8_9ee2_74e54394bcc7row0_col0" class="data row0 col0" >13</td> 
        <td id="T_3fbfd1ca_19ec_11e8_9ee2_74e54394bcc7row0_col1" class="data row0 col1" >17.57%</td> 
    </tr>    <tr> 
        <th id="T_3fbfd1ca_19ec_11e8_9ee2_74e54394bcc7level0_row1" class="row_heading level0 row1" >Male</th> 
        <td id="T_3fbfd1ca_19ec_11e8_9ee2_74e54394bcc7row1_col0" class="data row1 col0" >60</td> 
        <td id="T_3fbfd1ca_19ec_11e8_9ee2_74e54394bcc7row1_col1" class="data row1 col1" >81.08%</td> 
    </tr>    <tr> 
        <th id="T_3fbfd1ca_19ec_11e8_9ee2_74e54394bcc7level0_row2" class="row_heading level0 row2" >Other / Non-Disclosed</th> 
        <td id="T_3fbfd1ca_19ec_11e8_9ee2_74e54394bcc7row2_col0" class="data row2 col0" >1</td> 
        <td id="T_3fbfd1ca_19ec_11e8_9ee2_74e54394bcc7row2_col1" class="data row2 col1" >1.35%</td> 
    </tr></tbody> 
</table> 



## Purchasing Analysis (Gender)
**What the hell is 'Normalized Total'?**


```python
df_grouped = df_purchase.groupby('Gender')['Price']
df_grouped.agg(
    { 
        'Purchase Count': 'count',
        'Average Purchase Price': 'mean',
        'Total Purchase Value': 'sum'
    }
).style.format(
    {'Average Purchase Price': '${:,.2f}',
     'Total Purchase Value': '${:,.2f}',
    }
)

```

    C:\ProgramData\Anaconda3\lib\site-packages\ipykernel_launcher.py:6: FutureWarning: using a dict on a Series for aggregation
    is deprecated and will be removed in a future version
      
    




<style  type="text/css" >
</style>  
<table id="T_44d34af6_19ec_11e8_95f2_74e54394bcc7" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >Purchase Count</th> 
        <th class="col_heading level0 col1" >Average Purchase Price</th> 
        <th class="col_heading level0 col2" >Total Purchase Value</th> 
    </tr>    <tr> 
        <th class="index_name level0" >Gender</th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_44d34af6_19ec_11e8_95f2_74e54394bcc7level0_row0" class="row_heading level0 row0" >Female</th> 
        <td id="T_44d34af6_19ec_11e8_95f2_74e54394bcc7row0_col0" class="data row0 col0" >13</td> 
        <td id="T_44d34af6_19ec_11e8_95f2_74e54394bcc7row0_col1" class="data row0 col1" >$3.18</td> 
        <td id="T_44d34af6_19ec_11e8_95f2_74e54394bcc7row0_col2" class="data row0 col2" >$41.38</td> 
    </tr>    <tr> 
        <th id="T_44d34af6_19ec_11e8_95f2_74e54394bcc7level0_row1" class="row_heading level0 row1" >Male</th> 
        <td id="T_44d34af6_19ec_11e8_95f2_74e54394bcc7row1_col0" class="data row1 col0" >64</td> 
        <td id="T_44d34af6_19ec_11e8_95f2_74e54394bcc7row1_col1" class="data row1 col1" >$2.88</td> 
        <td id="T_44d34af6_19ec_11e8_95f2_74e54394bcc7row1_col2" class="data row1 col2" >$184.60</td> 
    </tr>    <tr> 
        <th id="T_44d34af6_19ec_11e8_95f2_74e54394bcc7level0_row2" class="row_heading level0 row2" >Other / Non-Disclosed</th> 
        <td id="T_44d34af6_19ec_11e8_95f2_74e54394bcc7row2_col0" class="data row2 col0" >1</td> 
        <td id="T_44d34af6_19ec_11e8_95f2_74e54394bcc7row2_col1" class="data row2 col1" >$2.12</td> 
        <td id="T_44d34af6_19ec_11e8_95f2_74e54394bcc7row2_col2" class="data row2 col2" >$2.12</td> 
    </tr></tbody> 
</table> 



This method does look straight forwad but the feature will be dropped from Pands in the future. Try to pass list then rename instead.


```python
df_purchase.groupby('Gender')['Price'].agg(
    ['count', 'mean', 'sum']
).rename(
    columns = {
        'count': 'Purchase Count',
        'mean': 'Average Purchase Price',
        'sum': 'Total Purchase Value'
    }
).style.format(
    {'Average Purchase Price': '${:,.2f}',
     'Total Purchase Value': '${:,.2f}',
    }
)
```




<style  type="text/css" >
</style>  
<table id="T_4c269536_19f1_11e8_89e6_74e54394bcc7" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >Purchase Count</th> 
        <th class="col_heading level0 col1" >Average Purchase Price</th> 
        <th class="col_heading level0 col2" >Total Purchase Value</th> 
    </tr>    <tr> 
        <th class="index_name level0" >Gender</th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_4c269536_19f1_11e8_89e6_74e54394bcc7level0_row0" class="row_heading level0 row0" >Female</th> 
        <td id="T_4c269536_19f1_11e8_89e6_74e54394bcc7row0_col0" class="data row0 col0" >13</td> 
        <td id="T_4c269536_19f1_11e8_89e6_74e54394bcc7row0_col1" class="data row0 col1" >$3.18</td> 
        <td id="T_4c269536_19f1_11e8_89e6_74e54394bcc7row0_col2" class="data row0 col2" >$41.38</td> 
    </tr>    <tr> 
        <th id="T_4c269536_19f1_11e8_89e6_74e54394bcc7level0_row1" class="row_heading level0 row1" >Male</th> 
        <td id="T_4c269536_19f1_11e8_89e6_74e54394bcc7row1_col0" class="data row1 col0" >64</td> 
        <td id="T_4c269536_19f1_11e8_89e6_74e54394bcc7row1_col1" class="data row1 col1" >$2.88</td> 
        <td id="T_4c269536_19f1_11e8_89e6_74e54394bcc7row1_col2" class="data row1 col2" >$184.60</td> 
    </tr>    <tr> 
        <th id="T_4c269536_19f1_11e8_89e6_74e54394bcc7level0_row2" class="row_heading level0 row2" >Other / Non-Disclosed</th> 
        <td id="T_4c269536_19f1_11e8_89e6_74e54394bcc7row2_col0" class="data row2 col0" >1</td> 
        <td id="T_4c269536_19f1_11e8_89e6_74e54394bcc7row2_col1" class="data row2 col1" >$2.12</td> 
        <td id="T_4c269536_19f1_11e8_89e6_74e54394bcc7row2_col2" class="data row2 col2" >$2.12</td> 
    </tr></tbody> 
</table> 



## Age Demographics


```python
bin_start = 10
bins = [0]
max_age = df_purchase.Age.max() 
bins.extend(range(bin_start, max_age + 1,5))
bins.append(max_age + 1)
bins
```




    [0, 10, 15, 20, 25, 30, 35, 40, 41]




```python
labels = ['< ' + str(bin_start)]
for i in range(1, len(bins) - 2):
    labels.append('{}-{}'.format(bins[i], bins[i+1] -1))
labels.append(str(bins[-2]) + '+')
labels
```




    ['< 10', '10-14', '15-19', '20-24', '25-29', '30-34', '35-39', '40+']




```python
df_purchase.loc[:, 'Bined Age'] = pd.cut(df_purchase['Age'], bins=bins, labels=labels)
```


```python
df_purchase.groupby('Bined Age')['Price'].agg(
    ['count', 'mean', 'sum']
).rename(
    columns = {
        'count': 'Purchase Count',
        'mean': 'Average Purchase Price',
        'sum': 'Total Purchase Value'
    }
).style.format(
    {
        'Average Purchase Price': '${:,.2f}',
        'Total Purchase Value': '${:,.2f}',
    }
)
```




<style  type="text/css" >
</style>  
<table id="T_9497a752_19f3_11e8_be45_74e54394bcc7" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >Purchase Count</th> 
        <th class="col_heading level0 col1" >Average Purchase Price</th> 
        <th class="col_heading level0 col2" >Total Purchase Value</th> 
    </tr>    <tr> 
        <th class="index_name level0" >Bined Age</th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_9497a752_19f3_11e8_be45_74e54394bcc7level0_row0" class="row_heading level0 row0" >< 10</th> 
        <td id="T_9497a752_19f3_11e8_be45_74e54394bcc7row0_col0" class="data row0 col0" >5</td> 
        <td id="T_9497a752_19f3_11e8_be45_74e54394bcc7row0_col1" class="data row0 col1" >$2.76</td> 
        <td id="T_9497a752_19f3_11e8_be45_74e54394bcc7row0_col2" class="data row0 col2" >$13.82</td> 
    </tr>    <tr> 
        <th id="T_9497a752_19f3_11e8_be45_74e54394bcc7level0_row1" class="row_heading level0 row1" >10-14</th> 
        <td id="T_9497a752_19f3_11e8_be45_74e54394bcc7row1_col0" class="data row1 col0" >4</td> 
        <td id="T_9497a752_19f3_11e8_be45_74e54394bcc7row1_col1" class="data row1 col1" >$3.05</td> 
        <td id="T_9497a752_19f3_11e8_be45_74e54394bcc7row1_col2" class="data row1 col2" >$12.21</td> 
    </tr>    <tr> 
        <th id="T_9497a752_19f3_11e8_be45_74e54394bcc7level0_row2" class="row_heading level0 row2" >15-19</th> 
        <td id="T_9497a752_19f3_11e8_be45_74e54394bcc7row2_col0" class="data row2 col0" >20</td> 
        <td id="T_9497a752_19f3_11e8_be45_74e54394bcc7row2_col1" class="data row2 col1" >$2.73</td> 
        <td id="T_9497a752_19f3_11e8_be45_74e54394bcc7row2_col2" class="data row2 col2" >$54.69</td> 
    </tr>    <tr> 
        <th id="T_9497a752_19f3_11e8_be45_74e54394bcc7level0_row3" class="row_heading level0 row3" >20-24</th> 
        <td id="T_9497a752_19f3_11e8_be45_74e54394bcc7row3_col0" class="data row3 col0" >33</td> 
        <td id="T_9497a752_19f3_11e8_be45_74e54394bcc7row3_col1" class="data row3 col1" >$3.04</td> 
        <td id="T_9497a752_19f3_11e8_be45_74e54394bcc7row3_col2" class="data row3 col2" >$100.42</td> 
    </tr>    <tr> 
        <th id="T_9497a752_19f3_11e8_be45_74e54394bcc7level0_row4" class="row_heading level0 row4" >25-29</th> 
        <td id="T_9497a752_19f3_11e8_be45_74e54394bcc7row4_col0" class="data row4 col0" >4</td> 
        <td id="T_9497a752_19f3_11e8_be45_74e54394bcc7row4_col1" class="data row4 col1" >$2.69</td> 
        <td id="T_9497a752_19f3_11e8_be45_74e54394bcc7row4_col2" class="data row4 col2" >$10.77</td> 
    </tr>    <tr> 
        <th id="T_9497a752_19f3_11e8_be45_74e54394bcc7level0_row5" class="row_heading level0 row5" >30-34</th> 
        <td id="T_9497a752_19f3_11e8_be45_74e54394bcc7row5_col0" class="data row5 col0" >7</td> 
        <td id="T_9497a752_19f3_11e8_be45_74e54394bcc7row5_col1" class="data row5 col1" >$2.35</td> 
        <td id="T_9497a752_19f3_11e8_be45_74e54394bcc7row5_col2" class="data row5 col2" >$16.47</td> 
    </tr>    <tr> 
        <th id="T_9497a752_19f3_11e8_be45_74e54394bcc7level0_row6" class="row_heading level0 row6" >35-39</th> 
        <td id="T_9497a752_19f3_11e8_be45_74e54394bcc7row6_col0" class="data row6 col0" >5</td> 
        <td id="T_9497a752_19f3_11e8_be45_74e54394bcc7row6_col1" class="data row6 col1" >$3.94</td> 
        <td id="T_9497a752_19f3_11e8_be45_74e54394bcc7row6_col2" class="data row6 col2" >$19.72</td> 
    </tr>    <tr> 
        <th id="T_9497a752_19f3_11e8_be45_74e54394bcc7level0_row7" class="row_heading level0 row7" >40+</th> 
        <td id="T_9497a752_19f3_11e8_be45_74e54394bcc7row7_col0" class="data row7 col0" >0</td> 
        <td id="T_9497a752_19f3_11e8_be45_74e54394bcc7row7_col1" class="data row7 col1" >$nan</td> 
        <td id="T_9497a752_19f3_11e8_be45_74e54394bcc7row7_col2" class="data row7 col2" >$nan</td> 
    </tr></tbody> 
</table> 



## Top Spenders


```python
df_purchase.groupby('SN')['Price'].agg(
    ['count', 'mean', 'sum']
).sort_values(
    by='sum', 
    ascending=False
).rename(
    columns={
        'count': 'Purchase Count',
        'mean': 'Average Purchase Price',
        'sum': 'Total Purchase Value'
    }
).head(5).style.format(
    {
        'Average Purchase Price': '${:,.2f}',
         'Total Purchase Value': '${:,.2f}',
    }
)
```




<style  type="text/css" >
</style>  
<table id="T_eb9c6bc0_19f4_11e8_bd1b_74e54394bcc7" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >Purchase Count</th> 
        <th class="col_heading level0 col1" >Average Purchase Price</th> 
        <th class="col_heading level0 col2" >Total Purchase Value</th> 
    </tr>    <tr> 
        <th class="index_name level0" >SN</th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_eb9c6bc0_19f4_11e8_bd1b_74e54394bcc7level0_row0" class="row_heading level0 row0" >Sundaky74</th> 
        <td id="T_eb9c6bc0_19f4_11e8_bd1b_74e54394bcc7row0_col0" class="data row0 col0" >2</td> 
        <td id="T_eb9c6bc0_19f4_11e8_bd1b_74e54394bcc7row0_col1" class="data row0 col1" >$3.71</td> 
        <td id="T_eb9c6bc0_19f4_11e8_bd1b_74e54394bcc7row0_col2" class="data row0 col2" >$7.41</td> 
    </tr>    <tr> 
        <th id="T_eb9c6bc0_19f4_11e8_bd1b_74e54394bcc7level0_row1" class="row_heading level0 row1" >Aidaira26</th> 
        <td id="T_eb9c6bc0_19f4_11e8_bd1b_74e54394bcc7row1_col0" class="data row1 col0" >2</td> 
        <td id="T_eb9c6bc0_19f4_11e8_bd1b_74e54394bcc7row1_col1" class="data row1 col1" >$2.56</td> 
        <td id="T_eb9c6bc0_19f4_11e8_bd1b_74e54394bcc7row1_col2" class="data row1 col2" >$5.13</td> 
    </tr>    <tr> 
        <th id="T_eb9c6bc0_19f4_11e8_bd1b_74e54394bcc7level0_row2" class="row_heading level0 row2" >Eusty71</th> 
        <td id="T_eb9c6bc0_19f4_11e8_bd1b_74e54394bcc7row2_col0" class="data row2 col0" >1</td> 
        <td id="T_eb9c6bc0_19f4_11e8_bd1b_74e54394bcc7row2_col1" class="data row2 col1" >$4.81</td> 
        <td id="T_eb9c6bc0_19f4_11e8_bd1b_74e54394bcc7row2_col2" class="data row2 col2" >$4.81</td> 
    </tr>    <tr> 
        <th id="T_eb9c6bc0_19f4_11e8_bd1b_74e54394bcc7level0_row3" class="row_heading level0 row3" >Chanirra64</th> 
        <td id="T_eb9c6bc0_19f4_11e8_bd1b_74e54394bcc7row3_col0" class="data row3 col0" >1</td> 
        <td id="T_eb9c6bc0_19f4_11e8_bd1b_74e54394bcc7row3_col1" class="data row3 col1" >$4.78</td> 
        <td id="T_eb9c6bc0_19f4_11e8_bd1b_74e54394bcc7row3_col2" class="data row3 col2" >$4.78</td> 
    </tr>    <tr> 
        <th id="T_eb9c6bc0_19f4_11e8_bd1b_74e54394bcc7level0_row4" class="row_heading level0 row4" >Alarap40</th> 
        <td id="T_eb9c6bc0_19f4_11e8_bd1b_74e54394bcc7row4_col0" class="data row4 col0" >1</td> 
        <td id="T_eb9c6bc0_19f4_11e8_bd1b_74e54394bcc7row4_col1" class="data row4 col1" >$4.71</td> 
        <td id="T_eb9c6bc0_19f4_11e8_bd1b_74e54394bcc7row4_col2" class="data row4 col2" >$4.71</td> 
    </tr></tbody> 
</table> 



## Most Popular Items


```python
df_purchase.groupby(['Item ID', 'Item Name'])['Price'].agg(
    ['count', 'mean', 'sum']
).sort_values(
    by='count', 
    ascending=False
).rename(
    columns={
        'count': 'Purchase Count',
        'mean': 'Average Purchase Price',
        'sum': 'Total Purchase Value'
    }
).head(5).style.format(
    {
        'Average Purchase Price': '${:,.2f}',
         'Total Purchase Value': '${:,.2f}',
    }
)
```




<style  type="text/css" >
</style>  
<table id="T_69b17166_19f5_11e8_9ce0_74e54394bcc7" > 
<thead>    <tr> 
        <th class="blank" ></th> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >Purchase Count</th> 
        <th class="col_heading level0 col1" >Average Purchase Price</th> 
        <th class="col_heading level0 col2" >Total Purchase Value</th> 
    </tr>    <tr> 
        <th class="index_name level0" >Item ID</th> 
        <th class="index_name level1" >Item Name</th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_69b17166_19f5_11e8_9ce0_74e54394bcc7level0_row0" class="row_heading level0 row0" >94</th> 
        <th id="T_69b17166_19f5_11e8_9ce0_74e54394bcc7level1_row0" class="row_heading level1 row0" >Mourning Blade</th> 
        <td id="T_69b17166_19f5_11e8_9ce0_74e54394bcc7row0_col0" class="data row0 col0" >3</td> 
        <td id="T_69b17166_19f5_11e8_9ce0_74e54394bcc7row0_col1" class="data row0 col1" >$3.64</td> 
        <td id="T_69b17166_19f5_11e8_9ce0_74e54394bcc7row0_col2" class="data row0 col2" >$10.92</td> 
    </tr>    <tr> 
        <th id="T_69b17166_19f5_11e8_9ce0_74e54394bcc7level0_row1" class="row_heading level0 row1" >90</th> 
        <th id="T_69b17166_19f5_11e8_9ce0_74e54394bcc7level1_row1" class="row_heading level1 row1" >Betrayer</th> 
        <td id="T_69b17166_19f5_11e8_9ce0_74e54394bcc7row1_col0" class="data row1 col0" >2</td> 
        <td id="T_69b17166_19f5_11e8_9ce0_74e54394bcc7row1_col1" class="data row1 col1" >$4.12</td> 
        <td id="T_69b17166_19f5_11e8_9ce0_74e54394bcc7row1_col2" class="data row1 col2" >$8.24</td> 
    </tr>    <tr> 
        <th id="T_69b17166_19f5_11e8_9ce0_74e54394bcc7level0_row2" class="row_heading level0 row2" >111</th> 
        <th id="T_69b17166_19f5_11e8_9ce0_74e54394bcc7level1_row2" class="row_heading level1 row2" >Misery's End</th> 
        <td id="T_69b17166_19f5_11e8_9ce0_74e54394bcc7row2_col0" class="data row2 col0" >2</td> 
        <td id="T_69b17166_19f5_11e8_9ce0_74e54394bcc7row2_col1" class="data row2 col1" >$1.79</td> 
        <td id="T_69b17166_19f5_11e8_9ce0_74e54394bcc7row2_col2" class="data row2 col2" >$3.58</td> 
    </tr>    <tr> 
        <th id="T_69b17166_19f5_11e8_9ce0_74e54394bcc7level0_row3" class="row_heading level0 row3" >64</th> 
        <th id="T_69b17166_19f5_11e8_9ce0_74e54394bcc7level1_row3" class="row_heading level1 row3" >Fusion Pummel</th> 
        <td id="T_69b17166_19f5_11e8_9ce0_74e54394bcc7row3_col0" class="data row3 col0" >2</td> 
        <td id="T_69b17166_19f5_11e8_9ce0_74e54394bcc7row3_col1" class="data row3 col1" >$2.42</td> 
        <td id="T_69b17166_19f5_11e8_9ce0_74e54394bcc7row3_col2" class="data row3 col2" >$4.84</td> 
    </tr>    <tr> 
        <th id="T_69b17166_19f5_11e8_9ce0_74e54394bcc7level0_row4" class="row_heading level0 row4" >154</th> 
        <th id="T_69b17166_19f5_11e8_9ce0_74e54394bcc7level1_row4" class="row_heading level1 row4" >Feral Katana</th> 
        <td id="T_69b17166_19f5_11e8_9ce0_74e54394bcc7row4_col0" class="data row4 col0" >2</td> 
        <td id="T_69b17166_19f5_11e8_9ce0_74e54394bcc7row4_col1" class="data row4 col1" >$4.11</td> 
        <td id="T_69b17166_19f5_11e8_9ce0_74e54394bcc7row4_col2" class="data row4 col2" >$8.22</td> 
    </tr></tbody> 
</table> 



## Most Porfitable Items


```python
df_purchase.groupby(['Item ID', 'Item Name'])['Price'].agg(
    ['count', 'mean', 'sum']
).sort_values(
    by='sum', 
    ascending=False
).rename(
    columns={
        'count': 'Purchase Count',
        'mean': 'Average Purchase Price',
        'sum': 'Total Purchase Value'
    }
).head(5).style.format(
    {
        'Average Purchase Price': '${:,.2f}',
         'Total Purchase Value': '${:,.2f}',
    }
)
```




<style  type="text/css" >
</style>  
<table id="T_7123520a_19f5_11e8_940d_74e54394bcc7" > 
<thead>    <tr> 
        <th class="blank" ></th> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >Purchase Count</th> 
        <th class="col_heading level0 col1" >Average Purchase Price</th> 
        <th class="col_heading level0 col2" >Total Purchase Value</th> 
    </tr>    <tr> 
        <th class="index_name level0" >Item ID</th> 
        <th class="index_name level1" >Item Name</th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_7123520a_19f5_11e8_940d_74e54394bcc7level0_row0" class="row_heading level0 row0" >94</th> 
        <th id="T_7123520a_19f5_11e8_940d_74e54394bcc7level1_row0" class="row_heading level1 row0" >Mourning Blade</th> 
        <td id="T_7123520a_19f5_11e8_940d_74e54394bcc7row0_col0" class="data row0 col0" >3</td> 
        <td id="T_7123520a_19f5_11e8_940d_74e54394bcc7row0_col1" class="data row0 col1" >$3.64</td> 
        <td id="T_7123520a_19f5_11e8_940d_74e54394bcc7row0_col2" class="data row0 col2" >$10.92</td> 
    </tr>    <tr> 
        <th id="T_7123520a_19f5_11e8_940d_74e54394bcc7level0_row1" class="row_heading level0 row1" >117</th> 
        <th id="T_7123520a_19f5_11e8_940d_74e54394bcc7level1_row1" class="row_heading level1 row1" >Heartstriker, Legacy of the Light</th> 
        <td id="T_7123520a_19f5_11e8_940d_74e54394bcc7row1_col0" class="data row1 col0" >2</td> 
        <td id="T_7123520a_19f5_11e8_940d_74e54394bcc7row1_col1" class="data row1 col1" >$4.71</td> 
        <td id="T_7123520a_19f5_11e8_940d_74e54394bcc7row1_col2" class="data row1 col2" >$9.42</td> 
    </tr>    <tr> 
        <th id="T_7123520a_19f5_11e8_940d_74e54394bcc7level0_row2" class="row_heading level0 row2" >93</th> 
        <th id="T_7123520a_19f5_11e8_940d_74e54394bcc7level1_row2" class="row_heading level1 row2" >Apocalyptic Battlescythe</th> 
        <td id="T_7123520a_19f5_11e8_940d_74e54394bcc7row2_col0" class="data row2 col0" >2</td> 
        <td id="T_7123520a_19f5_11e8_940d_74e54394bcc7row2_col1" class="data row2 col1" >$4.49</td> 
        <td id="T_7123520a_19f5_11e8_940d_74e54394bcc7row2_col2" class="data row2 col2" >$8.98</td> 
    </tr>    <tr> 
        <th id="T_7123520a_19f5_11e8_940d_74e54394bcc7level0_row3" class="row_heading level0 row3" >90</th> 
        <th id="T_7123520a_19f5_11e8_940d_74e54394bcc7level1_row3" class="row_heading level1 row3" >Betrayer</th> 
        <td id="T_7123520a_19f5_11e8_940d_74e54394bcc7row3_col0" class="data row3 col0" >2</td> 
        <td id="T_7123520a_19f5_11e8_940d_74e54394bcc7row3_col1" class="data row3 col1" >$4.12</td> 
        <td id="T_7123520a_19f5_11e8_940d_74e54394bcc7row3_col2" class="data row3 col2" >$8.24</td> 
    </tr>    <tr> 
        <th id="T_7123520a_19f5_11e8_940d_74e54394bcc7level0_row4" class="row_heading level0 row4" >154</th> 
        <th id="T_7123520a_19f5_11e8_940d_74e54394bcc7level1_row4" class="row_heading level1 row4" >Feral Katana</th> 
        <td id="T_7123520a_19f5_11e8_940d_74e54394bcc7row4_col0" class="data row4 col0" >2</td> 
        <td id="T_7123520a_19f5_11e8_940d_74e54394bcc7row4_col1" class="data row4 col1" >$4.11</td> 
        <td id="T_7123520a_19f5_11e8_940d_74e54394bcc7row4_col2" class="data row4 col2" >$8.22</td> 
    </tr></tbody> 
</table> 


