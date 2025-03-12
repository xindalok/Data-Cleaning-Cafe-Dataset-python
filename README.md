# Data Cleaning Project

### Project overview
This project focuses on cleaning and standardizing a raw café sales dataset to ensure accuracy and consistency for analysis. The cleaned dataset provides a reliable foundation for business insights and decision-making.

Key tasks included handling missing values, correcting erroneous data in item names, prices, and quantities, and standardizing data types. Logical recalculations were applied to rectify inconsistencies in financial columns. 

### Data source
https://www.kaggle.com/datasets/ahmedmohamed2003/cafe-sales-dirty-data-for-cleaning-training
<br><br>
### Tools 
Python - Jupyter Notebook
<br><br>

### **Data Issues & Preprocessing Summary**  

The dataset contained **missing, incorrect, and inconsistent values** in multiple columns, including 'Item,' 'Quantity,' 'Total Spent,' 'Payment Method,' 'Location,' and 'Transaction Date.' <br><br>
Errors in quantity and total spent were recalculated, and unknown item names were inferred using 'Price per Unit.' <br> 
Missing values were **evaluated for retention or removal**, and **data types were corrected** for numerical and datetime fields. <br>
<br>
These steps ensure clean, structured, and accurate data for analysis. 
<br><br>
### **Fixing Missing Item Details with Price and Quantity Logic**
The code cross-references item names and prices to reconcile missing or incorrect values. It checks if an item's price per unit matches a known price in a dictionary and fills in the correct name. If the price per unit is missing, it calculates it using 'Total Spent' and 'Quantity.' When prices are ambiguous (like $3 or $4) or both 'Total Spent' and 'Quantity' contain errors, the row is removed to maintain data accuracy.
