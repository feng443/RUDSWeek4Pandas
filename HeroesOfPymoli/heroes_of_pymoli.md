
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
<table id="T_ccc890cc_1b6f_11e8_bc0d_74e54394bcc7" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >Number of Unique Items</th> 
        <th class="col_heading level0 col1" >Average Price</th> 
        <th class="col_heading level0 col2" >Number of Purchases</th> 
        <th class="col_heading level0 col3" >Total Revenue</th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_ccc890cc_1b6f_11e8_bc0d_74e54394bcc7level0_row0" class="row_heading level0 row0" >0</th> 
        <td id="T_ccc890cc_1b6f_11e8_bc0d_74e54394bcc7row0_col0" class="data row0 col0" >183</td> 
        <td id="T_ccc890cc_1b6f_11e8_bc0d_74e54394bcc7row0_col1" class="data row0 col1" >$2.93</td> 
        <td id="T_ccc890cc_1b6f_11e8_bc0d_74e54394bcc7row0_col2" class="data row0 col2" >780</td> 
        <td id="T_ccc890cc_1b6f_11e8_bc0d_74e54394bcc7row0_col3" class="data row0 col3" >$2,286.33</td> 
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
<table id="T_cd951a3a_1b6f_11e8_af7a_74e54394bcc7" > 
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
        <th id="T_cd951a3a_1b6f_11e8_af7a_74e54394bcc7level0_row0" class="row_heading level0 row0" >Female</th> 
        <td id="T_cd951a3a_1b6f_11e8_af7a_74e54394bcc7row0_col0" class="data row0 col0" >100</td> 
        <td id="T_cd951a3a_1b6f_11e8_af7a_74e54394bcc7row0_col1" class="data row0 col1" >17.45%</td> 
    </tr>    <tr> 
        <th id="T_cd951a3a_1b6f_11e8_af7a_74e54394bcc7level0_row1" class="row_heading level0 row1" >Male</th> 
        <td id="T_cd951a3a_1b6f_11e8_af7a_74e54394bcc7row1_col0" class="data row1 col0" >465</td> 
        <td id="T_cd951a3a_1b6f_11e8_af7a_74e54394bcc7row1_col1" class="data row1 col1" >81.15%</td> 
    </tr>    <tr> 
        <th id="T_cd951a3a_1b6f_11e8_af7a_74e54394bcc7level0_row2" class="row_heading level0 row2" >Other / Non-Disclosed</th> 
        <td id="T_cd951a3a_1b6f_11e8_af7a_74e54394bcc7row2_col0" class="data row2 col0" >8</td> 
        <td id="T_cd951a3a_1b6f_11e8_af7a_74e54394bcc7row2_col1" class="data row2 col1" >1.40%</td> 
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
<table id="T_ce747692_1b6f_11e8_a845_74e54394bcc7" > 
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
        <th id="T_ce747692_1b6f_11e8_a845_74e54394bcc7level0_row0" class="row_heading level0 row0" >Female</th> 
        <td id="T_ce747692_1b6f_11e8_a845_74e54394bcc7row0_col0" class="data row0 col0" >136</td> 
        <td id="T_ce747692_1b6f_11e8_a845_74e54394bcc7row0_col1" class="data row0 col1" >$2.82</td> 
        <td id="T_ce747692_1b6f_11e8_a845_74e54394bcc7row0_col2" class="data row0 col2" >$382.91</td> 
        <td id="T_ce747692_1b6f_11e8_a845_74e54394bcc7row0_col3" class="data row0 col3" >$3.83</td> 
    </tr>    <tr> 
        <th id="T_ce747692_1b6f_11e8_a845_74e54394bcc7level0_row1" class="row_heading level0 row1" >Male</th> 
        <td id="T_ce747692_1b6f_11e8_a845_74e54394bcc7row1_col0" class="data row1 col0" >633</td> 
        <td id="T_ce747692_1b6f_11e8_a845_74e54394bcc7row1_col1" class="data row1 col1" >$2.95</td> 
        <td id="T_ce747692_1b6f_11e8_a845_74e54394bcc7row1_col2" class="data row1 col2" >$1,867.68</td> 
        <td id="T_ce747692_1b6f_11e8_a845_74e54394bcc7row1_col3" class="data row1 col3" >$4.02</td> 
    </tr>    <tr> 
        <th id="T_ce747692_1b6f_11e8_a845_74e54394bcc7level0_row2" class="row_heading level0 row2" >Other / Non-Disclosed</th> 
        <td id="T_ce747692_1b6f_11e8_a845_74e54394bcc7row2_col0" class="data row2 col0" >11</td> 
        <td id="T_ce747692_1b6f_11e8_a845_74e54394bcc7row2_col1" class="data row2 col1" >$3.25</td> 
        <td id="T_ce747692_1b6f_11e8_a845_74e54394bcc7row2_col2" class="data row2 col2" >$35.74</td> 
        <td id="T_ce747692_1b6f_11e8_a845_74e54394bcc7row2_col3" class="data row2 col3" >$4.47</td> 
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
<table id="T_cefd5eb6_1b6f_11e8_b083_74e54394bcc7" > 
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
        <th id="T_cefd5eb6_1b6f_11e8_b083_74e54394bcc7level0_row0" class="row_heading level0 row0" >0 to 9</th> 
        <td id="T_cefd5eb6_1b6f_11e8_b083_74e54394bcc7row0_col0" class="data row0 col0" >22</td> 
        <td id="T_cefd5eb6_1b6f_11e8_b083_74e54394bcc7row0_col1" class="data row0 col1" >3.84%</td> 
    </tr>    <tr> 
        <th id="T_cefd5eb6_1b6f_11e8_b083_74e54394bcc7level0_row1" class="row_heading level0 row1" >10 to 14</th> 
        <td id="T_cefd5eb6_1b6f_11e8_b083_74e54394bcc7row1_col0" class="data row1 col0" >54</td> 
        <td id="T_cefd5eb6_1b6f_11e8_b083_74e54394bcc7row1_col1" class="data row1 col1" >9.42%</td> 
    </tr>    <tr> 
        <th id="T_cefd5eb6_1b6f_11e8_b083_74e54394bcc7level0_row2" class="row_heading level0 row2" >15 to 19</th> 
        <td id="T_cefd5eb6_1b6f_11e8_b083_74e54394bcc7row2_col0" class="data row2 col0" >139</td> 
        <td id="T_cefd5eb6_1b6f_11e8_b083_74e54394bcc7row2_col1" class="data row2 col1" >24.26%</td> 
    </tr>    <tr> 
        <th id="T_cefd5eb6_1b6f_11e8_b083_74e54394bcc7level0_row3" class="row_heading level0 row3" >20 to 24</th> 
        <td id="T_cefd5eb6_1b6f_11e8_b083_74e54394bcc7row3_col0" class="data row3 col0" >234</td> 
        <td id="T_cefd5eb6_1b6f_11e8_b083_74e54394bcc7row3_col1" class="data row3 col1" >40.84%</td> 
    </tr>    <tr> 
        <th id="T_cefd5eb6_1b6f_11e8_b083_74e54394bcc7level0_row4" class="row_heading level0 row4" >25 to 29</th> 
        <td id="T_cefd5eb6_1b6f_11e8_b083_74e54394bcc7row4_col0" class="data row4 col0" >52</td> 
        <td id="T_cefd5eb6_1b6f_11e8_b083_74e54394bcc7row4_col1" class="data row4 col1" >9.08%</td> 
    </tr>    <tr> 
        <th id="T_cefd5eb6_1b6f_11e8_b083_74e54394bcc7level0_row5" class="row_heading level0 row5" >30 to 34</th> 
        <td id="T_cefd5eb6_1b6f_11e8_b083_74e54394bcc7row5_col0" class="data row5 col0" >44</td> 
        <td id="T_cefd5eb6_1b6f_11e8_b083_74e54394bcc7row5_col1" class="data row5 col1" >7.68%</td> 
    </tr>    <tr> 
        <th id="T_cefd5eb6_1b6f_11e8_b083_74e54394bcc7level0_row6" class="row_heading level0 row6" >35 to 39</th> 
        <td id="T_cefd5eb6_1b6f_11e8_b083_74e54394bcc7row6_col0" class="data row6 col0" >25</td> 
        <td id="T_cefd5eb6_1b6f_11e8_b083_74e54394bcc7row6_col1" class="data row6 col1" >4.36%</td> 
    </tr>    <tr> 
        <th id="T_cefd5eb6_1b6f_11e8_b083_74e54394bcc7level0_row7" class="row_heading level0 row7" >40 to 45</th> 
        <td id="T_cefd5eb6_1b6f_11e8_b083_74e54394bcc7row7_col0" class="data row7 col0" >3</td> 
        <td id="T_cefd5eb6_1b6f_11e8_b083_74e54394bcc7row7_col1" class="data row7 col1" >0.52%</td> 
    </tr></tbody> 
</table> 



## Purchasing Analysis (Age)


```python
report_analysis(df_purchase, 'Age Range')
```




<style  type="text/css" >
</style>  
<table id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7" > 
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
        <th id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7level0_row0" class="row_heading level0 row0" >0 to 9</th> 
        <td id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7row0_col0" class="data row0 col0" >32</td> 
        <td id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7row0_col1" class="data row0 col1" >$3.02</td> 
        <td id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7row0_col2" class="data row0 col2" >$96.62</td> 
        <td id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7row0_col3" class="data row0 col3" >$4.39</td> 
    </tr>    <tr> 
        <th id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7level0_row1" class="row_heading level0 row1" >10 to 14</th> 
        <td id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7row1_col0" class="data row1 col0" >78</td> 
        <td id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7row1_col1" class="data row1 col1" >$2.87</td> 
        <td id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7row1_col2" class="data row1 col2" >$224.15</td> 
        <td id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7row1_col3" class="data row1 col3" >$4.15</td> 
    </tr>    <tr> 
        <th id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7level0_row2" class="row_heading level0 row2" >15 to 19</th> 
        <td id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7row2_col0" class="data row2 col0" >184</td> 
        <td id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7row2_col1" class="data row2 col1" >$2.87</td> 
        <td id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7row2_col2" class="data row2 col2" >$528.74</td> 
        <td id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7row2_col3" class="data row2 col3" >$3.80</td> 
    </tr>    <tr> 
        <th id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7level0_row3" class="row_heading level0 row3" >20 to 24</th> 
        <td id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7row3_col0" class="data row3 col0" >305</td> 
        <td id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7row3_col1" class="data row3 col1" >$2.96</td> 
        <td id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7row3_col2" class="data row3 col2" >$902.61</td> 
        <td id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7row3_col3" class="data row3 col3" >$3.86</td> 
    </tr>    <tr> 
        <th id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7level0_row4" class="row_heading level0 row4" >25 to 29</th> 
        <td id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7row4_col0" class="data row4 col0" >76</td> 
        <td id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7row4_col1" class="data row4 col1" >$2.89</td> 
        <td id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7row4_col2" class="data row4 col2" >$219.82</td> 
        <td id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7row4_col3" class="data row4 col3" >$4.23</td> 
    </tr>    <tr> 
        <th id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7level0_row5" class="row_heading level0 row5" >30 to 34</th> 
        <td id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7row5_col0" class="data row5 col0" >58</td> 
        <td id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7row5_col1" class="data row5 col1" >$3.07</td> 
        <td id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7row5_col2" class="data row5 col2" >$178.26</td> 
        <td id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7row5_col3" class="data row5 col3" >$4.05</td> 
    </tr>    <tr> 
        <th id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7level0_row6" class="row_heading level0 row6" >35 to 39</th> 
        <td id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7row6_col0" class="data row6 col0" >44</td> 
        <td id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7row6_col1" class="data row6 col1" >$2.90</td> 
        <td id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7row6_col2" class="data row6 col2" >$127.49</td> 
        <td id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7row6_col3" class="data row6 col3" >$5.10</td> 
    </tr>    <tr> 
        <th id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7level0_row7" class="row_heading level0 row7" >40 to 45</th> 
        <td id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7row7_col0" class="data row7 col0" >3</td> 
        <td id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7row7_col1" class="data row7 col1" >$2.88</td> 
        <td id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7row7_col2" class="data row7 col2" >$8.64</td> 
        <td id="T_cf4d130a_1b6f_11e8_bc77_74e54394bcc7row7_col3" class="data row7 col3" >$2.88</td> 
    </tr></tbody> 
</table> 



## Top Spenders

Use dense rank here since item count is integer and possible be the same.


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
    df_summary = df.groupby(groupby)['Price'].agg(
        ['count', 'mean', 'sum']
    )
    
    df_summary['Rank'] = df_summary[by].rank(method='dense', ascending=False)
    
    return df_summary.loc[df_summary['Rank'] <= top].sort_values(
        by=by, 
        ascending=False
    ).rename(
        columns={
            'count': 'Purchase Count',
            'mean': 'Average Purchase Price',
            'sum': 'Total Purchase Value'
        }
    ).style.format(FORMAT)
```


```python
report_top(df_purchase, 'SN')
```




<style  type="text/css" >
</style>  
<table id="T_210a44cc_1b79_11e8_844f_74e54394bcc7" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >Purchase Count</th> 
        <th class="col_heading level0 col1" >Average Purchase Price</th> 
        <th class="col_heading level0 col2" >Total Purchase Value</th> 
        <th class="col_heading level0 col3" >Rank</th> 
    </tr>    <tr> 
        <th class="index_name level0" >SN</th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_210a44cc_1b79_11e8_844f_74e54394bcc7level0_row0" class="row_heading level0 row0" >Undirrala66</th> 
        <td id="T_210a44cc_1b79_11e8_844f_74e54394bcc7row0_col0" class="data row0 col0" >5</td> 
        <td id="T_210a44cc_1b79_11e8_844f_74e54394bcc7row0_col1" class="data row0 col1" >$3.41</td> 
        <td id="T_210a44cc_1b79_11e8_844f_74e54394bcc7row0_col2" class="data row0 col2" >$17.06</td> 
        <td id="T_210a44cc_1b79_11e8_844f_74e54394bcc7row0_col3" class="data row0 col3" >1</td> 
    </tr>    <tr> 
        <th id="T_210a44cc_1b79_11e8_844f_74e54394bcc7level0_row1" class="row_heading level0 row1" >Saedue76</th> 
        <td id="T_210a44cc_1b79_11e8_844f_74e54394bcc7row1_col0" class="data row1 col0" >4</td> 
        <td id="T_210a44cc_1b79_11e8_844f_74e54394bcc7row1_col1" class="data row1 col1" >$3.39</td> 
        <td id="T_210a44cc_1b79_11e8_844f_74e54394bcc7row1_col2" class="data row1 col2" >$13.56</td> 
        <td id="T_210a44cc_1b79_11e8_844f_74e54394bcc7row1_col3" class="data row1 col3" >2</td> 
    </tr>    <tr> 
        <th id="T_210a44cc_1b79_11e8_844f_74e54394bcc7level0_row2" class="row_heading level0 row2" >Mindimnya67</th> 
        <td id="T_210a44cc_1b79_11e8_844f_74e54394bcc7row2_col0" class="data row2 col0" >4</td> 
        <td id="T_210a44cc_1b79_11e8_844f_74e54394bcc7row2_col1" class="data row2 col1" >$3.18</td> 
        <td id="T_210a44cc_1b79_11e8_844f_74e54394bcc7row2_col2" class="data row2 col2" >$12.74</td> 
        <td id="T_210a44cc_1b79_11e8_844f_74e54394bcc7row2_col3" class="data row2 col3" >3</td> 
    </tr>    <tr> 
        <th id="T_210a44cc_1b79_11e8_844f_74e54394bcc7level0_row3" class="row_heading level0 row3" >Haellysu29</th> 
        <td id="T_210a44cc_1b79_11e8_844f_74e54394bcc7row3_col0" class="data row3 col0" >3</td> 
        <td id="T_210a44cc_1b79_11e8_844f_74e54394bcc7row3_col1" class="data row3 col1" >$4.24</td> 
        <td id="T_210a44cc_1b79_11e8_844f_74e54394bcc7row3_col2" class="data row3 col2" >$12.73</td> 
        <td id="T_210a44cc_1b79_11e8_844f_74e54394bcc7row3_col3" class="data row3 col3" >4</td> 
    </tr>    <tr> 
        <th id="T_210a44cc_1b79_11e8_844f_74e54394bcc7level0_row4" class="row_heading level0 row4" >Eoda93</th> 
        <td id="T_210a44cc_1b79_11e8_844f_74e54394bcc7row4_col0" class="data row4 col0" >3</td> 
        <td id="T_210a44cc_1b79_11e8_844f_74e54394bcc7row4_col1" class="data row4 col1" >$3.86</td> 
        <td id="T_210a44cc_1b79_11e8_844f_74e54394bcc7row4_col2" class="data row4 col2" >$11.58</td> 
        <td id="T_210a44cc_1b79_11e8_844f_74e54394bcc7row4_col3" class="data row4 col3" >5</td> 
    </tr></tbody> 
</table> 



## Most Popular Items


```python
report_top(df_purchase, ['Item ID', 'Item Name'], by='count')
```




<style  type="text/css" >
</style>  
<table id="T_21c67506_1b79_11e8_a729_74e54394bcc7" > 
<thead>    <tr> 
        <th class="blank" ></th> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >Purchase Count</th> 
        <th class="col_heading level0 col1" >Average Purchase Price</th> 
        <th class="col_heading level0 col2" >Total Purchase Value</th> 
        <th class="col_heading level0 col3" >Rank</th> 
    </tr>    <tr> 
        <th class="index_name level0" >Item ID</th> 
        <th class="index_name level1" >Item Name</th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row0" class="row_heading level0 row0" >84</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row0" class="row_heading level1 row0" >Arcane Gem</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row0_col0" class="data row0 col0" >11</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row0_col1" class="data row0 col1" >$2.23</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row0_col2" class="data row0 col2" >$24.53</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row0_col3" class="data row0 col3" >1</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row1" class="row_heading level0 row1" >39</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row1" class="row_heading level1 row1" >Betrayal, Whisper of Grieving Widows</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row1_col0" class="data row1 col0" >11</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row1_col1" class="data row1 col1" >$2.35</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row1_col2" class="data row1 col2" >$25.85</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row1_col3" class="data row1 col3" >1</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row2" class="row_heading level0 row2" >175</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row2" class="row_heading level1 row2" >Woeful Adamantite Claymore</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row2_col0" class="data row2 col0" >9</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row2_col1" class="data row2 col1" >$1.24</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row2_col2" class="data row2 col2" >$11.16</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row2_col3" class="data row2 col3" >2</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row3" class="row_heading level0 row3" >13</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row3" class="row_heading level1 row3" >Serenity</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row3_col0" class="data row3 col0" >9</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row3_col1" class="data row3 col1" >$1.49</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row3_col2" class="data row3 col2" >$13.41</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row3_col3" class="data row3 col3" >2</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row4" class="row_heading level0 row4" >31</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row4" class="row_heading level1 row4" >Trickster</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row4_col0" class="data row4 col0" >9</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row4_col1" class="data row4 col1" >$2.07</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row4_col2" class="data row4 col2" >$18.63</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row4_col3" class="data row4 col3" >2</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row5" class="row_heading level0 row5" >34</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row5" class="row_heading level1 row5" >Retribution Axe</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row5_col0" class="data row5 col0" >9</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row5_col1" class="data row5 col1" >$4.14</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row5_col2" class="data row5 col2" >$37.26</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row5_col3" class="data row5 col3" >2</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row6" class="row_heading level0 row6" >65</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row6" class="row_heading level1 row6" >Conqueror Adamantite Mace</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row6_col0" class="data row6 col0" >8</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row6_col1" class="data row6 col1" >$1.96</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row6_col2" class="data row6 col2" >$15.68</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row6_col3" class="data row6 col3" >3</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row7" class="row_heading level0 row7" >107</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row7" class="row_heading level1 row7" >Splitter, Foe Of Subtlety</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row7_col0" class="data row7 col0" >8</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row7_col1" class="data row7 col1" >$3.61</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row7_col2" class="data row7 col2" >$28.88</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row7_col3" class="data row7 col3" >3</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row8" class="row_heading level0 row8" >106</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row8" class="row_heading level1 row8" >Crying Steel Sickle</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row8_col0" class="data row8 col0" >8</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row8_col1" class="data row8 col1" >$2.29</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row8_col2" class="data row8 col2" >$18.32</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row8_col3" class="data row8 col3" >3</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row9" class="row_heading level0 row9" >92</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row9" class="row_heading level1 row9" >Final Critic</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row9_col0" class="data row9 col0" >8</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row9_col1" class="data row9 col1" >$1.36</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row9_col2" class="data row9 col2" >$10.88</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row9_col3" class="data row9 col3" >3</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row10" class="row_heading level0 row10" >44</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row10" class="row_heading level1 row10" >Bonecarvin Battle Axe</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row10_col0" class="data row10 col0" >8</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row10_col1" class="data row10 col1" >$2.46</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row10_col2" class="data row10 col2" >$19.68</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row10_col3" class="data row10 col3" >3</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row11" class="row_heading level0 row11" >152</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row11" class="row_heading level1 row11" >Darkheart</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row11_col0" class="data row11 col0" >8</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row11_col1" class="data row11 col1" >$3.15</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row11_col2" class="data row11 col2" >$25.20</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row11_col3" class="data row11 col3" >3</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row12" class="row_heading level0 row12" >66</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row12" class="row_heading level1 row12" >Victor Iron Spikes</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row12_col0" class="data row12 col0" >7</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row12_col1" class="data row12 col1" >$3.55</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row12_col2" class="data row12 col2" >$24.85</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row12_col3" class="data row12 col3" >4</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row13" class="row_heading level0 row13" >130</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row13" class="row_heading level1 row13" >Alpha</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row13_col0" class="data row13 col0" >7</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row13_col1" class="data row13 col1" >$1.56</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row13_col2" class="data row13 col2" >$10.92</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row13_col3" class="data row13 col3" >4</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row14" class="row_heading level0 row14" >79</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row14" class="row_heading level1 row14" >Alpha, Oath of Zeal</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row14_col0" class="data row14 col0" >7</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row14_col1" class="data row14 col1" >$2.88</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row14_col2" class="data row14 col2" >$20.16</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row14_col3" class="data row14 col3" >4</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row15" class="row_heading level0 row15" >179</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row15" class="row_heading level1 row15" >Wolf, Promise of the Moonwalker</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row15_col0" class="data row15 col0" >7</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row15_col1" class="data row15 col1" >$1.88</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row15_col2" class="data row15 col2" >$13.16</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row15_col3" class="data row15 col3" >4</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row16" class="row_heading level0 row16" >115</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row16" class="row_heading level1 row16" >Spectral Diamond Doomblade</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row16_col0" class="data row16 col0" >7</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row16_col1" class="data row16 col1" >$4.25</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row16_col2" class="data row16 col2" >$29.75</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row16_col3" class="data row16 col3" >4</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row17" class="row_heading level0 row17" >154</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row17" class="row_heading level1 row17" >Feral Katana</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row17_col0" class="data row17 col0" >7</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row17_col1" class="data row17 col1" >$2.19</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row17_col2" class="data row17 col2" >$15.33</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row17_col3" class="data row17 col3" >4</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row18" class="row_heading level0 row18" >158</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row18" class="row_heading level1 row18" >Darkheart, Butcher of the Champion</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row18_col0" class="data row18 col0" >7</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row18_col1" class="data row18 col1" >$3.56</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row18_col2" class="data row18 col2" >$24.92</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row18_col3" class="data row18 col3" >4</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row19" class="row_heading level0 row19" >18</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row19" class="row_heading level1 row19" >Torchlight, Bond of Storms</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row19_col0" class="data row19 col0" >7</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row19_col1" class="data row19 col1" >$1.77</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row19_col2" class="data row19 col2" >$12.39</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row19_col3" class="data row19 col3" >4</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row20" class="row_heading level0 row20" >172</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row20" class="row_heading level1 row20" >Blade of the Grave</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row20_col0" class="data row20 col0" >7</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row20_col1" class="data row20 col1" >$1.69</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row20_col2" class="data row20 col2" >$11.83</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row20_col3" class="data row20 col3" >4</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row21" class="row_heading level0 row21" >11</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row21" class="row_heading level1 row21" >Brimstone</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row21_col0" class="data row21 col0" >7</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row21_col1" class="data row21 col1" >$2.52</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row21_col2" class="data row21 col2" >$17.64</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row21_col3" class="data row21 col3" >4</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row22" class="row_heading level0 row22" >108</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row22" class="row_heading level1 row22" >Extraction, Quickblade Of Trembling Hands</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row22_col0" class="data row22 col0" >7</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row22_col1" class="data row22 col1" >$3.39</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row22_col2" class="data row22 col2" >$23.73</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row22_col3" class="data row22 col3" >4</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row23" class="row_heading level0 row23" >148</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row23" class="row_heading level1 row23" >Warmonger, Gift of Suffering's End</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row23_col0" class="data row23 col0" >6</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row23_col1" class="data row23 col1" >$3.96</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row23_col2" class="data row23 col2" >$23.76</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row23_col3" class="data row23 col3" >5</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row24" class="row_heading level0 row24" >145</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row24" class="row_heading level1 row24" >Fiery Glass Crusader</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row24_col0" class="data row24 col0" >6</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row24_col1" class="data row24 col1" >$4.45</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row24_col2" class="data row24 col2" >$26.70</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row24_col3" class="data row24 col3" >5</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row25" class="row_heading level0 row25" >128</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row25" class="row_heading level1 row25" >Blazeguard, Reach of Eternity</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row25_col0" class="data row25 col0" >6</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row25_col1" class="data row25 col1" >$4.00</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row25_col2" class="data row25 col2" >$24.00</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row25_col3" class="data row25 col3" >5</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row26" class="row_heading level0 row26" >124</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row26" class="row_heading level1 row26" >Venom Claymore</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row26_col0" class="data row26 col0" >6</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row26_col1" class="data row26 col1" >$2.72</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row26_col2" class="data row26 col2" >$16.32</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row26_col3" class="data row26 col3" >5</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row27" class="row_heading level0 row27" >121</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row27" class="row_heading level1 row27" >Massacre</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row27_col0" class="data row27 col0" >6</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row27_col1" class="data row27 col1" >$3.42</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row27_col2" class="data row27 col2" >$20.52</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row27_col3" class="data row27 col3" >5</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row28" class="row_heading level0 row28" >7</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row28" class="row_heading level1 row28" >Thorn, Satchel of Dark Souls</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row28_col0" class="data row28 col0" >6</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row28_col1" class="data row28 col1" >$4.51</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row28_col2" class="data row28 col2" >$27.06</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row28_col3" class="data row28 col3" >5</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row29" class="row_heading level0 row29" >103</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row29" class="row_heading level1 row29" >Singed Scalpel</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row29_col0" class="data row29 col0" >6</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row29_col1" class="data row29 col1" >$4.87</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row29_col2" class="data row29 col2" >$29.22</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row29_col3" class="data row29 col3" >5</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row30" class="row_heading level0 row30" >102</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row30" class="row_heading level1 row30" >Avenger</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row30_col0" class="data row30 col0" >6</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row30_col1" class="data row30 col1" >$4.16</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row30_col2" class="data row30 col2" >$24.96</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row30_col3" class="data row30 col3" >5</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row31" class="row_heading level0 row31" >101</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row31" class="row_heading level1 row31" >Final Critic</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row31_col0" class="data row31 col0" >6</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row31_col1" class="data row31 col1" >$4.62</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row31_col2" class="data row31 col2" >$27.72</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row31_col3" class="data row31 col3" >5</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row32" class="row_heading level0 row32" >8</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row32" class="row_heading level1 row32" >Purgatory, Gem of Regret</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row32_col0" class="data row32 col0" >6</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row32_col1" class="data row32 col1" >$3.91</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row32_col2" class="data row32 col2" >$23.46</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row32_col3" class="data row32 col3" >5</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row33" class="row_heading level0 row33" >85</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row33" class="row_heading level1 row33" >Malificent Bag</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row33_col0" class="data row33 col0" >6</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row33_col1" class="data row33 col1" >$2.17</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row33_col2" class="data row33 col2" >$13.02</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row33_col3" class="data row33 col3" >5</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row34" class="row_heading level0 row34" >73</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row34" class="row_heading level1 row34" >Ritual Mace</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row34_col0" class="data row34 col0" >6</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row34_col1" class="data row34 col1" >$3.74</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row34_col2" class="data row34 col2" >$22.44</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row34_col3" class="data row34 col3" >5</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row35" class="row_heading level0 row35" >47</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row35" class="row_heading level1 row35" >Alpha, Reach of Ending Hope</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row35_col0" class="data row35 col0" >6</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row35_col1" class="data row35 col1" >$1.55</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row35_col2" class="data row35 col2" >$9.30</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row35_col3" class="data row35 col3" >5</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row36" class="row_heading level0 row36" >32</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row36" class="row_heading level1 row36" >Orenmir</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row36_col0" class="data row36 col0" >6</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row36_col1" class="data row36 col1" >$4.95</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row36_col2" class="data row36 col2" >$29.70</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row36_col3" class="data row36 col3" >5</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row37" class="row_heading level0 row37" >22</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row37" class="row_heading level1 row37" >Amnesia</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row37_col0" class="data row37 col0" >6</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row37_col1" class="data row37 col1" >$3.57</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row37_col2" class="data row37 col2" >$21.42</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row37_col3" class="data row37 col3" >5</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row38" class="row_heading level0 row38" >15</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row38" class="row_heading level1 row38" >Soul Infused Crystal</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row38_col0" class="data row38 col0" >6</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row38_col1" class="data row38 col1" >$1.03</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row38_col2" class="data row38 col2" >$6.18</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row38_col3" class="data row38 col3" >5</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row39" class="row_heading level0 row39" >10</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row39" class="row_heading level1 row39" >Sleepwalker</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row39_col0" class="data row39 col0" >6</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row39_col1" class="data row39 col1" >$1.73</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row39_col2" class="data row39 col2" >$10.38</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row39_col3" class="data row39 col3" >5</td> 
    </tr>    <tr> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level0_row40" class="row_heading level0 row40" >91</th> 
        <th id="T_21c67506_1b79_11e8_a729_74e54394bcc7level1_row40" class="row_heading level1 row40" >Celeste</th> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row40_col0" class="data row40 col0" >6</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row40_col1" class="data row40 col1" >$3.71</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row40_col2" class="data row40 col2" >$22.26</td> 
        <td id="T_21c67506_1b79_11e8_a729_74e54394bcc7row40_col3" class="data row40 col3" >5</td> 
    </tr></tbody> 
</table> 



## Most Porfitable Items


```python
report_top(df_purchase, ['Item ID', 'Item Name'])
```




<style  type="text/css" >
</style>  
<table id="T_26028f40_1b79_11e8_b6d0_74e54394bcc7" > 
<thead>    <tr> 
        <th class="blank" ></th> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >Purchase Count</th> 
        <th class="col_heading level0 col1" >Average Purchase Price</th> 
        <th class="col_heading level0 col2" >Total Purchase Value</th> 
        <th class="col_heading level0 col3" >Rank</th> 
    </tr>    <tr> 
        <th class="index_name level0" >Item ID</th> 
        <th class="index_name level1" >Item Name</th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_26028f40_1b79_11e8_b6d0_74e54394bcc7level0_row0" class="row_heading level0 row0" >34</th> 
        <th id="T_26028f40_1b79_11e8_b6d0_74e54394bcc7level1_row0" class="row_heading level1 row0" >Retribution Axe</th> 
        <td id="T_26028f40_1b79_11e8_b6d0_74e54394bcc7row0_col0" class="data row0 col0" >9</td> 
        <td id="T_26028f40_1b79_11e8_b6d0_74e54394bcc7row0_col1" class="data row0 col1" >$4.14</td> 
        <td id="T_26028f40_1b79_11e8_b6d0_74e54394bcc7row0_col2" class="data row0 col2" >$37.26</td> 
        <td id="T_26028f40_1b79_11e8_b6d0_74e54394bcc7row0_col3" class="data row0 col3" >1</td> 
    </tr>    <tr> 
        <th id="T_26028f40_1b79_11e8_b6d0_74e54394bcc7level0_row1" class="row_heading level0 row1" >115</th> 
        <th id="T_26028f40_1b79_11e8_b6d0_74e54394bcc7level1_row1" class="row_heading level1 row1" >Spectral Diamond Doomblade</th> 
        <td id="T_26028f40_1b79_11e8_b6d0_74e54394bcc7row1_col0" class="data row1 col0" >7</td> 
        <td id="T_26028f40_1b79_11e8_b6d0_74e54394bcc7row1_col1" class="data row1 col1" >$4.25</td> 
        <td id="T_26028f40_1b79_11e8_b6d0_74e54394bcc7row1_col2" class="data row1 col2" >$29.75</td> 
        <td id="T_26028f40_1b79_11e8_b6d0_74e54394bcc7row1_col3" class="data row1 col3" >2</td> 
    </tr>    <tr> 
        <th id="T_26028f40_1b79_11e8_b6d0_74e54394bcc7level0_row2" class="row_heading level0 row2" >32</th> 
        <th id="T_26028f40_1b79_11e8_b6d0_74e54394bcc7level1_row2" class="row_heading level1 row2" >Orenmir</th> 
        <td id="T_26028f40_1b79_11e8_b6d0_74e54394bcc7row2_col0" class="data row2 col0" >6</td> 
        <td id="T_26028f40_1b79_11e8_b6d0_74e54394bcc7row2_col1" class="data row2 col1" >$4.95</td> 
        <td id="T_26028f40_1b79_11e8_b6d0_74e54394bcc7row2_col2" class="data row2 col2" >$29.70</td> 
        <td id="T_26028f40_1b79_11e8_b6d0_74e54394bcc7row2_col3" class="data row2 col3" >3</td> 
    </tr>    <tr> 
        <th id="T_26028f40_1b79_11e8_b6d0_74e54394bcc7level0_row3" class="row_heading level0 row3" >103</th> 
        <th id="T_26028f40_1b79_11e8_b6d0_74e54394bcc7level1_row3" class="row_heading level1 row3" >Singed Scalpel</th> 
        <td id="T_26028f40_1b79_11e8_b6d0_74e54394bcc7row3_col0" class="data row3 col0" >6</td> 
        <td id="T_26028f40_1b79_11e8_b6d0_74e54394bcc7row3_col1" class="data row3 col1" >$4.87</td> 
        <td id="T_26028f40_1b79_11e8_b6d0_74e54394bcc7row3_col2" class="data row3 col2" >$29.22</td> 
        <td id="T_26028f40_1b79_11e8_b6d0_74e54394bcc7row3_col3" class="data row3 col3" >4</td> 
    </tr>    <tr> 
        <th id="T_26028f40_1b79_11e8_b6d0_74e54394bcc7level0_row4" class="row_heading level0 row4" >107</th> 
        <th id="T_26028f40_1b79_11e8_b6d0_74e54394bcc7level1_row4" class="row_heading level1 row4" >Splitter, Foe Of Subtlety</th> 
        <td id="T_26028f40_1b79_11e8_b6d0_74e54394bcc7row4_col0" class="data row4 col0" >8</td> 
        <td id="T_26028f40_1b79_11e8_b6d0_74e54394bcc7row4_col1" class="data row4 col1" >$3.61</td> 
        <td id="T_26028f40_1b79_11e8_b6d0_74e54394bcc7row4_col2" class="data row4 col2" >$28.88</td> 
        <td id="T_26028f40_1b79_11e8_b6d0_74e54394bcc7row4_col3" class="data row4 col3" >5</td> 
    </tr></tbody> 
</table> 


