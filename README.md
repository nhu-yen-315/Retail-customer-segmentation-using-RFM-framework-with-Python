# Analyze customer segmentation of SuperStore - a global retail chain with Python 
<img src="https://github.com/user-attachments/assets/0c3ec864-b594-46e6-9739-f1d5d234298d" width=700>

(Source: Drapers)

Author: Huỳnh Như Yến <br>
Date: 4/3/2024 <br>
Tool: Python [Library: Pandas, Seaborn]

## Table of Contents 📋
1. [Introduction](#introduction)
   
2. [Exploratory data analysis (EDA)](#exploratory-data-analysis)
   
   2.1 [Data description](#data-description)

   2.2 [Data cleaning](#data-cleaning)

   2.3 [Customer segmentation using Recency-Frequency-Money framework](#receny-frequency-money)

   
3. [Visualizations and Insights](#visualizations-and-insights)
   
4. [Recommendations](#recommendations)

## 1. Introduction

**Context:** 
<br>
The global retail chain SuperStore is preparing a marketing strategy for Christmas and New Year holidays. The marketing campaign has two main goals: <br>
      
            🎯 Show appreciation to customers making purchase this year 
      
            🎯 Convert more customers to loyalists 

To support the marketing team, data analyst team will analyse the **transaction database** to **segment customers using Recency - Frequency - Monetary framework**.

**👤 Who is this project for?**

Marketers

Data analysts

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

 🔑 Function: info() to display missing values and data types.

```python
# Load the Excel file
df_original = pd.read_excel('/Users/nhuyenhuynh/Downloads/Final_project_RFM/ecommerce retail.xlsx', sheet_name='ecommerce retail')
      
df_original.info()
```

<img src="https://github.com/user-attachments/assets/41ced5d2-ae8d-4055-aba3-1e63e3732673" width="400"/>


✅ The dataset contains 541909 rows and 8 columns. 

✅ "Description" and "CustomerID" columns have 1454 and 135080 missing values respectively.

✅ The data type of all variables are well defined.

--------------------------------------------------------------------

#### ❓❓❓ Solution to missing values
 
- In the context of RFM analysis, information about what kind of products customers buy is not important. Hence, I will **remove "Description" column** from the dataset. 

- A transaction without customer id is useless in the RFM analysis; thus, I **drop all rows without customer id**. 

- Besides, only successful purchase should be counted. Hence, I will **drop all cancelled orders** which have the "InvoiceNo" starting with the letter "C". The code is as follow:

```python
# Drop the "Description" column
df = df.drop('Description', axis='columns')

# Drop all rows without customer id
df = df[~df['CustomerID'].isnull()]

# Drop all cancelled orders
df = df[~df['InvoiceNo'].str.contains('c', case=False, na=False)]

df.info()
```
<br>

<img src="https://github.com/user-attachments/assets/312f41e3-e444-46fe-9265-d36b9125a895" width="400"/>

<br>
✅ The dataset now has 397924 rows and 7 columns. It is free from missing values.

<br>

--------------------------------------------------------------------

#### ❓❓❓ Solution to Duplicates

Normally, customers buy more than one type of products in a transaction. Thus, I assume that the combination of invoice number ("InvoiceNo") and product code ("StockCode") is the primary key. In this context, **duplicates are defined as rows with the same combination of 'InvoiceNo' and 'StockCode'.**

```python
# Remove duplicates with the same combination of 'InvoiceNo' and 'StockCode'
df_no_duplicates = df.drop_duplicates(subset=['InvoiceNo', 'StockCode'])

# See the changes after removing duplicates
df_no_duplicates.info()

```

<img src="https://github.com/user-attachments/assets/cdcf7ca6-9db5-48be-8baf-ac66aed45e09" width="400">

✅ The dataset now contains 387875 rows and 7 columns. It is free from duplicates.

### 2.3 Customer segmentation using Recency-Frequency-Money framework

⚒️ 3 steps: Generate absolute RFM -> Generate relative RFM -> Customer segmentation

#### ❓❓❓ Step 1: Generate absolute RFM 

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

<br>

--------------------------------------------------------------------
#### ❓❓❓ Step 2: Generate relative RFM (1-5 scale)

The **scale of 1 to 5** can be used to segment RFM into 5 groups. Before applying quintile, it is important to **check for outliers**. 

- Recency:
  ```python
  sns.boxplot(data= df_rfm['Recency'])
   plt.title('Many customers have not made any purchase this year.')
   plt.savefig('boxplot.png', dpi=300)
   plt.show()
  ```
<img src='https://github.com/user-attachments/assets/dbc13db3-0858-4601-9e0b-b109c1607f65' width=400>

✅ The boxplot shows that many customers have not made any purchase in 2012. They can be considered as already lost customers. Thus, I will remove them from the analysis.

- Frequency:
  
  <img src='https://github.com/user-attachments/assets/e4799685-de26-484d-a9e1-391c1e1994a3' width=400>

✅ The boxplot shows that there is a wide range of transaction numbers from 1-200. That's a good sign; however, there are many upper outliers. I will remove these outliers.

- Monetary: Similarly, "Monetary" variable also sees many outliers.
  
  <img src='https://github.com/user-attachments/assets/0d7620a7-9e95-4192-80a8-eac47d54e433' width=400>

Code to remove outliers:
   ```python
   # Remove customers who have not made any purchase in 2012
   df_no_outliers = df_rfm[df_rfm['Recency'] > 365]


   # Remove outliers based on "Frequency" criteria
   Q1 = df_rfm['Frequency'].quantile(0.25)
   Q3 = df_rfm['Frequency'].quantile(0.75)
   IQR = Q3 - Q1
   lower_bound = Q1 - 1.5 * IQR
   upper_bound = Q3 + 1.5 * IQR
   df_no_outliers = df_rfm[(df_rfm['Frequency'] >= lower_bound) & (df_rfm['Frequency'] <= upper_bound)]


   # Remove outliers based on "Monetary" criteria
   Q1 = df_rfm['Monetary'].quantile(0.25)
   Q3 = df_rfm['Monetary'].quantile(0.75)
   IQR = Q3 - Q1
   upper_bound = Q3 + 1.5 * IQR
   df_no_outliers = df_rfm[df_rfm['Monetary'] <= upper_bound]

   df_no_outliers.info()
   ```
   <img src='https://github.com/user-attachments/assets/99df3383-9b63-4f9b-8b53-99042a3a2c31' width=300>
   
   
✅ After removing outliers, the existing customer base has 3912 people. <br>

   Code to generate relative RFM:
   ```python
   df_no_outliers['FrequencyRate'] = pd.cut(df_no_outliers["Frequency"], 5, labels=['1','2','3','4','5'], duplicates='drop')
   df_no_outliers['RecencyRate'] = pd.cut(df_no_outliers['Recency'], 5, labels=['5','4','3','2','1'], duplicates='drop')
   df_no_outliers['MonetaryRate'] = pd.cut(df_no_outliers['Monetary'], 5, labels=['1','2','3','4','5'], duplicates='drop')
   df_no_outliers['Rate'] = df_no_outliers['FrequencyRate'].str.cat(df_no_outliers['RecencyRate']).str.cat(df_no_outliers['MonetaryRate'])
   df_rate = df_no_outliers.copy()
   df_rate.head()
   ```
   Output: 

   <img src='https://github.com/user-attachments/assets/ccf89dec-6cf5-4645-9c1d-b9b163b5f49b' width=500>


--------------------------------------------------------------------

#### ❓❓❓ Step 3: Customer segmentation

Use "Rate" variable for segmentation. Customers can be divided into 11 groups including champions, loyal, potential loyalist, new customers, promising, need attention, about to sleep, at risk, cannot lose them, hibernating customers, lost customers. 

   ```python
   conditions = [(df_rate['Rate'].isin(['555', '554', '544', '545', '454', '455', '445'])),
             (df_rate['Rate'].isin(['543', '444', '435', '355', '354', '345', '344', '335'])),
             (df_rate['Rate'].isin(['553', '551', '552', '541', '542', '533', '532', '531', '452', '451', '442', '441', '431', '453', '433', '432', '423', '353', '352', '351', '342', '341', '333', '323'])),
             (df_rate['Rate'].isin(['512', '511', '422', '421', '412', '411', '311'])),
              (df_rate['Rate'].isin(['525', '524', '523', '522', '521', '515', '514', '513', '425','424', '413','414','415', '315', '314', '313'])),
              (df_rate['Rate'].isin(['535', '534', '443', '434', '343', '334', '325', '324'])),
              (df_rate['Rate'].isin(['331', '321', '312', '221', '213', '231', '241', '251'])),
              (df_rate['Rate'].isin(['255', '254', '245', '244', '253', '252', '243', '242', '235', '234', '225', '224', '153', '152', '145', '143', '142', '135', '134', '133', '125', '124'])),
              (df_rate['Rate'].isin(['155', '154', '144', '214','215','115', '114', '113'])),
              (df_rate['Rate'].isin(['332', '322', '233', '232', '223', '222', '132', '123', '122', '212', '211'])),
              (df_rate['Rate'].isin(['111', '112', '121', '131','141','151']))
        ]

choices = ['Champions','Loyal','Potential Loyalist','New Customers','Promising','Need Attention',
           'About To Sleep','At Risk','Cannot Lose Them', 'Hibernating customers','Lost customers']

df_rate['Segment'] = np.select(conditions, choices)
df_rate.head()
   ```
Output: 

<img width="750" alt="image" src="https://github.com/user-attachments/assets/4b2b7f30-ca52-418c-8fde-5b9f13960f50" />


<br>

✅ **In summary, I just derive a dataframe including important information about each customer. Next, I will visualize the dataset to extract insights.**
   


## 3. Visualizations and Insights
This part is to visualize data in different dimensions in order to extract insights.

❓❓❓ **Number of customers by segment:**
  ```python
  # Visualize the number of customers by segment
   sns.set_style('whitegrid')
   visual1 = sns.countplot(data=df_rate, x='Segment')
   visual1.set_xticklabels(visual1.get_xticklabels(), rotation=60)
   visual1.bar_label(visual1.containers[0], fontsize=10, color='black')
   visual1.set_title("Figure 1: The number of customers by segment")
   plt.show()
  ```
   <img src='https://github.com/user-attachments/assets/0b1f4a86-442b-4906-b2b2-821079af41bc' width=400>
   
   Figure 1 shows that though there are 11 categories, SuperStore only has 6 customer groups: at risk, lost customers, cannot lose them, hibernating customers, potential loyalist, loyal.
   
  ✅ Nearly 60% customers are already lost. The company should not focus the marketing effort on this group.
  
  ✅ Around 30% customers are at risk. Their last purchase was a long time ago; however, they used to buy frequently from the company and their spendings were significant.
  
  ✅ Around 6% customers are can't lose. Their characteristics were somehow similar to that of 'at risk' customers.
  
  ✅ Hibernating group is relatively small. They made purchase not so long time before 31/12/2012. Their frequency and spending are at the average level.
  
  ✅ It is worthnoting that SuperStore has very few loyal and potential loyal customers. The company needs to figure out the reasons for this.

--------------------------------------------------------------------

❓❓❓ **The average spending by segment:**
  
  <img src='https://github.com/user-attachments/assets/abb27ceb-795a-4ed7-903e-1723bdd90579' width=400>
  
  ✅ Loyal customers have the highest average spending.
  
  ✅ 'Cannot lost them' group ranks the second in term of spending. 
  
  ✅ 'Potential loyal' and 'at risk' groups also spend a good amount of money.

--------------------------------------------------------------------

❓❓❓ **The average number of successful purchase by segment:**
  
  <img src='https://github.com/user-attachments/assets/ace5f9ae-ec56-41b9-b194-62ec1bea0243' width=400>


  ✅ Figure 3 illustrates that 'cannot lose them' group is really worth noting. They didn't make frequent purchases while the average spending was really high, around 2700.

--------------------------------------------------------------------

❓❓❓ **Recency by segment:**
  
  <img src='https://github.com/user-attachments/assets/1367b5f0-0305-4cd1-82fd-4dce3194f171' width=400>

   ✅ On average, 'loyal' and 'potential loyal' customers made the last transaction within the last 30 days.
   
   ✅ 'Cannot lose them' and 'at risk' customers made the last transaction within the last 2 months.

--------------------------------------------------------------------

❓❓❓ **Monetary vs. Frequency by segment:**
  
  <img src='https://github.com/user-attachments/assets/92009bfe-de41-4899-bbe2-3793f2acbda7' width=400>

   ✅ Figure 5 confirms that 'at risk' and 'cannot lose them' are important to SuperStore. They are a big group of customers. 'Cannot lose them' segment spent much more on average compared to 'at risk' group with the same frequency. 


## 4. Recommendations

The first goal of marketing campaign is to appreciate customers who stay with SuperStore in 2012. So:

      📍 Ignore the 'lost' group since they have not made purchase this year. 

      📍 The appreciation program should be directed to 'loyal', 'potential loyal', 'at risk' and 'cannot lose them' segments. 


The second goal is to convert potential customers in loyalists. So:

      📍 Focus on keeping 'at risk' and 'cannot lose them' customers. The reason is that these two groups have good average spendings, while the last purchase was around 70 days from 31/12/2012.

      📍 To convert them into 'loyal' group, the recency should be cut shorter to around 1 month. The suggested action is to encourage them make purchases again by offering good deals. Further analysis can be conducted to find out the most frequently bought products by these two groups. So SuperStore can tailor a more suitable sales program for Christmas and New Year holidays.
