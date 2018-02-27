
## Trend Observed
1. Charter schools has much higher passing rate than district schools.
2. Schools with higer budget per student have lower average math scores.
3. Smllaer schools have higher average math scores.


```python
import os
from collections import OrderedDict
import numpy as np
import pandas as pd
from pandas import Series, DataFrame

PASSING_SCORE = 60 # Assumption on passing score
passing = lambda x: sum( x >= PASSING_SCORE ) / x.count()
passing.__name__ = 'passing' # To avoid <lambda> in column name

COUNT_FORMAT =  '{:,}'
SCORE_FORMAT = '{:.2f}'
PERC_FORMAT = '{:.2%}'
BUDGET_FORAMT = '${:,.2f}'

OUTPUT_FORMAT = {
    'Total Schools': COUNT_FORMAT,
    'Total Students': COUNT_FORMAT, 
    'Total Budget': BUDGET_FORAMT,
    'Total School Budget': BUDGET_FORAMT,
    'Per Student Budget': BUDGET_FORAMT,
    'Average Math Score': SCORE_FORMAT,
    'Average Reading Score': SCORE_FORMAT,
    '% Passing Math': PERC_FORMAT,
    '% Passing Reading': PERC_FORMAT,
    'Overall Passing Rate (Average of the above two)': PERC_FORMAT,
}
```


```python
BASE_DIR = 'raw_data'
school_file = os.path.join(BASE_DIR, 'schools_complete.csv')
df_school = pd.read_csv(school_file)

student_file = os.path.join(BASE_DIR, 'students_complete.csv')
df_student = pd.read_csv(student_file)

df_school.rename(
    columns={
        'name': 'school'
    },
    inplace=True
)
```

## District Summary


```python
num_student = df_student.name.count()
passing_rate_math = (df_student.math_score >= PASSING_SCORE).sum() / num_student
passing_rate_reading = (df_student.reading_score > PASSING_SCORE).sum() / num_student
avg_math_score =  df_student.math_score.mean()
avg_reading_score = df_student.reading_score.mean()

district_summary = DataFrame(
    columns = [
        'Total Schools',
        'Total Students',
        'Total Budget',
        'Average Math Score',
        'Average Reading Score',
        '% Passing Math',
        '% Passing Reading',
        'Overall Passing Rate (Average of the above two)',
    ],
    data = [[
        df_school.school.count(),
        num_student,
        df_school.budget.sum(),
        df_student.math_score.mean(),
        df_student.reading_score.mean(),
        passing_rate_math,
        passing_rate_reading,
        (passing_rate_math + passing_rate_reading)/2,
        ]   
    ]
)

district_summary.style.format(OUTPUT_FORMAT)
```




<style  type="text/css" >
</style>  
<table id="T_396276cc_1b82_11e8_bfc4_74e54394bcc7" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >Total Schools</th> 
        <th class="col_heading level0 col1" >Total Students</th> 
        <th class="col_heading level0 col2" >Total Budget</th> 
        <th class="col_heading level0 col3" >Average Math Score</th> 
        <th class="col_heading level0 col4" >Average Reading Score</th> 
        <th class="col_heading level0 col5" >% Passing Math</th> 
        <th class="col_heading level0 col6" >% Passing Reading</th> 
        <th class="col_heading level0 col7" >Overall Passing Rate (Average of the above two)</th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_396276cc_1b82_11e8_bfc4_74e54394bcc7level0_row0" class="row_heading level0 row0" >0</th> 
        <td id="T_396276cc_1b82_11e8_bfc4_74e54394bcc7row0_col0" class="data row0 col0" >15</td> 
        <td id="T_396276cc_1b82_11e8_bfc4_74e54394bcc7row0_col1" class="data row0 col1" >39,170</td> 
        <td id="T_396276cc_1b82_11e8_bfc4_74e54394bcc7row0_col2" class="data row0 col2" >$24,649,428.00</td> 
        <td id="T_396276cc_1b82_11e8_bfc4_74e54394bcc7row0_col3" class="data row0 col3" >78.99</td> 
        <td id="T_396276cc_1b82_11e8_bfc4_74e54394bcc7row0_col4" class="data row0 col4" >81.88</td> 
        <td id="T_396276cc_1b82_11e8_bfc4_74e54394bcc7row0_col5" class="data row0 col5" >92.45%</td> 
        <td id="T_396276cc_1b82_11e8_bfc4_74e54394bcc7row0_col6" class="data row0 col6" >100.00%</td> 
        <td id="T_396276cc_1b82_11e8_bfc4_74e54394bcc7row0_col7" class="data row0 col7" >96.22%</td> 
    </tr></tbody> 
</table> 



## School Summary


```python
# Get budget per school
budget_by_school = df_school.groupby(['school', 'type'])['budget'].sum()

# Get summary on student side.
school_summary = df_student.groupby(['school']).agg(
    {
        'name':'count',
        'math_score': ['mean', passing],
        'reading_score': ['mean', passing]
    }
)
# Flatten two level multi-index
school_summary.columns = ['_'.join(col) for col in school_summary.columns]

# Join by index ID ensures values align
school_summary = school_summary.join(budget_by_school).assign(
    per_student_budget=lambda x: x['budget']/x['name_count'],
    average_passing=lambda x: (x['math_score_passing'] + x['reading_score_passing'])/2
).reset_index().rename(
    columns=OrderedDict(
        [
            ('school', 'School Name'),
            ('type','School Type'),
            ('budget', 'Total School Budget'),
            ('name_count', 'Total Students'),
            ('per_student_budget', 'Per Student Budget'),
            ('math_score_mean', 'Average Math Score'),
            ('reading_score_mean', 'Average Reading Score'),
            ('math_score_passing', '% Passing Math'),
            ('reading_score_passing', '% Passing Reading'),
            ('average_passing',  'Overall Passing Rate (Average of the above two)')
        ]
    )
).reindex(
    columns = [
        'School Name',
        'School Type',
        'Total Students',
        'Total School Budget',
        'Per Student Budget',
        'Average Math Score',
        'Average Reading Score',
        '% Passing Math',
        '% Passing Reading',
        'Overall Passing Rate (Average of the above two)'
    ]
)
school_summary.style.format(OUTPUT_FORMAT)
```




<style  type="text/css" >
</style>  
<table id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >School Name</th> 
        <th class="col_heading level0 col1" >School Type</th> 
        <th class="col_heading level0 col2" >Total Students</th> 
        <th class="col_heading level0 col3" >Total School Budget</th> 
        <th class="col_heading level0 col4" >Per Student Budget</th> 
        <th class="col_heading level0 col5" >Average Math Score</th> 
        <th class="col_heading level0 col6" >Average Reading Score</th> 
        <th class="col_heading level0 col7" >% Passing Math</th> 
        <th class="col_heading level0 col8" >% Passing Reading</th> 
        <th class="col_heading level0 col9" >Overall Passing Rate (Average of the above two)</th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7level0_row0" class="row_heading level0 row0" >0</th> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row0_col0" class="data row0 col0" >Bailey High School</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row0_col1" class="data row0 col1" >District</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row0_col2" class="data row0 col2" >4,976</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row0_col3" class="data row0 col3" >$3,124,928.00</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row0_col4" class="data row0 col4" >$628.00</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row0_col5" class="data row0 col5" >77.05</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row0_col6" class="data row0 col6" >81.03</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row0_col7" class="data row0 col7" >89.53%</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row0_col8" class="data row0 col8" >100.00%</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row0_col9" class="data row0 col9" >94.76%</td> 
    </tr>    <tr> 
        <th id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7level0_row1" class="row_heading level0 row1" >1</th> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row1_col0" class="data row1 col0" >Cabrera High School</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row1_col1" class="data row1 col1" >Charter</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row1_col2" class="data row1 col2" >1,858</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row1_col3" class="data row1 col3" >$1,081,356.00</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row1_col4" class="data row1 col4" >$582.00</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row1_col5" class="data row1 col5" >83.06</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row1_col6" class="data row1 col6" >83.98</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row1_col7" class="data row1 col7" >100.00%</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row1_col8" class="data row1 col8" >100.00%</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row1_col9" class="data row1 col9" >100.00%</td> 
    </tr>    <tr> 
        <th id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7level0_row2" class="row_heading level0 row2" >2</th> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row2_col0" class="data row2 col0" >Figueroa High School</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row2_col1" class="data row2 col1" >District</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row2_col2" class="data row2 col2" >2,949</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row2_col3" class="data row2 col3" >$1,884,411.00</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row2_col4" class="data row2 col4" >$639.00</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row2_col5" class="data row2 col5" >76.71</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row2_col6" class="data row2 col6" >81.16</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row2_col7" class="data row2 col7" >88.44%</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row2_col8" class="data row2 col8" >100.00%</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row2_col9" class="data row2 col9" >94.22%</td> 
    </tr>    <tr> 
        <th id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7level0_row3" class="row_heading level0 row3" >3</th> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row3_col0" class="data row3 col0" >Ford High School</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row3_col1" class="data row3 col1" >District</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row3_col2" class="data row3 col2" >2,739</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row3_col3" class="data row3 col3" >$1,763,916.00</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row3_col4" class="data row3 col4" >$644.00</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row3_col5" class="data row3 col5" >77.10</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row3_col6" class="data row3 col6" >80.75</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row3_col7" class="data row3 col7" >89.30%</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row3_col8" class="data row3 col8" >100.00%</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row3_col9" class="data row3 col9" >94.65%</td> 
    </tr>    <tr> 
        <th id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7level0_row4" class="row_heading level0 row4" >4</th> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row4_col0" class="data row4 col0" >Griffin High School</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row4_col1" class="data row4 col1" >Charter</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row4_col2" class="data row4 col2" >1,468</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row4_col3" class="data row4 col3" >$917,500.00</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row4_col4" class="data row4 col4" >$625.00</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row4_col5" class="data row4 col5" >83.35</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row4_col6" class="data row4 col6" >83.82</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row4_col7" class="data row4 col7" >100.00%</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row4_col8" class="data row4 col8" >100.00%</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row4_col9" class="data row4 col9" >100.00%</td> 
    </tr>    <tr> 
        <th id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7level0_row5" class="row_heading level0 row5" >5</th> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row5_col0" class="data row5 col0" >Hernandez High School</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row5_col1" class="data row5 col1" >District</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row5_col2" class="data row5 col2" >4,635</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row5_col3" class="data row5 col3" >$3,022,020.00</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row5_col4" class="data row5 col4" >$652.00</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row5_col5" class="data row5 col5" >77.29</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row5_col6" class="data row5 col6" >80.93</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row5_col7" class="data row5 col7" >89.08%</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row5_col8" class="data row5 col8" >100.00%</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row5_col9" class="data row5 col9" >94.54%</td> 
    </tr>    <tr> 
        <th id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7level0_row6" class="row_heading level0 row6" >6</th> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row6_col0" class="data row6 col0" >Holden High School</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row6_col1" class="data row6 col1" >Charter</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row6_col2" class="data row6 col2" >427</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row6_col3" class="data row6 col3" >$248,087.00</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row6_col4" class="data row6 col4" >$581.00</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row6_col5" class="data row6 col5" >83.80</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row6_col6" class="data row6 col6" >83.81</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row6_col7" class="data row6 col7" >100.00%</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row6_col8" class="data row6 col8" >100.00%</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row6_col9" class="data row6 col9" >100.00%</td> 
    </tr>    <tr> 
        <th id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7level0_row7" class="row_heading level0 row7" >7</th> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row7_col0" class="data row7 col0" >Huang High School</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row7_col1" class="data row7 col1" >District</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row7_col2" class="data row7 col2" >2,917</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row7_col3" class="data row7 col3" >$1,910,635.00</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row7_col4" class="data row7 col4" >$655.00</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row7_col5" class="data row7 col5" >76.63</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row7_col6" class="data row7 col6" >81.18</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row7_col7" class="data row7 col7" >88.86%</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row7_col8" class="data row7 col8" >100.00%</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row7_col9" class="data row7 col9" >94.43%</td> 
    </tr>    <tr> 
        <th id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7level0_row8" class="row_heading level0 row8" >8</th> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row8_col0" class="data row8 col0" >Johnson High School</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row8_col1" class="data row8 col1" >District</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row8_col2" class="data row8 col2" >4,761</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row8_col3" class="data row8 col3" >$3,094,650.00</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row8_col4" class="data row8 col4" >$650.00</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row8_col5" class="data row8 col5" >77.07</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row8_col6" class="data row8 col6" >80.97</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row8_col7" class="data row8 col7" >89.18%</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row8_col8" class="data row8 col8" >100.00%</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row8_col9" class="data row8 col9" >94.59%</td> 
    </tr>    <tr> 
        <th id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7level0_row9" class="row_heading level0 row9" >9</th> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row9_col0" class="data row9 col0" >Pena High School</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row9_col1" class="data row9 col1" >Charter</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row9_col2" class="data row9 col2" >962</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row9_col3" class="data row9 col3" >$585,858.00</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row9_col4" class="data row9 col4" >$609.00</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row9_col5" class="data row9 col5" >83.84</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row9_col6" class="data row9 col6" >84.04</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row9_col7" class="data row9 col7" >100.00%</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row9_col8" class="data row9 col8" >100.00%</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row9_col9" class="data row9 col9" >100.00%</td> 
    </tr>    <tr> 
        <th id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7level0_row10" class="row_heading level0 row10" >10</th> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row10_col0" class="data row10 col0" >Rodriguez High School</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row10_col1" class="data row10 col1" >District</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row10_col2" class="data row10 col2" >3,999</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row10_col3" class="data row10 col3" >$2,547,363.00</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row10_col4" class="data row10 col4" >$637.00</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row10_col5" class="data row10 col5" >76.84</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row10_col6" class="data row10 col6" >80.74</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row10_col7" class="data row10 col7" >88.55%</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row10_col8" class="data row10 col8" >100.00%</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row10_col9" class="data row10 col9" >94.27%</td> 
    </tr>    <tr> 
        <th id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7level0_row11" class="row_heading level0 row11" >11</th> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row11_col0" class="data row11 col0" >Shelton High School</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row11_col1" class="data row11 col1" >Charter</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row11_col2" class="data row11 col2" >1,761</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row11_col3" class="data row11 col3" >$1,056,600.00</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row11_col4" class="data row11 col4" >$600.00</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row11_col5" class="data row11 col5" >83.36</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row11_col6" class="data row11 col6" >83.73</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row11_col7" class="data row11 col7" >100.00%</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row11_col8" class="data row11 col8" >100.00%</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row11_col9" class="data row11 col9" >100.00%</td> 
    </tr>    <tr> 
        <th id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7level0_row12" class="row_heading level0 row12" >12</th> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row12_col0" class="data row12 col0" >Thomas High School</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row12_col1" class="data row12 col1" >Charter</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row12_col2" class="data row12 col2" >1,635</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row12_col3" class="data row12 col3" >$1,043,130.00</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row12_col4" class="data row12 col4" >$638.00</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row12_col5" class="data row12 col5" >83.42</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row12_col6" class="data row12 col6" >83.85</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row12_col7" class="data row12 col7" >100.00%</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row12_col8" class="data row12 col8" >100.00%</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row12_col9" class="data row12 col9" >100.00%</td> 
    </tr>    <tr> 
        <th id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7level0_row13" class="row_heading level0 row13" >13</th> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row13_col0" class="data row13 col0" >Wilson High School</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row13_col1" class="data row13 col1" >Charter</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row13_col2" class="data row13 col2" >2,283</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row13_col3" class="data row13 col3" >$1,319,574.00</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row13_col4" class="data row13 col4" >$578.00</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row13_col5" class="data row13 col5" >83.27</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row13_col6" class="data row13 col6" >83.99</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row13_col7" class="data row13 col7" >100.00%</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row13_col8" class="data row13 col8" >100.00%</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row13_col9" class="data row13 col9" >100.00%</td> 
    </tr>    <tr> 
        <th id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7level0_row14" class="row_heading level0 row14" >14</th> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row14_col0" class="data row14 col0" >Wright High School</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row14_col1" class="data row14 col1" >Charter</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row14_col2" class="data row14 col2" >1,800</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row14_col3" class="data row14 col3" >$1,049,400.00</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row14_col4" class="data row14 col4" >$583.00</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row14_col5" class="data row14 col5" >83.68</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row14_col6" class="data row14 col6" >83.95</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row14_col7" class="data row14 col7" >100.00%</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row14_col8" class="data row14 col8" >100.00%</td> 
        <td id="T_39f9ce3a_1b82_11e8_b5bd_74e54394bcc7row14_col9" class="data row14 col9" >100.00%</td> 
    </tr></tbody> 
</table> 



## Top Perfroming School (By Passing Rate)

Following are list of top 5 sorted by passing rate. However since there are 8 schools with perfect pasing rate, and there are only 15 schools, this list might not mean much.


```python
school_summary.sort_values('Overall Passing Rate (Average of the above two)',
                    ascending=False).head(5).style.format(OUTPUT_FORMAT)
```




<style  type="text/css" >
</style>  
<table id="T_aff7f188_1b80_11e8_855a_74e54394bcc7" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >School Name</th> 
        <th class="col_heading level0 col1" >School Type</th> 
        <th class="col_heading level0 col2" >Total Students</th> 
        <th class="col_heading level0 col3" >Total School Budget</th> 
        <th class="col_heading level0 col4" >Per Student Budget</th> 
        <th class="col_heading level0 col5" >Average Math Score</th> 
        <th class="col_heading level0 col6" >Average Reading Score</th> 
        <th class="col_heading level0 col7" >% Passing Math</th> 
        <th class="col_heading level0 col8" >% Passing Reading</th> 
        <th class="col_heading level0 col9" >Overall Passing Rate (Average of the above two)</th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_aff7f188_1b80_11e8_855a_74e54394bcc7level0_row0" class="row_heading level0 row0" >1</th> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row0_col0" class="data row0 col0" >Cabrera High School</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row0_col1" class="data row0 col1" >Charter</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row0_col2" class="data row0 col2" >1,858</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row0_col3" class="data row0 col3" >$1,081,356.00</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row0_col4" class="data row0 col4" >$582.00</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row0_col5" class="data row0 col5" >83.06</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row0_col6" class="data row0 col6" >83.98</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row0_col7" class="data row0 col7" >100.00%</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row0_col8" class="data row0 col8" >100.00%</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row0_col9" class="data row0 col9" >100.00%</td> 
    </tr>    <tr> 
        <th id="T_aff7f188_1b80_11e8_855a_74e54394bcc7level0_row1" class="row_heading level0 row1" >4</th> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row1_col0" class="data row1 col0" >Griffin High School</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row1_col1" class="data row1 col1" >Charter</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row1_col2" class="data row1 col2" >1,468</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row1_col3" class="data row1 col3" >$917,500.00</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row1_col4" class="data row1 col4" >$625.00</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row1_col5" class="data row1 col5" >83.35</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row1_col6" class="data row1 col6" >83.82</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row1_col7" class="data row1 col7" >100.00%</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row1_col8" class="data row1 col8" >100.00%</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row1_col9" class="data row1 col9" >100.00%</td> 
    </tr>    <tr> 
        <th id="T_aff7f188_1b80_11e8_855a_74e54394bcc7level0_row2" class="row_heading level0 row2" >6</th> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row2_col0" class="data row2 col0" >Holden High School</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row2_col1" class="data row2 col1" >Charter</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row2_col2" class="data row2 col2" >427</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row2_col3" class="data row2 col3" >$248,087.00</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row2_col4" class="data row2 col4" >$581.00</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row2_col5" class="data row2 col5" >83.80</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row2_col6" class="data row2 col6" >83.81</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row2_col7" class="data row2 col7" >100.00%</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row2_col8" class="data row2 col8" >100.00%</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row2_col9" class="data row2 col9" >100.00%</td> 
    </tr>    <tr> 
        <th id="T_aff7f188_1b80_11e8_855a_74e54394bcc7level0_row3" class="row_heading level0 row3" >9</th> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row3_col0" class="data row3 col0" >Pena High School</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row3_col1" class="data row3 col1" >Charter</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row3_col2" class="data row3 col2" >962</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row3_col3" class="data row3 col3" >$585,858.00</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row3_col4" class="data row3 col4" >$609.00</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row3_col5" class="data row3 col5" >83.84</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row3_col6" class="data row3 col6" >84.04</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row3_col7" class="data row3 col7" >100.00%</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row3_col8" class="data row3 col8" >100.00%</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row3_col9" class="data row3 col9" >100.00%</td> 
    </tr>    <tr> 
        <th id="T_aff7f188_1b80_11e8_855a_74e54394bcc7level0_row4" class="row_heading level0 row4" >11</th> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row4_col0" class="data row4 col0" >Shelton High School</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row4_col1" class="data row4 col1" >Charter</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row4_col2" class="data row4 col2" >1,761</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row4_col3" class="data row4 col3" >$1,056,600.00</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row4_col4" class="data row4 col4" >$600.00</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row4_col5" class="data row4 col5" >83.36</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row4_col6" class="data row4 col6" >83.73</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row4_col7" class="data row4 col7" >100.00%</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row4_col8" class="data row4 col8" >100.00%</td> 
        <td id="T_aff7f188_1b80_11e8_855a_74e54394bcc7row4_col9" class="data row4 col9" >100.00%</td> 
    </tr></tbody> 
</table> 



## Bottom Performing Schools (By Passing Rate)


```python
school_summary.sort_values('Overall Passing Rate (Average of the above two)').head(5).style.format(OUTPUT_FORMAT)
```




<style  type="text/css" >
</style>  
<table id="T_b136723e_1b80_11e8_bf54_74e54394bcc7" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >School Name</th> 
        <th class="col_heading level0 col1" >School Type</th> 
        <th class="col_heading level0 col2" >Total Students</th> 
        <th class="col_heading level0 col3" >Total School Budget</th> 
        <th class="col_heading level0 col4" >Per Student Budget</th> 
        <th class="col_heading level0 col5" >Average Math Score</th> 
        <th class="col_heading level0 col6" >Average Reading Score</th> 
        <th class="col_heading level0 col7" >% Passing Math</th> 
        <th class="col_heading level0 col8" >% Passing Reading</th> 
        <th class="col_heading level0 col9" >Overall Passing Rate (Average of the above two)</th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_b136723e_1b80_11e8_bf54_74e54394bcc7level0_row0" class="row_heading level0 row0" >2</th> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row0_col0" class="data row0 col0" >Figueroa High School</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row0_col1" class="data row0 col1" >District</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row0_col2" class="data row0 col2" >2,949</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row0_col3" class="data row0 col3" >$1,884,411.00</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row0_col4" class="data row0 col4" >$639.00</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row0_col5" class="data row0 col5" >76.71</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row0_col6" class="data row0 col6" >81.16</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row0_col7" class="data row0 col7" >88.44%</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row0_col8" class="data row0 col8" >100.00%</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row0_col9" class="data row0 col9" >94.22%</td> 
    </tr>    <tr> 
        <th id="T_b136723e_1b80_11e8_bf54_74e54394bcc7level0_row1" class="row_heading level0 row1" >10</th> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row1_col0" class="data row1 col0" >Rodriguez High School</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row1_col1" class="data row1 col1" >District</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row1_col2" class="data row1 col2" >3,999</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row1_col3" class="data row1 col3" >$2,547,363.00</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row1_col4" class="data row1 col4" >$637.00</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row1_col5" class="data row1 col5" >76.84</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row1_col6" class="data row1 col6" >80.74</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row1_col7" class="data row1 col7" >88.55%</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row1_col8" class="data row1 col8" >100.00%</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row1_col9" class="data row1 col9" >94.27%</td> 
    </tr>    <tr> 
        <th id="T_b136723e_1b80_11e8_bf54_74e54394bcc7level0_row2" class="row_heading level0 row2" >7</th> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row2_col0" class="data row2 col0" >Huang High School</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row2_col1" class="data row2 col1" >District</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row2_col2" class="data row2 col2" >2,917</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row2_col3" class="data row2 col3" >$1,910,635.00</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row2_col4" class="data row2 col4" >$655.00</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row2_col5" class="data row2 col5" >76.63</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row2_col6" class="data row2 col6" >81.18</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row2_col7" class="data row2 col7" >88.86%</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row2_col8" class="data row2 col8" >100.00%</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row2_col9" class="data row2 col9" >94.43%</td> 
    </tr>    <tr> 
        <th id="T_b136723e_1b80_11e8_bf54_74e54394bcc7level0_row3" class="row_heading level0 row3" >5</th> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row3_col0" class="data row3 col0" >Hernandez High School</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row3_col1" class="data row3 col1" >District</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row3_col2" class="data row3 col2" >4,635</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row3_col3" class="data row3 col3" >$3,022,020.00</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row3_col4" class="data row3 col4" >$652.00</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row3_col5" class="data row3 col5" >77.29</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row3_col6" class="data row3 col6" >80.93</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row3_col7" class="data row3 col7" >89.08%</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row3_col8" class="data row3 col8" >100.00%</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row3_col9" class="data row3 col9" >94.54%</td> 
    </tr>    <tr> 
        <th id="T_b136723e_1b80_11e8_bf54_74e54394bcc7level0_row4" class="row_heading level0 row4" >8</th> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row4_col0" class="data row4 col0" >Johnson High School</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row4_col1" class="data row4 col1" >District</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row4_col2" class="data row4 col2" >4,761</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row4_col3" class="data row4 col3" >$3,094,650.00</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row4_col4" class="data row4 col4" >$650.00</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row4_col5" class="data row4 col5" >77.07</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row4_col6" class="data row4 col6" >80.97</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row4_col7" class="data row4 col7" >89.18%</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row4_col8" class="data row4 col8" >100.00%</td> 
        <td id="T_b136723e_1b80_11e8_bf54_74e54394bcc7row4_col9" class="data row4 col9" >94.59%</td> 
    </tr></tbody> 
</table> 



## Math/Reading Scores by Grade

It is redundent to list math and reading scores in two tables so I list them in one table below.


```python
df_student['grade'] = pd.Categorical(
    df_student['grade'], 
    sorted(
        df_student['grade'].unique(),
        key=lambda x: int(x.replace('th', ''))
    )
)

df_by_grade = DataFrame(
    df_student.groupby(['school', 'grade'])[['math_score', 'reading_score']].mean()
).rename(
    columns = {
        'school': 'School Name',
        'grade': 'Grade',
        'math_score': 'Average Math Score',
        'reading_score': 'Average Reading Score'
    }
)

df_by_grade.style.format(OUTPUT_FORMAT)
```




<style  type="text/css" >
</style>  
<table id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7" > 
<thead>    <tr> 
        <th class="blank" ></th> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >Average Math Score</th> 
        <th class="col_heading level0 col1" >Average Reading Score</th> 
    </tr>    <tr> 
        <th class="index_name level0" >school</th> 
        <th class="index_name level1" >grade</th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level0_row0" class="row_heading level0 row0" rowspan=4>Bailey High School</th> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row0" class="row_heading level1 row0" >9th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row0_col0" class="data row0 col0" >77.08</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row0_col1" class="data row0 col1" >81.30</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row1" class="row_heading level1 row1" >10th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row1_col0" class="data row1 col0" >77.00</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row1_col1" class="data row1 col1" >80.91</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row2" class="row_heading level1 row2" >11th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row2_col0" class="data row2 col0" >77.52</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row2_col1" class="data row2 col1" >80.95</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row3" class="row_heading level1 row3" >12th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row3_col0" class="data row3 col0" >76.49</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row3_col1" class="data row3 col1" >80.91</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level0_row4" class="row_heading level0 row4" rowspan=4>Cabrera High School</th> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row4" class="row_heading level1 row4" >9th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row4_col0" class="data row4 col0" >83.09</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row4_col1" class="data row4 col1" >83.68</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row5" class="row_heading level1 row5" >10th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row5_col0" class="data row5 col0" >83.15</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row5_col1" class="data row5 col1" >84.25</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row6" class="row_heading level1 row6" >11th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row6_col0" class="data row6 col0" >82.77</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row6_col1" class="data row6 col1" >83.79</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row7" class="row_heading level1 row7" >12th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row7_col0" class="data row7 col0" >83.28</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row7_col1" class="data row7 col1" >84.29</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level0_row8" class="row_heading level0 row8" rowspan=4>Figueroa High School</th> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row8" class="row_heading level1 row8" >9th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row8_col0" class="data row8 col0" >76.40</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row8_col1" class="data row8 col1" >81.20</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row9" class="row_heading level1 row9" >10th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row9_col0" class="data row9 col0" >76.54</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row9_col1" class="data row9 col1" >81.41</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row10" class="row_heading level1 row10" >11th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row10_col0" class="data row10 col0" >76.88</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row10_col1" class="data row10 col1" >80.64</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row11" class="row_heading level1 row11" >12th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row11_col0" class="data row11 col0" >77.15</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row11_col1" class="data row11 col1" >81.38</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level0_row12" class="row_heading level0 row12" rowspan=4>Ford High School</th> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row12" class="row_heading level1 row12" >9th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row12_col0" class="data row12 col0" >77.36</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row12_col1" class="data row12 col1" >80.63</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row13" class="row_heading level1 row13" >10th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row13_col0" class="data row13 col0" >77.67</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row13_col1" class="data row13 col1" >81.26</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row14" class="row_heading level1 row14" >11th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row14_col0" class="data row14 col0" >76.92</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row14_col1" class="data row14 col1" >80.40</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row15" class="row_heading level1 row15" >12th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row15_col0" class="data row15 col0" >76.18</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row15_col1" class="data row15 col1" >80.66</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level0_row16" class="row_heading level0 row16" rowspan=4>Griffin High School</th> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row16" class="row_heading level1 row16" >9th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row16_col0" class="data row16 col0" >82.04</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row16_col1" class="data row16 col1" >83.37</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row17" class="row_heading level1 row17" >10th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row17_col0" class="data row17 col0" >84.23</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row17_col1" class="data row17 col1" >83.71</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row18" class="row_heading level1 row18" >11th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row18_col0" class="data row18 col0" >83.84</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row18_col1" class="data row18 col1" >84.29</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row19" class="row_heading level1 row19" >12th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row19_col0" class="data row19 col0" >83.36</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row19_col1" class="data row19 col1" >84.01</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level0_row20" class="row_heading level0 row20" rowspan=4>Hernandez High School</th> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row20" class="row_heading level1 row20" >9th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row20_col0" class="data row20 col0" >77.44</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row20_col1" class="data row20 col1" >80.87</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row21" class="row_heading level1 row21" >10th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row21_col0" class="data row21 col0" >77.34</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row21_col1" class="data row21 col1" >80.66</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row22" class="row_heading level1 row22" >11th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row22_col0" class="data row22 col0" >77.14</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row22_col1" class="data row22 col1" >81.40</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row23" class="row_heading level1 row23" >12th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row23_col0" class="data row23 col0" >77.19</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row23_col1" class="data row23 col1" >80.86</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level0_row24" class="row_heading level0 row24" rowspan=4>Holden High School</th> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row24" class="row_heading level1 row24" >9th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row24_col0" class="data row24 col0" >83.79</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row24_col1" class="data row24 col1" >83.68</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row25" class="row_heading level1 row25" >10th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row25_col0" class="data row25 col0" >83.43</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row25_col1" class="data row25 col1" >83.32</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row26" class="row_heading level1 row26" >11th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row26_col0" class="data row26 col0" >85.00</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row26_col1" class="data row26 col1" >83.82</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row27" class="row_heading level1 row27" >12th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row27_col0" class="data row27 col0" >82.86</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row27_col1" class="data row27 col1" >84.70</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level0_row28" class="row_heading level0 row28" rowspan=4>Huang High School</th> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row28" class="row_heading level1 row28" >9th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row28_col0" class="data row28 col0" >77.03</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row28_col1" class="data row28 col1" >81.29</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row29" class="row_heading level1 row29" >10th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row29_col0" class="data row29 col0" >75.91</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row29_col1" class="data row29 col1" >81.51</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row30" class="row_heading level1 row30" >11th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row30_col0" class="data row30 col0" >76.45</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row30_col1" class="data row30 col1" >81.42</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row31" class="row_heading level1 row31" >12th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row31_col0" class="data row31 col0" >77.23</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row31_col1" class="data row31 col1" >80.31</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level0_row32" class="row_heading level0 row32" rowspan=4>Johnson High School</th> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row32" class="row_heading level1 row32" >9th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row32_col0" class="data row32 col0" >77.19</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row32_col1" class="data row32 col1" >81.26</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row33" class="row_heading level1 row33" >10th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row33_col0" class="data row33 col0" >76.69</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row33_col1" class="data row33 col1" >80.77</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row34" class="row_heading level1 row34" >11th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row34_col0" class="data row34 col0" >77.49</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row34_col1" class="data row34 col1" >80.62</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row35" class="row_heading level1 row35" >12th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row35_col0" class="data row35 col0" >76.86</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row35_col1" class="data row35 col1" >81.23</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level0_row36" class="row_heading level0 row36" rowspan=4>Pena High School</th> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row36" class="row_heading level1 row36" >9th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row36_col0" class="data row36 col0" >83.63</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row36_col1" class="data row36 col1" >83.81</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row37" class="row_heading level1 row37" >10th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row37_col0" class="data row37 col0" >83.37</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row37_col1" class="data row37 col1" >83.61</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row38" class="row_heading level1 row38" >11th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row38_col0" class="data row38 col0" >84.33</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row38_col1" class="data row38 col1" >84.34</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row39" class="row_heading level1 row39" >12th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row39_col0" class="data row39 col0" >84.12</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row39_col1" class="data row39 col1" >84.59</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level0_row40" class="row_heading level0 row40" rowspan=4>Rodriguez High School</th> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row40" class="row_heading level1 row40" >9th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row40_col0" class="data row40 col0" >76.86</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row40_col1" class="data row40 col1" >80.99</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row41" class="row_heading level1 row41" >10th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row41_col0" class="data row41 col0" >76.61</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row41_col1" class="data row41 col1" >80.63</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row42" class="row_heading level1 row42" >11th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row42_col0" class="data row42 col0" >76.40</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row42_col1" class="data row42 col1" >80.86</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row43" class="row_heading level1 row43" >12th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row43_col0" class="data row43 col0" >77.69</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row43_col1" class="data row43 col1" >80.38</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level0_row44" class="row_heading level0 row44" rowspan=4>Shelton High School</th> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row44" class="row_heading level1 row44" >9th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row44_col0" class="data row44 col0" >83.42</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row44_col1" class="data row44 col1" >84.12</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row45" class="row_heading level1 row45" >10th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row45_col0" class="data row45 col0" >82.92</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row45_col1" class="data row45 col1" >83.44</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row46" class="row_heading level1 row46" >11th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row46_col0" class="data row46 col0" >83.38</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row46_col1" class="data row46 col1" >84.37</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row47" class="row_heading level1 row47" >12th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row47_col0" class="data row47 col0" >83.78</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row47_col1" class="data row47 col1" >82.78</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level0_row48" class="row_heading level0 row48" rowspan=4>Thomas High School</th> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row48" class="row_heading level1 row48" >9th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row48_col0" class="data row48 col0" >83.59</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row48_col1" class="data row48 col1" >83.73</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row49" class="row_heading level1 row49" >10th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row49_col0" class="data row49 col0" >83.09</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row49_col1" class="data row49 col1" >84.25</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row50" class="row_heading level1 row50" >11th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row50_col0" class="data row50 col0" >83.50</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row50_col1" class="data row50 col1" >83.59</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row51" class="row_heading level1 row51" >12th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row51_col0" class="data row51 col0" >83.50</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row51_col1" class="data row51 col1" >83.83</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level0_row52" class="row_heading level0 row52" rowspan=4>Wilson High School</th> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row52" class="row_heading level1 row52" >9th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row52_col0" class="data row52 col0" >83.09</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row52_col1" class="data row52 col1" >83.94</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row53" class="row_heading level1 row53" >10th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row53_col0" class="data row53 col0" >83.72</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row53_col1" class="data row53 col1" >84.02</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row54" class="row_heading level1 row54" >11th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row54_col0" class="data row54 col0" >83.20</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row54_col1" class="data row54 col1" >83.76</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row55" class="row_heading level1 row55" >12th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row55_col0" class="data row55 col0" >83.04</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row55_col1" class="data row55 col1" >84.32</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level0_row56" class="row_heading level0 row56" rowspan=4>Wright High School</th> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row56" class="row_heading level1 row56" >9th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row56_col0" class="data row56 col0" >83.26</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row56_col1" class="data row56 col1" >83.83</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row57" class="row_heading level1 row57" >10th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row57_col0" class="data row57 col0" >84.01</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row57_col1" class="data row57 col1" >83.81</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row58" class="row_heading level1 row58" >11th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row58_col0" class="data row58 col0" >83.84</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row58_col1" class="data row58 col1" >84.16</td> 
    </tr>    <tr> 
        <th id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7level1_row59" class="row_heading level1 row59" >12th</th> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row59_col0" class="data row59 col0" >83.64</td> 
        <td id="T_2fb9398c_1b83_11e8_a651_74e54394bcc7row59_col1" class="data row59 col1" >84.07</td> 
    </tr></tbody> 
</table> 



## Scores by School Spending

### Following code logic shared in three reports


```python
def report_scores(df, groupby):
    '''
    Generate report of scores summary
    
    :param df: DataFrame input
    :param groupby: Summarized by field
    :return: Formatted output
    '''

    df_grouped = df.groupby(groupby).agg(
        {
            'math_score': ['mean', passing],
            'reading_score': ['mean', passing]
        }
    )
    
    # Flattern multilevel columns
    df_grouped.columns = ['_'.join(x) for x in df_grouped.columns]
    
    # Computer overal average
    df_grouped = df_grouped.assign(
        overall_rate=lambda x: (x['math_score_passing'] + x['reading_score_passing'] ) / 2
    ).rename(
        columns={
            'math_score_mean': 'Average Math Score',
            'reading_score_mean': 'Average Reading Score',
            'math_score_<lambda>': '% Passing Math',
            'reading_score_<lambda>': '% Passing Reading',
            'overall_rate': 'Overall Passing Rate (Average of the above two)'
        }
    ).reindex(
        columns = [
            'Average Math Score',
            'Average Reading Score',
            '% Passing Math',
            '% Passing Reading',
            'Overall Passing Rate (Average of the above two)'
        ]
    )
    
    return df_grouped.style.format(OUTPUT_FORMAT)
```


```python
# Computer Bin Size
(min_per_student_budget, max_per_student_budget) = school_summary['Per Student Budget'].agg(['min', 'max'])

# Substract and add 1 to handle float rounding
bins = np.arange(
        min_per_student_budget - 1,
        max_per_student_budget + 1,
         (max_per_student_budget - min_per_student_budget + 1) / 4 
)
labels = ['>={} and <{}'.format(x,y) for x,y in zip(bins, bins[1:])]

school_summary['Budget Range'] = pd.cut(school_summary['Per Student Budget'], bins, labels=labels)

report_scores(
    df_student.merge(
        school_summary, left_on='school', right_on='School Name'
    ),
    'Budget Range'
)
```




<style  type="text/css" >
</style>  
<table id="T_6cda2b70_1b6f_11e8_8468_74e54394bcc7" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >Average Math Score</th> 
        <th class="col_heading level0 col1" >Average Reading Score</th> 
        <th class="col_heading level0 col2" >% Passing Math</th> 
        <th class="col_heading level0 col3" >% Passing Reading</th> 
        <th class="col_heading level0 col4" >Overall Passing Rate (Average of the above two)</th> 
    </tr>    <tr> 
        <th class="index_name level0" >Budget Range</th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_6cda2b70_1b6f_11e8_8468_74e54394bcc7level0_row0" class="row_heading level0 row0" >>=577.0 and <596.5</th> 
        <td id="T_6cda2b70_1b6f_11e8_8468_74e54394bcc7row0_col0" class="data row0 col0" >83.36</td> 
        <td id="T_6cda2b70_1b6f_11e8_8468_74e54394bcc7row0_col1" class="data row0 col1" >83.96</td> 
        <td id="T_6cda2b70_1b6f_11e8_8468_74e54394bcc7row0_col2" class="data row0 col2" >nan%</td> 
        <td id="T_6cda2b70_1b6f_11e8_8468_74e54394bcc7row0_col3" class="data row0 col3" >nan%</td> 
        <td id="T_6cda2b70_1b6f_11e8_8468_74e54394bcc7row0_col4" class="data row0 col4" >100.0%</td> 
    </tr>    <tr> 
        <th id="T_6cda2b70_1b6f_11e8_8468_74e54394bcc7level0_row1" class="row_heading level0 row1" >>=596.5 and <616.0</th> 
        <td id="T_6cda2b70_1b6f_11e8_8468_74e54394bcc7row1_col0" class="data row1 col0" >83.53</td> 
        <td id="T_6cda2b70_1b6f_11e8_8468_74e54394bcc7row1_col1" class="data row1 col1" >83.84</td> 
        <td id="T_6cda2b70_1b6f_11e8_8468_74e54394bcc7row1_col2" class="data row1 col2" >nan%</td> 
        <td id="T_6cda2b70_1b6f_11e8_8468_74e54394bcc7row1_col3" class="data row1 col3" >nan%</td> 
        <td id="T_6cda2b70_1b6f_11e8_8468_74e54394bcc7row1_col4" class="data row1 col4" >100.0%</td> 
    </tr>    <tr> 
        <th id="T_6cda2b70_1b6f_11e8_8468_74e54394bcc7level0_row2" class="row_heading level0 row2" >>=616.0 and <635.5</th> 
        <td id="T_6cda2b70_1b6f_11e8_8468_74e54394bcc7row2_col0" class="data row2 col0" >78.48</td> 
        <td id="T_6cda2b70_1b6f_11e8_8468_74e54394bcc7row2_col1" class="data row2 col1" >81.67</td> 
        <td id="T_6cda2b70_1b6f_11e8_8468_74e54394bcc7row2_col2" class="data row2 col2" >nan%</td> 
        <td id="T_6cda2b70_1b6f_11e8_8468_74e54394bcc7row2_col3" class="data row2 col3" >nan%</td> 
        <td id="T_6cda2b70_1b6f_11e8_8468_74e54394bcc7row2_col4" class="data row2 col4" >96.0%</td> 
    </tr>    <tr> 
        <th id="T_6cda2b70_1b6f_11e8_8468_74e54394bcc7level0_row3" class="row_heading level0 row3" >>=635.5 and <655.0</th> 
        <td id="T_6cda2b70_1b6f_11e8_8468_74e54394bcc7row3_col0" class="data row3 col0" >77.42</td> 
        <td id="T_6cda2b70_1b6f_11e8_8468_74e54394bcc7row3_col1" class="data row3 col1" >81.15</td> 
        <td id="T_6cda2b70_1b6f_11e8_8468_74e54394bcc7row3_col2" class="data row3 col2" >nan%</td> 
        <td id="T_6cda2b70_1b6f_11e8_8468_74e54394bcc7row3_col3" class="data row3 col3" >nan%</td> 
        <td id="T_6cda2b70_1b6f_11e8_8468_74e54394bcc7row3_col4" class="data row3 col4" >94.8%</td> 
    </tr></tbody> 
</table> 



## Scores by School Size


```python
# Computer Bin Size
(min_size, max_size) = df_school['size'].agg(['min', 'max'])

# Substract and add 1 to handle float rounding
bins = np.arange(
        min_size - 1,
        max_size + 1,
        (max_size - min_size + 1) / 3 
)

labels = ['Small', 'Medium', 'Large']

df_school['School Size'] = pd.cut(df_school['size'], bins, labels=labels)

report_scores(
    df_student.merge(
        df_school, on='school'
    ),
    'School Size'
)
```




<style  type="text/css" >
</style>  
<table id="T_6d1ed65c_1b6f_11e8_8482_74e54394bcc7" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >Average Math Score</th> 
        <th class="col_heading level0 col1" >Average Reading Score</th> 
        <th class="col_heading level0 col2" >% Passing Math</th> 
        <th class="col_heading level0 col3" >% Passing Reading</th> 
        <th class="col_heading level0 col4" >Overall Passing Rate (Average of the above two)</th> 
    </tr>    <tr> 
        <th class="index_name level0" >School Size</th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_6d1ed65c_1b6f_11e8_8482_74e54394bcc7level0_row0" class="row_heading level0 row0" >Small</th> 
        <td id="T_6d1ed65c_1b6f_11e8_8482_74e54394bcc7row0_col0" class="data row0 col0" >83.44</td> 
        <td id="T_6d1ed65c_1b6f_11e8_8482_74e54394bcc7row0_col1" class="data row0 col1" >83.88</td> 
        <td id="T_6d1ed65c_1b6f_11e8_8482_74e54394bcc7row0_col2" class="data row0 col2" >nan%</td> 
        <td id="T_6d1ed65c_1b6f_11e8_8482_74e54394bcc7row0_col3" class="data row0 col3" >nan%</td> 
        <td id="T_6d1ed65c_1b6f_11e8_8482_74e54394bcc7row0_col4" class="data row0 col4" >100.0%</td> 
    </tr>    <tr> 
        <th id="T_6d1ed65c_1b6f_11e8_8482_74e54394bcc7level0_row1" class="row_heading level0 row1" >Medium</th> 
        <td id="T_6d1ed65c_1b6f_11e8_8482_74e54394bcc7row1_col0" class="data row1 col0" >78.16</td> 
        <td id="T_6d1ed65c_1b6f_11e8_8482_74e54394bcc7row1_col1" class="data row1 col1" >81.65</td> 
        <td id="T_6d1ed65c_1b6f_11e8_8482_74e54394bcc7row1_col2" class="data row1 col2" >nan%</td> 
        <td id="T_6d1ed65c_1b6f_11e8_8482_74e54394bcc7row1_col3" class="data row1 col3" >nan%</td> 
        <td id="T_6d1ed65c_1b6f_11e8_8482_74e54394bcc7row1_col4" class="data row1 col4" >95.6%</td> 
    </tr>    <tr> 
        <th id="T_6d1ed65c_1b6f_11e8_8482_74e54394bcc7level0_row2" class="row_heading level0 row2" >Large</th> 
        <td id="T_6d1ed65c_1b6f_11e8_8482_74e54394bcc7row2_col0" class="data row2 col0" >77.07</td> 
        <td id="T_6d1ed65c_1b6f_11e8_8482_74e54394bcc7row2_col1" class="data row2 col1" >80.93</td> 
        <td id="T_6d1ed65c_1b6f_11e8_8482_74e54394bcc7row2_col2" class="data row2 col2" >nan%</td> 
        <td id="T_6d1ed65c_1b6f_11e8_8482_74e54394bcc7row2_col3" class="data row2 col3" >nan%</td> 
        <td id="T_6d1ed65c_1b6f_11e8_8482_74e54394bcc7row2_col4" class="data row2 col4" >94.6%</td> 
    </tr></tbody> 
</table> 



## Scores by School Type


```python
report_scores(
    df_student.merge(
        df_school, on='school'
    ),
    'type'
)
```




<style  type="text/css" >
</style>  
<table id="T_6d65cb5e_1b6f_11e8_ae2a_74e54394bcc7" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >Average Math Score</th> 
        <th class="col_heading level0 col1" >Average Reading Score</th> 
        <th class="col_heading level0 col2" >% Passing Math</th> 
        <th class="col_heading level0 col3" >% Passing Reading</th> 
        <th class="col_heading level0 col4" >Overall Passing Rate (Average of the above two)</th> 
    </tr>    <tr> 
        <th class="index_name level0" >type</th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_6d65cb5e_1b6f_11e8_ae2a_74e54394bcc7level0_row0" class="row_heading level0 row0" >Charter</th> 
        <td id="T_6d65cb5e_1b6f_11e8_ae2a_74e54394bcc7row0_col0" class="data row0 col0" >83.41</td> 
        <td id="T_6d65cb5e_1b6f_11e8_ae2a_74e54394bcc7row0_col1" class="data row0 col1" >83.90</td> 
        <td id="T_6d65cb5e_1b6f_11e8_ae2a_74e54394bcc7row0_col2" class="data row0 col2" >nan%</td> 
        <td id="T_6d65cb5e_1b6f_11e8_ae2a_74e54394bcc7row0_col3" class="data row0 col3" >nan%</td> 
        <td id="T_6d65cb5e_1b6f_11e8_ae2a_74e54394bcc7row0_col4" class="data row0 col4" >100.0%</td> 
    </tr>    <tr> 
        <th id="T_6d65cb5e_1b6f_11e8_ae2a_74e54394bcc7level0_row1" class="row_heading level0 row1" >District</th> 
        <td id="T_6d65cb5e_1b6f_11e8_ae2a_74e54394bcc7row1_col0" class="data row1 col0" >76.99</td> 
        <td id="T_6d65cb5e_1b6f_11e8_ae2a_74e54394bcc7row1_col1" class="data row1 col1" >80.96</td> 
        <td id="T_6d65cb5e_1b6f_11e8_ae2a_74e54394bcc7row1_col2" class="data row1 col2" >nan%</td> 
        <td id="T_6d65cb5e_1b6f_11e8_ae2a_74e54394bcc7row1_col3" class="data row1 col3" >nan%</td> 
        <td id="T_6d65cb5e_1b6f_11e8_ae2a_74e54394bcc7row1_col4" class="data row1 col4" >94.5%</td> 
    </tr></tbody> 
</table> 


