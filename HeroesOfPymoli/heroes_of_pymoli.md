
# Exploring Data Analsis of Purchase Data

## Trends
1. Majority of players are male
2. Majority of purchaes are made by players between 15 to 24 
3. Most items are only purchased once based on the data file. This might not be true if the data is just a sample.
4. Plyer age distribution is close to normal but slightly right skewed


```python
import os
from pandas import Series, DataFrame
import pandas as pd
import matplotlib.pyplot as plt

MONEY_FORMAT = '${:,.2f}'
PERC_FORMAT = '{:.2%}'
CNT_FORMAT = '{:,}'
FORMAT = {
    'Average Purchase Price': MONEY_FORMAT,
    'Total Purchase Value': MONEY_FORMAT,
    'Percenage of Players': '{:.2%}',
    'Normalized Total':  MONEY_FORMAT,
    'Average Price': MONEY_FORMAT,
    'Total Revenue': MONEY_FORMAT,
    'Total Players': CNT_FORMAT,
    'Number of Purchases': CNT_FORMAT
}
```

## Load to DataFrame


```python
DATA_PATH = '.'
file = input("What's the input file?")
full_path = os.path.join(DATA_PATH, file)
df_purchase = pd.read_json(full_path)
```

    What's the input file?purchase_data.json
    

## Player Count

The SN looks like can uniquely identify a player so get the unique count of it.


```python
DataFrame( 
    [ 
        {
            'Total Players': len(df_purchase['SN'].unique()),
        }
    ]
)
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
      <th>Total Players</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>573</td>
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
).style.format(FORMAT)
```




<style  type="text/css" >
</style>  
<table id="T_dff410ee_1b6e_11e8_a25b_74e54394bcc7" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >Number of Unique Items</th> 
        <th class="col_heading level0 col1" >Average Price</th> 
        <th class="col_heading level0 col2" >Number of Purchases</th> 
        <th class="col_heading level0 col3" >Total Revenue</th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_dff410ee_1b6e_11e8_a25b_74e54394bcc7level0_row0" class="row_heading level0 row0" >0</th> 
        <td id="T_dff410ee_1b6e_11e8_a25b_74e54394bcc7row0_col0" class="data row0 col0" >183</td> 
        <td id="T_dff410ee_1b6e_11e8_a25b_74e54394bcc7row0_col1" class="data row0 col1" >$2.93</td> 
        <td id="T_dff410ee_1b6e_11e8_a25b_74e54394bcc7row0_col2" class="data row0 col2" >780</td> 
        <td id="T_dff410ee_1b6e_11e8_a25b_74e54394bcc7row0_col3" class="data row0 col3" >$2,286.33</td> 
    </tr></tbody> 
</table> 



## Gender Demographics


```python
def report_demographics(df, groupby):
    '''
    Report on demographics summary
    
    :param df: DataFrame input
    :param groupby: Summarized by field
    :return: Formatted output
    '''
    return DataFrame(
        df.groupby(groupby)['SN'].unique()
    ).assign(
        cnt = lambda x: x['SN'].map(len)
    ).assign(
        perc=lambda x: x['cnt'] / len(df['SN'].unique())
    ).drop(
        'SN', axis=1
    ).rename(
        columns = {
            'cnt': 'Toatl Count',
            'perc': 'Percenage of Players'
        }
    ).style.format(FORMAT)
```


```python
report_demographics(df_purchase, 'Gender')
```




<style  type="text/css" >
</style>  
<table id="T_e00612a2_1b6e_11e8_a975_74e54394bcc7" > 
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
        <th id="T_e00612a2_1b6e_11e8_a975_74e54394bcc7level0_row0" class="row_heading level0 row0" >Female</th> 
        <td id="T_e00612a2_1b6e_11e8_a975_74e54394bcc7row0_col0" class="data row0 col0" >100</td> 
        <td id="T_e00612a2_1b6e_11e8_a975_74e54394bcc7row0_col1" class="data row0 col1" >17.45%</td> 
    </tr>    <tr> 
        <th id="T_e00612a2_1b6e_11e8_a975_74e54394bcc7level0_row1" class="row_heading level0 row1" >Male</th> 
        <td id="T_e00612a2_1b6e_11e8_a975_74e54394bcc7row1_col0" class="data row1 col0" >465</td> 
        <td id="T_e00612a2_1b6e_11e8_a975_74e54394bcc7row1_col1" class="data row1 col1" >81.15%</td> 
    </tr>    <tr> 
        <th id="T_e00612a2_1b6e_11e8_a975_74e54394bcc7level0_row2" class="row_heading level0 row2" >Other / Non-Disclosed</th> 
        <td id="T_e00612a2_1b6e_11e8_a975_74e54394bcc7row2_col0" class="data row2 col0" >8</td> 
        <td id="T_e00612a2_1b6e_11e8_a975_74e54394bcc7row2_col1" class="data row2 col1" >1.40%</td> 
    </tr></tbody> 
</table> 



## Purchasing Analysis (Gender)
**What the hell is 'Normalized Total'?**

Normalized totals is a very strange name but from the sample chart, it is mostly the same as average purchase per item but some times lightly higer, and when it is higher there are some player purchse multiple items, so I guess it is actually the purchase value per player.


```python
def report_analysis(df, groupby):
    '''
     Generate report of price statistics
    
    :param df: DataFrame input
    :param groupby: Summarized by field
    :return: Formatted output
    '''
    count_unique = lambda x: len(x.unique())
    count_unique.__name__ = 'count_unique'
    df_summary = df.groupby(groupby)['Price', 'SN'].agg(
        {
            'Price': ['count', 'mean', 'sum'],
            'SN': count_unique
        }
    )
 
    # Flattern multi level index for easier selection
    df_summary.columns = ['_'.join(x) for x in df_summary.columns]
    
    return df_summary.assign(
        normalized_total=lambda x: x['Price_sum']/x['SN_count_unique']
    ).drop(
        'SN_count_unique', axis=1
    ).rename(
        columns = {
            'Price_count': 'Purchase Count',
            'Price_mean': 'Average Purchase Price',
            'Price_sum': 'Total Purchase Value',
            'normalized_total': 'Normalized Total',
        }
    ).style.format(FORMAT)
```


```python
report_analysis(df_purchase, 'Gender')
```




<style  type="text/css" >
</style>  
<table id="T_e0183ae2_1b6e_11e8_8a6d_74e54394bcc7" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >Purchase Count</th> 
        <th class="col_heading level0 col1" >Average Purchase Price</th> 
        <th class="col_heading level0 col2" >Total Purchase Value</th> 
        <th class="col_heading level0 col3" >Normalized Total</th> 
    </tr>    <tr> 
        <th class="index_name level0" >Gender</th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_e0183ae2_1b6e_11e8_8a6d_74e54394bcc7level0_row0" class="row_heading level0 row0" >Female</th> 
        <td id="T_e0183ae2_1b6e_11e8_8a6d_74e54394bcc7row0_col0" class="data row0 col0" >136</td> 
        <td id="T_e0183ae2_1b6e_11e8_8a6d_74e54394bcc7row0_col1" class="data row0 col1" >$2.82</td> 
        <td id="T_e0183ae2_1b6e_11e8_8a6d_74e54394bcc7row0_col2" class="data row0 col2" >$382.91</td> 
        <td id="T_e0183ae2_1b6e_11e8_8a6d_74e54394bcc7row0_col3" class="data row0 col3" >$3.83</td> 
    </tr>    <tr> 
        <th id="T_e0183ae2_1b6e_11e8_8a6d_74e54394bcc7level0_row1" class="row_heading level0 row1" >Male</th> 
        <td id="T_e0183ae2_1b6e_11e8_8a6d_74e54394bcc7row1_col0" class="data row1 col0" >633</td> 
        <td id="T_e0183ae2_1b6e_11e8_8a6d_74e54394bcc7row1_col1" class="data row1 col1" >$2.95</td> 
        <td id="T_e0183ae2_1b6e_11e8_8a6d_74e54394bcc7row1_col2" class="data row1 col2" >$1,867.68</td> 
        <td id="T_e0183ae2_1b6e_11e8_8a6d_74e54394bcc7row1_col3" class="data row1 col3" >$4.02</td> 
    </tr>    <tr> 
        <th id="T_e0183ae2_1b6e_11e8_8a6d_74e54394bcc7level0_row2" class="row_heading level0 row2" >Other / Non-Disclosed</th> 
        <td id="T_e0183ae2_1b6e_11e8_8a6d_74e54394bcc7row2_col0" class="data row2 col0" >11</td> 
        <td id="T_e0183ae2_1b6e_11e8_8a6d_74e54394bcc7row2_col1" class="data row2 col1" >$3.25</td> 
        <td id="T_e0183ae2_1b6e_11e8_8a6d_74e54394bcc7row2_col2" class="data row2 col2" >$35.74</td> 
        <td id="T_e0183ae2_1b6e_11e8_8a6d_74e54394bcc7row2_col3" class="data row2 col3" >$4.47</td> 
    </tr></tbody> 
</table> 



## Age Demographics

The sample PDF ashow both Age Demographics and Purchasing Analysis (Age) so I did both.


```python
bin_start = 10
bins = [0]
max_age = df_purchase.Age.max() 
bins.extend(range(bin_start, max_age, 5))
bins.append(max_age + 1)

labels = ['{} to {}'.format(x,y-1) for x,y in zip(bins, bins[1:])]

df_purchase.loc[:, 'Age Range'] = pd.cut(df_purchase['Age'], bins=bins, labels=labels)

report_demographics(df_purchase, 'Age Range')
```




<style  type="text/css" >
</style>  
<table id="T_e025a8be_1b6e_11e8_a178_74e54394bcc7" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >Toatl Count</th> 
        <th class="col_heading level0 col1" >Percenage of Players</th> 
    </tr>    <tr> 
        <th class="index_name level0" >Age Range</th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_e025a8be_1b6e_11e8_a178_74e54394bcc7level0_row0" class="row_heading level0 row0" >0 to 9</th> 
        <td id="T_e025a8be_1b6e_11e8_a178_74e54394bcc7row0_col0" class="data row0 col0" >22</td> 
        <td id="T_e025a8be_1b6e_11e8_a178_74e54394bcc7row0_col1" class="data row0 col1" >3.84%</td> 
    </tr>    <tr> 
        <th id="T_e025a8be_1b6e_11e8_a178_74e54394bcc7level0_row1" class="row_heading level0 row1" >10 to 14</th> 
        <td id="T_e025a8be_1b6e_11e8_a178_74e54394bcc7row1_col0" class="data row1 col0" >54</td> 
        <td id="T_e025a8be_1b6e_11e8_a178_74e54394bcc7row1_col1" class="data row1 col1" >9.42%</td> 
    </tr>    <tr> 
        <th id="T_e025a8be_1b6e_11e8_a178_74e54394bcc7level0_row2" class="row_heading level0 row2" >15 to 19</th> 
        <td id="T_e025a8be_1b6e_11e8_a178_74e54394bcc7row2_col0" class="data row2 col0" >139</td> 
        <td id="T_e025a8be_1b6e_11e8_a178_74e54394bcc7row2_col1" class="data row2 col1" >24.26%</td> 
    </tr>    <tr> 
        <th id="T_e025a8be_1b6e_11e8_a178_74e54394bcc7level0_row3" class="row_heading level0 row3" >20 to 24</th> 
        <td id="T_e025a8be_1b6e_11e8_a178_74e54394bcc7row3_col0" class="data row3 col0" >234</td> 
        <td id="T_e025a8be_1b6e_11e8_a178_74e54394bcc7row3_col1" class="data row3 col1" >40.84%</td> 
    </tr>    <tr> 
        <th id="T_e025a8be_1b6e_11e8_a178_74e54394bcc7level0_row4" class="row_heading level0 row4" >25 to 29</th> 
        <td id="T_e025a8be_1b6e_11e8_a178_74e54394bcc7row4_col0" class="data row4 col0" >52</td> 
        <td id="T_e025a8be_1b6e_11e8_a178_74e54394bcc7row4_col1" class="data row4 col1" >9.08%</td> 
    </tr>    <tr> 
        <th id="T_e025a8be_1b6e_11e8_a178_74e54394bcc7level0_row5" class="row_heading level0 row5" >30 to 34</th> 
        <td id="T_e025a8be_1b6e_11e8_a178_74e54394bcc7row5_col0" class="data row5 col0" >44</td> 
        <td id="T_e025a8be_1b6e_11e8_a178_74e54394bcc7row5_col1" class="data row5 col1" >7.68%</td> 
    </tr>    <tr> 
        <th id="T_e025a8be_1b6e_11e8_a178_74e54394bcc7level0_row6" class="row_heading level0 row6" >35 to 39</th> 
        <td id="T_e025a8be_1b6e_11e8_a178_74e54394bcc7row6_col0" class="data row6 col0" >25</td> 
        <td id="T_e025a8be_1b6e_11e8_a178_74e54394bcc7row6_col1" class="data row6 col1" >4.36%</td> 
    </tr>    <tr> 
        <th id="T_e025a8be_1b6e_11e8_a178_74e54394bcc7level0_row7" class="row_heading level0 row7" >40 to 45</th> 
        <td id="T_e025a8be_1b6e_11e8_a178_74e54394bcc7row7_col0" class="data row7 col0" >3</td> 
        <td id="T_e025a8be_1b6e_11e8_a178_74e54394bcc7row7_col1" class="data row7 col1" >0.52%</td> 
    </tr></tbody> 
</table> 



## Purchasing Analysis (Age)


```python
report_analysis(df_purchase, 'Age Range')
```




<style  type="text/css" >
</style>  
<table id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >Purchase Count</th> 
        <th class="col_heading level0 col1" >Average Purchase Price</th> 
        <th class="col_heading level0 col2" >Total Purchase Value</th> 
        <th class="col_heading level0 col3" >Normalized Total</th> 
    </tr>    <tr> 
        <th class="index_name level0" >Age Range</th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7level0_row0" class="row_heading level0 row0" >0 to 9</th> 
        <td id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7row0_col0" class="data row0 col0" >32</td> 
        <td id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7row0_col1" class="data row0 col1" >$3.02</td> 
        <td id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7row0_col2" class="data row0 col2" >$96.62</td> 
        <td id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7row0_col3" class="data row0 col3" >$4.39</td> 
    </tr>    <tr> 
        <th id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7level0_row1" class="row_heading level0 row1" >10 to 14</th> 
        <td id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7row1_col0" class="data row1 col0" >78</td> 
        <td id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7row1_col1" class="data row1 col1" >$2.87</td> 
        <td id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7row1_col2" class="data row1 col2" >$224.15</td> 
        <td id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7row1_col3" class="data row1 col3" >$4.15</td> 
    </tr>    <tr> 
        <th id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7level0_row2" class="row_heading level0 row2" >15 to 19</th> 
        <td id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7row2_col0" class="data row2 col0" >184</td> 
        <td id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7row2_col1" class="data row2 col1" >$2.87</td> 
        <td id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7row2_col2" class="data row2 col2" >$528.74</td> 
        <td id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7row2_col3" class="data row2 col3" >$3.80</td> 
    </tr>    <tr> 
        <th id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7level0_row3" class="row_heading level0 row3" >20 to 24</th> 
        <td id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7row3_col0" class="data row3 col0" >305</td> 
        <td id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7row3_col1" class="data row3 col1" >$2.96</td> 
        <td id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7row3_col2" class="data row3 col2" >$902.61</td> 
        <td id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7row3_col3" class="data row3 col3" >$3.86</td> 
    </tr>    <tr> 
        <th id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7level0_row4" class="row_heading level0 row4" >25 to 29</th> 
        <td id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7row4_col0" class="data row4 col0" >76</td> 
        <td id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7row4_col1" class="data row4 col1" >$2.89</td> 
        <td id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7row4_col2" class="data row4 col2" >$219.82</td> 
        <td id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7row4_col3" class="data row4 col3" >$4.23</td> 
    </tr>    <tr> 
        <th id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7level0_row5" class="row_heading level0 row5" >30 to 34</th> 
        <td id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7row5_col0" class="data row5 col0" >58</td> 
        <td id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7row5_col1" class="data row5 col1" >$3.07</td> 
        <td id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7row5_col2" class="data row5 col2" >$178.26</td> 
        <td id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7row5_col3" class="data row5 col3" >$4.05</td> 
    </tr>    <tr> 
        <th id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7level0_row6" class="row_heading level0 row6" >35 to 39</th> 
        <td id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7row6_col0" class="data row6 col0" >44</td> 
        <td id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7row6_col1" class="data row6 col1" >$2.90</td> 
        <td id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7row6_col2" class="data row6 col2" >$127.49</td> 
        <td id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7row6_col3" class="data row6 col3" >$5.10</td> 
    </tr>    <tr> 
        <th id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7level0_row7" class="row_heading level0 row7" >40 to 45</th> 
        <td id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7row7_col0" class="data row7 col0" >3</td> 
        <td id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7row7_col1" class="data row7 col1" >$2.88</td> 
        <td id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7row7_col2" class="data row7 col2" >$8.64</td> 
        <td id="T_e03141ae_1b6e_11e8_9d91_74e54394bcc7row7_col3" class="data row7 col3" >$2.88</td> 
    </tr></tbody> 
</table> 



## Top Spenders


```python
def report_top(df, groupby, by='sum', top=5):
    '''
    Report on top items. Default to 'sum' statistics, and top 5.
    
    :param df: DataFrame input
    :param groupby: Summarized by field
    :param by: Summary statistics
    :param top: How many top items
    :return: Formatted output
    '''
    return df.groupby(groupby)['Price'].agg(
        ['count', 'mean', 'sum']
    ).sort_values(
        by=by, 
        ascending=False
    ).rename(
        columns={
            'count': 'Purchase Count',
            'mean': 'Average Purchase Price',
            'sum': 'Total Purchase Value'
        }
    ).head(top).style.format(FORMAT)
```


```python
report_top(df_purchase, 'SN')
```




<style  type="text/css" >
</style>  
<table id="T_e047b010_1b6e_11e8_b964_74e54394bcc7" > 
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
        <th id="T_e047b010_1b6e_11e8_b964_74e54394bcc7level0_row0" class="row_heading level0 row0" >Undirrala66</th> 
        <td id="T_e047b010_1b6e_11e8_b964_74e54394bcc7row0_col0" class="data row0 col0" >5</td> 
        <td id="T_e047b010_1b6e_11e8_b964_74e54394bcc7row0_col1" class="data row0 col1" >$3.41</td> 
        <td id="T_e047b010_1b6e_11e8_b964_74e54394bcc7row0_col2" class="data row0 col2" >$17.06</td> 
    </tr>    <tr> 
        <th id="T_e047b010_1b6e_11e8_b964_74e54394bcc7level0_row1" class="row_heading level0 row1" >Saedue76</th> 
        <td id="T_e047b010_1b6e_11e8_b964_74e54394bcc7row1_col0" class="data row1 col0" >4</td> 
        <td id="T_e047b010_1b6e_11e8_b964_74e54394bcc7row1_col1" class="data row1 col1" >$3.39</td> 
        <td id="T_e047b010_1b6e_11e8_b964_74e54394bcc7row1_col2" class="data row1 col2" >$13.56</td> 
    </tr>    <tr> 
        <th id="T_e047b010_1b6e_11e8_b964_74e54394bcc7level0_row2" class="row_heading level0 row2" >Mindimnya67</th> 
        <td id="T_e047b010_1b6e_11e8_b964_74e54394bcc7row2_col0" class="data row2 col0" >4</td> 
        <td id="T_e047b010_1b6e_11e8_b964_74e54394bcc7row2_col1" class="data row2 col1" >$3.18</td> 
        <td id="T_e047b010_1b6e_11e8_b964_74e54394bcc7row2_col2" class="data row2 col2" >$12.74</td> 
    </tr>    <tr> 
        <th id="T_e047b010_1b6e_11e8_b964_74e54394bcc7level0_row3" class="row_heading level0 row3" >Haellysu29</th> 
        <td id="T_e047b010_1b6e_11e8_b964_74e54394bcc7row3_col0" class="data row3 col0" >3</td> 
        <td id="T_e047b010_1b6e_11e8_b964_74e54394bcc7row3_col1" class="data row3 col1" >$4.24</td> 
        <td id="T_e047b010_1b6e_11e8_b964_74e54394bcc7row3_col2" class="data row3 col2" >$12.73</td> 
    </tr>    <tr> 
        <th id="T_e047b010_1b6e_11e8_b964_74e54394bcc7level0_row4" class="row_heading level0 row4" >Eoda93</th> 
        <td id="T_e047b010_1b6e_11e8_b964_74e54394bcc7row4_col0" class="data row4 col0" >3</td> 
        <td id="T_e047b010_1b6e_11e8_b964_74e54394bcc7row4_col1" class="data row4 col1" >$3.86</td> 
        <td id="T_e047b010_1b6e_11e8_b964_74e54394bcc7row4_col2" class="data row4 col2" >$11.58</td> 
    </tr></tbody> 
</table> 



## Most Popular Items


```python
report_top(df_purchase, ['Item ID', 'Item Name'], by='count')
```




<style  type="text/css" >
</style>  
<table id="T_e0517436_1b6e_11e8_a02c_74e54394bcc7" > 
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
        <th id="T_e0517436_1b6e_11e8_a02c_74e54394bcc7level0_row0" class="row_heading level0 row0" >39</th> 
        <th id="T_e0517436_1b6e_11e8_a02c_74e54394bcc7level1_row0" class="row_heading level1 row0" >Betrayal, Whisper of Grieving Widows</th> 
        <td id="T_e0517436_1b6e_11e8_a02c_74e54394bcc7row0_col0" class="data row0 col0" >11</td> 
        <td id="T_e0517436_1b6e_11e8_a02c_74e54394bcc7row0_col1" class="data row0 col1" >$2.35</td> 
        <td id="T_e0517436_1b6e_11e8_a02c_74e54394bcc7row0_col2" class="data row0 col2" >$25.85</td> 
    </tr>    <tr> 
        <th id="T_e0517436_1b6e_11e8_a02c_74e54394bcc7level0_row1" class="row_heading level0 row1" >84</th> 
        <th id="T_e0517436_1b6e_11e8_a02c_74e54394bcc7level1_row1" class="row_heading level1 row1" >Arcane Gem</th> 
        <td id="T_e0517436_1b6e_11e8_a02c_74e54394bcc7row1_col0" class="data row1 col0" >11</td> 
        <td id="T_e0517436_1b6e_11e8_a02c_74e54394bcc7row1_col1" class="data row1 col1" >$2.23</td> 
        <td id="T_e0517436_1b6e_11e8_a02c_74e54394bcc7row1_col2" class="data row1 col2" >$24.53</td> 
    </tr>    <tr> 
        <th id="T_e0517436_1b6e_11e8_a02c_74e54394bcc7level0_row2" class="row_heading level0 row2" >31</th> 
        <th id="T_e0517436_1b6e_11e8_a02c_74e54394bcc7level1_row2" class="row_heading level1 row2" >Trickster</th> 
        <td id="T_e0517436_1b6e_11e8_a02c_74e54394bcc7row2_col0" class="data row2 col0" >9</td> 
        <td id="T_e0517436_1b6e_11e8_a02c_74e54394bcc7row2_col1" class="data row2 col1" >$2.07</td> 
        <td id="T_e0517436_1b6e_11e8_a02c_74e54394bcc7row2_col2" class="data row2 col2" >$18.63</td> 
    </tr>    <tr> 
        <th id="T_e0517436_1b6e_11e8_a02c_74e54394bcc7level0_row3" class="row_heading level0 row3" >175</th> 
        <th id="T_e0517436_1b6e_11e8_a02c_74e54394bcc7level1_row3" class="row_heading level1 row3" >Woeful Adamantite Claymore</th> 
        <td id="T_e0517436_1b6e_11e8_a02c_74e54394bcc7row3_col0" class="data row3 col0" >9</td> 
        <td id="T_e0517436_1b6e_11e8_a02c_74e54394bcc7row3_col1" class="data row3 col1" >$1.24</td> 
        <td id="T_e0517436_1b6e_11e8_a02c_74e54394bcc7row3_col2" class="data row3 col2" >$11.16</td> 
    </tr>    <tr> 
        <th id="T_e0517436_1b6e_11e8_a02c_74e54394bcc7level0_row4" class="row_heading level0 row4" >13</th> 
        <th id="T_e0517436_1b6e_11e8_a02c_74e54394bcc7level1_row4" class="row_heading level1 row4" >Serenity</th> 
        <td id="T_e0517436_1b6e_11e8_a02c_74e54394bcc7row4_col0" class="data row4 col0" >9</td> 
        <td id="T_e0517436_1b6e_11e8_a02c_74e54394bcc7row4_col1" class="data row4 col1" >$1.49</td> 
        <td id="T_e0517436_1b6e_11e8_a02c_74e54394bcc7row4_col2" class="data row4 col2" >$13.41</td> 
    </tr></tbody> 
</table> 



## Most Porfitable Items


```python
report_top(df_purchase, ['Item ID', 'Item Name'])
```




<style  type="text/css" >
</style>  
<table id="T_e058ee4c_1b6e_11e8_9fd7_74e54394bcc7" > 
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
        <th id="T_e058ee4c_1b6e_11e8_9fd7_74e54394bcc7level0_row0" class="row_heading level0 row0" >34</th> 
        <th id="T_e058ee4c_1b6e_11e8_9fd7_74e54394bcc7level1_row0" class="row_heading level1 row0" >Retribution Axe</th> 
        <td id="T_e058ee4c_1b6e_11e8_9fd7_74e54394bcc7row0_col0" class="data row0 col0" >9</td> 
        <td id="T_e058ee4c_1b6e_11e8_9fd7_74e54394bcc7row0_col1" class="data row0 col1" >$4.14</td> 
        <td id="T_e058ee4c_1b6e_11e8_9fd7_74e54394bcc7row0_col2" class="data row0 col2" >$37.26</td> 
    </tr>    <tr> 
        <th id="T_e058ee4c_1b6e_11e8_9fd7_74e54394bcc7level0_row1" class="row_heading level0 row1" >115</th> 
        <th id="T_e058ee4c_1b6e_11e8_9fd7_74e54394bcc7level1_row1" class="row_heading level1 row1" >Spectral Diamond Doomblade</th> 
        <td id="T_e058ee4c_1b6e_11e8_9fd7_74e54394bcc7row1_col0" class="data row1 col0" >7</td> 
        <td id="T_e058ee4c_1b6e_11e8_9fd7_74e54394bcc7row1_col1" class="data row1 col1" >$4.25</td> 
        <td id="T_e058ee4c_1b6e_11e8_9fd7_74e54394bcc7row1_col2" class="data row1 col2" >$29.75</td> 
    </tr>    <tr> 
        <th id="T_e058ee4c_1b6e_11e8_9fd7_74e54394bcc7level0_row2" class="row_heading level0 row2" >32</th> 
        <th id="T_e058ee4c_1b6e_11e8_9fd7_74e54394bcc7level1_row2" class="row_heading level1 row2" >Orenmir</th> 
        <td id="T_e058ee4c_1b6e_11e8_9fd7_74e54394bcc7row2_col0" class="data row2 col0" >6</td> 
        <td id="T_e058ee4c_1b6e_11e8_9fd7_74e54394bcc7row2_col1" class="data row2 col1" >$4.95</td> 
        <td id="T_e058ee4c_1b6e_11e8_9fd7_74e54394bcc7row2_col2" class="data row2 col2" >$29.70</td> 
    </tr>    <tr> 
        <th id="T_e058ee4c_1b6e_11e8_9fd7_74e54394bcc7level0_row3" class="row_heading level0 row3" >103</th> 
        <th id="T_e058ee4c_1b6e_11e8_9fd7_74e54394bcc7level1_row3" class="row_heading level1 row3" >Singed Scalpel</th> 
        <td id="T_e058ee4c_1b6e_11e8_9fd7_74e54394bcc7row3_col0" class="data row3 col0" >6</td> 
        <td id="T_e058ee4c_1b6e_11e8_9fd7_74e54394bcc7row3_col1" class="data row3 col1" >$4.87</td> 
        <td id="T_e058ee4c_1b6e_11e8_9fd7_74e54394bcc7row3_col2" class="data row3 col2" >$29.22</td> 
    </tr>    <tr> 
        <th id="T_e058ee4c_1b6e_11e8_9fd7_74e54394bcc7level0_row4" class="row_heading level0 row4" >107</th> 
        <th id="T_e058ee4c_1b6e_11e8_9fd7_74e54394bcc7level1_row4" class="row_heading level1 row4" >Splitter, Foe Of Subtlety</th> 
        <td id="T_e058ee4c_1b6e_11e8_9fd7_74e54394bcc7row4_col0" class="data row4 col0" >8</td> 
        <td id="T_e058ee4c_1b6e_11e8_9fd7_74e54394bcc7row4_col1" class="data row4 col1" >$3.61</td> 
        <td id="T_e058ee4c_1b6e_11e8_9fd7_74e54394bcc7row4_col2" class="data row4 col2" >$28.88</td> 
    </tr></tbody> 
</table> 


