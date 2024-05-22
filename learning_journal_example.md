# Example of a learning journal with code snippets

Create your own learning journal with your own code snippets.



## Variables

### How to modify a global variable from within a function in Python

Use the keyword <b>global</b> variable_name

```Python
def rl():
    global output_file # tells pythong to use the global variable output_file instead of creating a local variable
    output_file = generate_file_name("output")
    print(output_file)
    %run "./openpyxl-utils.ipynb"    
    return
```    
    
## Data acquisition


### Using a file browser

```Python
from tkinter.filedialog import *

// this line would pop a file browser
file_w_path = askopenfile()

file_w_path

// e.g. file_w_path could contain a string value like 'U:/data/test.xls'
```



```Python
xls = pd.ExcelFile(input_file)
df = pd.read_excel(xls, 'All staff (daily refresh)', skiprows=7)
```



## EDA (exploratory data analysis)

## Sorting

### Sort with multiple keys

```Python
df.sort_values(['Area_Sort','Sub_Area','Name'],ascending=True,inplace=True)
```

### Pivot (groupby)

#### Adding a total row

```Python
df.loc['Total']= df.sum(numeric_only=True, axis=0)
```

### Combining datasets

#### Concat

```Python
dfn_pre=dfn.add_prefix("dfn_")
dfp_pre=dfp.add_prefix("dfp_")
dfn_pre.set_index(['dfn_id'],inplace=True) # setting the employee ID as index
dfp_pre.set_index(['dfp_id'],inplace=True) # setting the employee ID as index
df = pd.concat([dfp_pre,dfn_pre],axis=1) # combines the 2 DataFrames into 1 by combining columns from both using the index as key.
```

## Transformation and feature engineering

### List comprehension to rename columns with regular expression

Replace the hashtags by 'N' to make it easier to use the column names

```Python
df.columns=[re.sub('#', "N", col) for col in df.columns]
```

### Lambda functions to extract values from columns

#### Split to retrieve the first element

note: split gives the flexibility to have an element of variable length such as "AD" or "MDSA"

```Python
dfn["ProposedRank"] = dfn.PromotionRequest.str.split().map(lambda x: x[0])
```

### Store a percentage value in a dataframe and prevent division by zero nan% errors

```Python
import math

df_rate['Perc_HC']=(df_rate.Nominated_/df_rate.dfp_Name).map(lambda num: '0.0%' if math.isnan(num) else '{0:.1f}%'.format(round(num * 100, 1)))
df_rate['Perc_Elig']=(df_rate.Nominated_/df_rate.dfp_Eligible).map(lambda num: '0.0%' if math.isnan(num) else '{0:.1f}%'.format(round(num * 100, 1)))
```



### Modifying values of filtered rows using .loc

Python requires the use of .loc to identify rows of a DataFrame which are modified.

NOTE: the use of <b>.loc[mask,"column_name"]</b>

```Python
# creates the new column containing a datetime delta
dfp['DaysSinceLastPromotion'] = (datetime.datetime.today() - dfp.LastPromotionDate) 
# converts the delta to a number of days
dfp.loc[~dfp.LastPromotionDate.isnull(),'DaysSinceLastPromotion'] = dfp.loc[~dfp.LastPromotionDate.isnull(),'DaysSinceLastPromotion'].dt.days 
# convers to a number of years (with decimals)
dfp['YearsSinceLastPromotion']= dfp.DaysSinceLastPromotion/365.0 
```

### Transforming boolean to int (1 for True and 0 for False)

```Python
dfp['Eligible']=dfp.PromotionEligibilityWithSec=='Meets all criteria'  # boolean value
dfp['Eligible_']=dfp.Eligible.astype(int) # int value
```

### Lambda mean function

If you have a dataframe with multiple columns for which you need to calculate a mean value while exluding null and 0 entries, you can write your own cusom mean function.

The alternative would be to filter our rows with null or 0 values but since these rows might be different for each columns, you'd need to do it column by column.

```Python
custom_mean = lambda x: x[(x != 0) & pd.notna(x)].mean()
hopr = opr.groupby(['Area_Code']).agg(custom_mean)[['Opr_2018','Opr_2019','Opr_2020','Opr_2021','Opr_2022']]

# creating a total mean row
totals = opr[['Opr_2018','Opr_2019','Opr_2020','Opr_2021','Opr_2022']].apply(custom_mean)
df_total = pd.DataFrame([totals.values], columns=totals.index,index=['Total'])

# adding the total row to the dataframe
df_OPRs = pd.concat([hopr, df_total])
df_OPRs
```
### Renaming multi-level index headers and values

Given a dataframe with a 2-level index structure, how do I rename the index headers and values

```Python
import pandas as pd

data = {
    'Value': [10, 20, 30, 40, 15, 25, 35, 45]
}

index = pd.MultiIndex.from_product([['A', 'B', 'C', 'D'], ['X', 'Y']], names=['Level1', 'Level2'])

df = pd.DataFrame(data, index=index)
df
```

```text
	        
                Value
Level1	Level2	
A	    X	    10
            Y	    20
B	    X	    30
            Y	    40
C	    X	    15
            Y	    25
D	    X	    35
            Y	    45
```

#### Renaming the index headers

```Python
# Rename the index headers
df = df.rename_axis(index={'Level1': 'New_Level1', 'Level2': 'New_Level2'})
```

#### Renaming the index values

```Python
# Rename Level1 and Level2 values
df.index.set_levels(['New_A', 'New_B', 'New_C', 'New_D'], level='Level1', inplace=True)
df.index.set_levels(['New_X', 'New_Y'], level='Level2', inplace=True)
```


## Visualization

### using a lambda funciton to show some columns as %

Given a dataframe with nominal data in the first 3 rows, we want to calculate the percentages in the last 3 rows then display the data with "%" sign.

~~~Python
gtmat=df.groupby(['Work pensum group']).agg({'dfp_Name':'count', 'dfp_Eligible_': 'sum','Nominated_':'sum'})
gtmat.index.names=['Work pensum group']
gtmat.columns=['Headcount','Eligible','Nominated']

gtmat['Nomination split'] = 100 * gtmat.Nominated/gtmat.Nominated.sum()
gtmat['Eligible split'] = 100 * gtmat.Eligible/gtmat.Eligible.sum()
gtmat['Headcount split'] = 100 * gtmat.Headcount/gtmat.Headcount.sum()
gtmat=gtmat.fillna(0)

# adding the total row
gtmat.loc['Total']=gtmat.sum(axis=0,numeric_only=True)

# using lambda functions to display the percentages as 0% if null, 100% if 100.0% and otherwise xx.x% (value with 1 decimal point)
gtmat['Eligible split']=gtmat['Eligible split'].map(lambda num: '0%' if math.isnan(num) else '100%' if (num==100.0)  else  '{0:.1f}%'.format(num))
gtmat['Headcount split']=gtmat['Headcount split'].map(lambda num: '0%' if math.isnan(num) else '100%' if (num==100.0)  else  '{0:.1f}%'.format(num))    
gtmat['Nomination split']=gtmat['Nomination split'].map(lambda num: '0%' if math.isnan(num) else '100%' if (num==100.0)  else  '{0:.1f}%'.format(num))    

~~~

smarter way to apply the lambda funciton on the last 3 columns in 1 line using ```.iloc```


~~~Python
gtmatESG.iloc[:, -3:] = gtmatESG.iloc[:, -3:].applymap(lambda num: '0%' if math.isnan(num) else '100%' if (num==100.0)  else  '{0:.1f}%'.format(num))
~~~        
