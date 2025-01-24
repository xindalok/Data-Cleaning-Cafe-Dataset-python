### Summary of Data Cleaning Steps  

- **Data Validation and De-duplication:**  
  Identified and resolved NaN values and duplicate records to ensure data integrity.  

- **Data Type Standardization:**  
  Converted columns to appropriate data types to align with analytical requirements and improve computational efficiency.  

- **Item Name Mapping:**  
  Leveraged a dictionary to map derived price values to corresponding item names in the 'Item' column, ensuring accurate categorization.  

- **Unit Price Derivation:**  
  Computed 'Price Per Unit' by dividing the 'Total Spent' column by the 'Quantity' column.  

- **Error Filtering:**  
  Removed rows with unresolved discrepancies in the 'Item' or 'Price Per Unit' columns after mapping operations.  

- **Quantity Adjustment:**  
  Corrected values in the 'Quantity' column by recalculating it as the quotient of 'Total Spent' and 'Price Per Unit.'  

- **Column Recalculation:**  
  Revalidated the 'Total Spent' column by recomputing it as the product of 'Quantity' and 'Price Per Unit.'  


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

#### 3. Dropping rows

missing_item_name = df[df["Item"]=="No Price per unit"]
- This gives a dataset that has no 'Item' AND 'Price per Unit' entry. 
- Use 'Total Spent' / 'Quantity' to get 'Price per Unit'
- Drop rows where I still get an error after the step above

### 4. Price and Item Data Reconciliation 

- **Price Per Unit Calculation:**  
  Iterates through rows in the `missing_item_name` DataFrame, calculating the 'Price Per Unit' by dividing 'Total Spent' by 'Quantity' with error handling for non-numeric values.  

- **Unresolvable Rows Handling:**  
  Drops rows where the 'Price Per Unit' equals $4 or $3, as these prices correspond to multiple items, making item reconciliation infeasible.  

- **Item Mapping via Dictionary:**  
  Matches the calculated 'Price Per Unit' to the `price_to_item` dictionary to assign the corresponding 'Item' name.  

- **Error Filtering:**  
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
<img src=images/empty.png width="7000" height="80"/>

