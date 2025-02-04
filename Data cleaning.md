# Table of Contents

1. [Summary of Data Cleaning Steps](#summary-of-data-cleaning-steps)
2. [Dataset](#dataset)
    - [Inspect dataset](#inspect-dataset)
    - [Observations on data](#observations-on-data)
3. [Data Preprocessing](#data-preprocessing)
    - [Convert column dtypes](#convert-column-dtypes)
    - [Item column](#rectify-item-column)
         - [Dictionary Mapping Prices to Items](#2-mapping-prices-to-items)
         - [Data Reconciliation](#3-data-reconciliation)
    - [Price Per Unit column](#rectify-price-per-unit-column)
         - [Reconcile NaN values](#reconcile-nan-price-per-unit-values)
    - [Quantity column](#rectify-quantity-column)
    - [Total Spent column](#rectify-total-spent-column)
4. [Inspect cleaned data](#inspect-cleaned-data)
   
<br>
<br>

### Summary of Data Cleaning Steps


- **Data Validation and De-duplication:**  
  Identified and resolved NaN values and duplicate records to ensure data integrity.  

- **Data Type Standardization:**  
  Converted columns to appropriate data types to align with analytical requirements and improve computational efficiency.  

- **Item Name Mapping:**  
  Leveraged dictionaries to map derived price values to corresponding item names, ensuring accurate categorization.  

- **Unit Price Derivation:**  
  Computed 'Price Per Unit' by dividing the 'Total Spent' column by the 'Quantity' column.  

- **Error Filtering:**  
  Removed rows with unresolved discrepancies in the 'Item' or 'Price Per Unit' columns after mapping operations.  

- **Quantity Adjustment:**  
  Corrected values in the 'Quantity' column by recalculating it as the quotient of 'Total Spent' and 'Price Per Unit.'  

- **Column Recalculation:**  
  Revalidated the 'Total Spent' column by recomputing it as the product of 'Quantity' and 'Price Per Unit.'  

## Dataset

``` python
import numpy as np 
import pandas as pd 

df = pd.read_csv('downloads/dirty_cafe_sales.csv')

```
``` python
print(df.head(50))
```

<img src=images/df.png width="900" height="900"/>

## Inspect dataset

``` python
print(df.info())
```
<img src=images/sinfo.png width="350" height="250"/>


``` python
print(df.isna().sum())
print(f'\nThere are {df.duplicated().sum()} duplicate rows in dataset.')
```
<img src=images/sumna.png width="300" height="250"/>

``` python
# inspect unique values 
for col in df.columns[1:5]:  
    print(f'{col} : {df[col].unique()}')
```
<img src=images/pre.png width="600" height="100"/>

``` python
txn_count = df["Transaction ID"].value_counts().sort_values(ascending = False)
txn_count
```
<img src=images/txn.png width="320" height="250"/>

### Observations on data

##### 1. There are no recurring Transaction IDs. <br>
##### 2. Messy data 
1. Unknown data in Item column
    - Use item_price dictionary to determine the item
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

## Data Preprocessing

### Convert column dtypes
- Convert column types to suitable dtypes.
- Dictionary mapping specific columns to their desired data types.  
- Iterate through the dictionary, converting columns to either numeric or datetime formats using `pd.to_numeric` and `pd.to_datetime`, with `errors="coerce"` to handle invalid values.  
- Ensure missing values in integer columns are replaced with `0` before converting to integer type.  

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

### Rectify Item column

#### Inspect Item values

``` python
print(f'There are {df["Item"].nunique()} unique items sold.')
print(f'Unique values in Item category: \n {df["Item"].unique()}')
```

<img src=images/itemin.png width="600" height="100"/>

### Steps to clean 'Items'
Using items_prices dict, replace incorrect values in 'Item' column using 'Price Per Unit' column values 

#### 1. Create Item Dictionary
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

# list of item names
correct_item_values = ['Coffee','Cake','Cookie','Salad','Smoothie', 'Sandwich', 'Juice' , 'Tea']

df["Price Per Unit"] = df["Total Spent"] / df["Quantity"]

# filter by 'Item' with correct item names, then use '~' to negate that, leaving all incorrect item names
df_item_incorrect = df[~df["Item"].isin(correct_item_values)]
(df_item_incorrect).head(50)


# generate dictionary of price-item
price_to_item = {value: key for key, value in items_prices.items()}

```
#### 2. Mapping Prices to Items 
- Replace incorrect "Item" values in a DataFrame by mapping the "Price Per Unit" column to a dictionary (price_to_item). 
- If a price exists in the dictionary, the corresponding item name is assigned; otherwise, the "Item" value is set to "No Price per unit."
- The unique updated items are printed, and rows with missing item names are identified and displayed for further review.

``` python
# use the index of rows in df_item_incorrect to replace incorrect values with the price dictionary 
for index in df_item_incorrect.index:
    # If the "Price Per Unit" exists in the price_to_item dictionary, replace "Item" with the corresponding key.
    # If no match is found in price_to_item, set "Item" to "No Price per unit".
    df.loc[index,"Item"] = price_to_item.get(df.loc[index, "Price Per Unit"], "No Price per unit")
    
print(df["Item"].unique())


missing_item_name = df[df["Item"]=="No Price per unit"]
print(missing_item_name)
```

<img src=images/map_dict.png width="850" height="300"/>

#### 3. Data Reconciliation 

- **Price Per Unit Calculation:**  
  Iterates through rows in the `missing_item_name` DataFrame, calculating the 'Price Per Unit' by dividing 'Total Spent' by 'Quantity' with error handling for non-numeric values.  

- **Unresolvable Rows Handling:**  
  Drops rows where the 'Price Per Unit' equals $4 or $3, as these prices correspond to multiple items, making item reconciliation not possible.  

- **Item Mapping via Dictionary:**  
  Matches the calculated 'Price Per Unit' to the `price_to_item` dictionary to assign the corresponding 'Item' name.  

- **Dropping rows:**  
  Drops rows where the 'Price Per Unit' cannot be matched to the dictionary or where errors in 'Total Spent' or 'Quantity' prevent accurate calculations.  

- **Validation:**  
  Outputs any remaining rows with unresolved 'Item' values for further review or verification.  

``` python
# Loop through index rows to get Price per unit value
for index in missing_item_name.index:
    df.loc[index,"Price Per Unit"] = pd.to_numeric(df.loc[index,"Total Spent"] / df.loc[index,"Quantity"], errors = "coerce")
    # drop rows that have price per unit of $4 or $3 because the Item name is not reconciliable 
    # there are items with same price of $4 or $3, unable to determine the item name 
    if df.loc[index,"Price Per Unit"] == 4 or df.loc[index,"Price Per Unit"] == 3:
        df.drop(index, axis = 0, inplace=True)
    elif df.loc[index,"Price Per Unit"] in price_to_item.keys():
        # match the 'price per unit' to 'price_to_item' dictionary to get item names 
        df.loc[index,"Item"] = price_to_item.get(df.loc[index,"Price Per Unit"], "To be dropped")
    else:
        # drop all rows. Means that these rows have errors either columns (Total Spent or Quantity)
        # so we can determine neither item name nor price per unit
        df.drop(index, axis = 0, inplace=True)
        
print(df[df["Item"] == "No Price per unit"])
```
<img src=images/empty.png width="900" height="70"/>

### Rectify 'Price Per Unit' column 

#### Reconcile NaN 'Price Per Unit' values 
1. Use price dictionary to match Item names
2. Use 'Total Spent' / 'Quantity'
3. Remove rows from the dataFrame where the "Price Per Unit" column contains infinite values.


``` python
missing__price_per_unit = df[df["Price Per Unit"].isna()]

for index in missing__price_per_unit.index:
    df.loc[index,"Price Per Unit"] = items_prices.get(df.loc[index, "Item"], df.loc[index,"Price Per Unit"])

df.loc[df["Price Per Unit"].isna(), "Price Per Unit"] = pd.to_numeric(
    df["Total Spent"] / df["Quantity"], 
    errors = "coerce")

print(sorted(df["Price Per Unit"].unique()))
```
<img src=images/inf.png width="400" height="40"/>

``` python
df = df.drop(df.loc[df["Price Per Unit"]== np.inf].index, axis = 0)
print(sorted(df["Price Per Unit"].unique()))
```
<img src=images/noinf.png width="300" height="40"/>

### Rectify Quantity column

- Check 'Quantity' values for errors
- Identified and corrected zero values in the Quantity column by recalculating them using the Total Spent and Price Per Unit columns.
- Any remaining rows with NaN values in Quantity were removed to ensure data integrity.
  
``` python
print(np.sort(df["Quantity"].unique()))
```
<img src=images/ar.png width="200" height="40"/>


``` python
# Identify values that are 0 in Quantity column 

df.loc[df["Quantity"] == 0, "Quantity"] = pd.to_numeric(
    df["Total Spent"] / df["Price Per Unit"], 
    errors='coerce')

print(sorted(df["Quantity"].unique()))

```
<img src=images/arn.png width="250" height="35"/>

``` python
# extract rows that still contain NaN values in Quantity
missing_qty = df[df["Quantity"].isna()]
print(missing_qty.head(50))

```

<img src=images/missing_qty.png width="900" height="400"/>

``` python
df = df.drop(missing_qty.index, axis = 0, errors = "ignore")
print(sorted(df["Quantity"].unique()))
```
<img src=images/qty.png width="250" height="35"/>


### Rectify 'Total Spent' column

- Identify missing values in the Total Spent column
- Recalculate and fill missing Total Spent values using 'Quantity' and 'Price Per Unit' values
- Address NaN values to ensure data completeness

``` python
missing_total_spent = df[df["Total Spent"].isna()]
print(missing_total_spent)
```
<img src=images/miss_ts.png width="700" height="350"/>

``` python
# Recalculate and fill missing values in the "Total Spent" column by multiplying 
# the corresponding "Quantity" and "Price Per Unit" for rows where "Total Spent" is NaN
df.loc[df["Total Spent"].isna(), "Total Spent"] = df.loc[df["Total Spent"].isna(), "Quantity"] * df.loc[df["Total Spent"].isna(), "Price Per Unit"]

print(sorted(df["Total Spent"].unique()))
```
<img src=images/ts.png width="70" height="350"/>


## Inspect cleaned data

``` python
for col in df.columns[1:5]:  
    print(f'{col} : {sorted(df[col].unique())}')
```

<img src=images/cleaned_data.png width="780" height="90"/>
