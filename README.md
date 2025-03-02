# RFM-Marketing-Analysis-with-Python 


## Table of Contents
1. [Introduction](#introduction)
   
2. [Exploratory data analysis (EDA)](#exploratory-data-analysis)
   
   2.1 [Data description](#data-description)

   2.2 [Data cleaning](#data-cleaning)

   2.3 [Customer segmentation using Recency-Frequency-Money framework](#receny-frequency-money)

   
3. [Visualizations and Insights](#visualizations-and-insights)
   
4. [Recommendations](#recommendations)

## 1. Introduction

The global retail chain SuperStore needs to come up with a marketing strategy for Christmas and New Year holidays. This year, they aim to focus on serving existing customers rather than acquiring new ones. SuperStore has two goals through the marketing campaign. First, they want to deliver their appreciation towards current customers. Second, they want to convert a group of existing buyers into loyal ones.

The first step in planning a marketing plan is understanding customer profiles. A data analyst will help marketers to segment existing customers. Marketers then can tailor marketing plan suitable for each customer group. Thus, SuperStore can effectively deliver its thanks to existing customers. Additionally, marketers can also identify the characteristics of customers who can be converted into loyalists to SuperStore. 

Customer segmentation can be based on different criteria. Recency - Frequency - Monetary are three useful criteria for companies operating in the retail industry. Thus, SuperStore's analyst will employ this framework to categorize existing buyers.


## 2. Exploratory data analysis
### 2.1 Data description
The dataset contains all transactions occurring between 1/12/2010 and 9/12/2011. All transactions were made in a UK-based and online retail. SuperStore mainly sells unique all-occasion gifts. Many customers are wholesalers. 

There are 541 909 data points in the dataset. It contains 8 variables:
| Code | Variable | Data type | Explanation|
|----------|----------|----------|----------|
| InvoiceNo     | Invoice number   | Nominal   |6-digit integral number uniquely assigned to each transaction. InvoiceNo starting with 'C' indicates a cancellation.|
| StockCode     | Product code   | Nominal   |5-digit integral number uniquely assigned to each product.|
| Description     | Product name   | Nominal   ||
| Quantity     | Quantity of each product per transaction   | Numeric   ||
| InvoiceDate     | Invoice date and time   | Numeric  |Day and time the transaction was generated.|
| UnitPrice     | Unit price  | Numeric  | Product price per unit in sterling|
| CustomerID     | Customer number  | Nominal  | 5-digit number uniquely assigned to each customer.|
| Country     | Country name  | Nominal  |Name of the country where each customer resides.|


### 2.2 Data cleaning

#### Missing values and Data type
info() of the data is useful in checking missing values and data type for each column.

```python
# Load the Excel file
df_original = pd.read_excel('/Users/nhuyenhuynh/Downloads/Final_project_RFM/ecommerce retail.xlsx', sheet_name='ecommerce retail')
      
df_original.info()
```

<img src="https://github.com/user-attachments/assets/41ced5d2-ae8d-4055-aba3-1e63e3732673" width="400"/>

- The dataset contains 541909 rows and 8 columns. 
- "Description" and "CustomerID" columns have 1454 and 135080 missing values respectively.
- The data type of all variables are well defined.

<br> 
<br>

Now, it is important to deal with missing values. In the context of RFM analysis, information about what kind of products customers buy is not important. Hence, I will remove "Description" column from the dataset. A transaction without customer id is useless in the RFM analysis; thus, I drop all rows without customer id. 

Besides, only successful purchase should be counted. Hence, I will drop all cancelled orders which have the "InvoiceNo" starting with the letter "C". The code is as follow:

```python
# Drop the "Description" column
df = df.drop('Description', axis='columns')

# Drop all rows without customer id
df = df[~df['CustomerID'].isnull()]

# Drop all cancelled orders
df = df[~df['InvoiceNo'].str.contains('c', case=False, na=False)]
```

<br>
<br>

After dealing with missing values and removing all cancellation, I use info() to see the changes in the dataset:

```python
df.info()
```

<img src="https://github.com/user-attachments/assets/312f41e3-e444-46fe-9265-d36b9125a895" width="400"/>

The dataset now has 397924 rows and 7 columns. It is free from missing values.

<br>

#### Duplicates
Normally, customers buy more than one type of products in a transaction. Thus, I assume that the combination of invoice number ("InvoiceNo") and product code ("StockCode") is the primary key. In this context, duplicates are defined as rows with the same combination of 'InvoiceNo' and 'StockCode'.

```python
# Remove duplicates with the same combination of 'InvoiceNo' and 'StockCode'
df_no_duplicates = df.drop_duplicates(subset=['InvoiceNo', 'StockCode'])

# See the changes after removing duplicates
df_no_duplicates.info()

```

<img src="https://github.com/user-attachments/assets/cdcf7ca6-9db5-48be-8baf-ac66aed45e09" width="400">

The dataset now contains 387875 rows and 7 columns. It is free from duplicates.

### 2.3 Customer segmentation using Recency-Frequency-Money framework

#### Generate RFM data for each customer

First, it is important to define RFM framework:
- Recency (R) measures the time since the last purchase or interaction. A smaller value indicates higher engagement while a higher value indicates lower engagement from the customer.
- Frequency (F) measures how often a customer makes a purchase or interacts with a business within a specific period.
- Monetary (M) measures how much money a customer spends during a specific period. A higher monetary value indicates the customer has spent more and more engaged with the company and vice versa.

The desired output is a dataframe with 4 columns: CustomerID, Recency, Frequency, Monetary. The code is as follow:

```python
# create the dataframe called df_rfm with the first column of MaxDate
df_rfm = df.groupby('CustomerID')['InvoiceDate'].max().to_frame()

# Create a new column containing the number of transactions each customer made
df_rfm['Frequency'] = df.groupby('CustomerID')['InvoiceNo'].nunique()

# Create the Recency column
specific_date = pd.to_datetime('2011-12-31')
df_rfm['Recency'] = (specific_date - df_rfm['InvoiceDate']).dt.days

# Create Value column
df['Price'] = df['UnitPrice']*df['Quantity']
df_rfm['Monetary'] = df.groupby('CustomerID')['Price'].sum()
df_rfm = df_rfm.reset_index()

# Finalize RFM dataframe
df_rfm = df_rfm[['CustomerID', 'Frequency', 'Recency', 'Monetary']].copy()

# Express 5 first values of RFM dataframe
df_rfm.head()
```
The RFM looks like this:

<img src="https://github.com/user-attachments/assets/dda971f9-3c3b-41c7-8dd9-253c5ede3719" width=300>

Info() of RFM dataframe:
```python
df_rfm.info()
```
<img src="https://github.com/user-attachments/assets/09efc650-5307-4a7d-9936-f081d56e72b5" width=300>

The output shows that SuperStore has 4339 existing customers in the period of 1/12/2010 and 9/12/2011. All columns are free from missing values. 




## 3. Visualizations and Insights
## 4. Recommendations

