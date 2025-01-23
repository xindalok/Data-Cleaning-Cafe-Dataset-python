``` python
import numpy as np # linear algebra
import pandas as pd # data processing, CSV file I/O (e.g. pd.read_csv)

df = pd.read_csv('downloads/dirty_cafe_sales.csv')

```
``` python
print(df.head(50))
```

<img src=images/df.png width="900" height="900"/>

### Inspect dataset

``` python
print(df.info())

print(df.isna().sum())
print(f'\nThere are {df.duplicated().sum()} duplicate rows in dataset.')
```

``` python
txn_count = df["Transaction ID"].value_counts().sort_values(ascending = False)
txn_count
```

<img src=images/odf.png width="400" height="300"/>
<img src=images/sumna.png width="300" height="250"/>
<img src=images/txn.png width="320" height="250"/>

### Comments on data

##### 1. There are no recurring Transaction IDs. <br>
##### 2. Messy data 
1. Unknown data in Item column
    - Use 'Price per unit' to determine the item
2. Error values in Quantity column
    - Use 'Total Spent' / 'Price per unit' to get value
3. Error values in 'Total Spent' column
    - Use 'Quantity' * 'Price per unit' to get value
4. Unknown / NaN / Error values in 'Payment Method' column
    - Have to determine if we need to drop them or just leave them
5. Unknown / NaN / Error values in 'Location' column
    - Have to determine if we need to drop them or just leave them
6. Unknown / NaN / Error values in 'Transaction Date' column
    - Have to determine if we need to drop them or just leave them

##### 3. Dtypes
1. To be changed to integer: 
    - Quantity
2. To be changed to float: 
    - Price per unit
    - Total Spent
3. To be changed to datetime: 
    - Transaction Date

### Data Preprocessing

Convert column dtypes

``` python 
columns_to_convert = {
    "Quantity": "int",
    "Price Per Unit": "float",
    "Total Spent": "float",
    "Transaction Date": "datetime64[ns]"
}

for col, dtype in columns_to_convert.items():
    if dtype.startswith("datetime"):
        df[col] = pd.to_datetime(df[col], errors="coerce")
    else:
        df[col] = pd.to_numeric(df[col], errors="coerce")
    # Handle missing values after coercion (optional)
    if dtype == "int":
        df[col] = df[col].fillna(0).astype("int")

df.info()
```
<img src=images/dtypes.png width="500" height="300"/>

----------

## Rectify Item column

### Inspect Item values

``` python
print(f'There are {df["Item"].nunique()} unique items sold.')
print(f'Unique values in Item category: \n {df["Item"].unique()}')
```

<img src=images/itemin.png width="600" height="120"/>

### Rectify 'Item' values
- Create dictionary of item to price
- Create dataset that has incorrect values in 'Item' column
- Replace the values accordingly using the new dictionary
   - Using dict, replace incorrect values in 'Item' column by cross-referencing 'Price Per Unit' column values


``` python
items_prices = {
    "Coffee": 2,
    "Tea": 1.5,
    "Sandwich": 4,
    "Salad": 5,
    "Cake": 3,
    "Cookie": 1,
    "Smoothie": 4,
    "Juice": 3
}
```
