# Python-RFM-segmentation-project
## I.Introduction
The project leverages Python to clean, group and visualize data to divide customers into different groups based on RFM model (recency, frequency and monetary value per transaction) in order to implement the right strategies to each group.

## II. Context & Dataset
SuperStore is a global retail company with a vast customer base.

On the occasion of Christmas and New Year, the Marketing department wants to launch marketing campaigns to appreciate customers who have supported the company throughout the year, as well as to identify potential customers who could become loyal customers.

However, the Marketing department has not yet been able to segment customers for this year because the dataset is too large to be manually processed as in previous years. Therefore, they have requested the Data Analytics department to support the implementation of a customer segmentation model to tailor marketing programs for different customer groups.

The Marketing Director has suggested using the RFM model. Previously, when the company was smaller, the team could calculate and classify customers using Excel. However, given the large scale of data now, they would like the Data team to develop a segmentation evaluation workflow using Python programming.

**About the dataset:**
| Column Name  | Description |
|-------------|------------|
| **InvoiceNo**  | Invoice number. A 6-digit number uniquely assigned to each transaction. If it starts with 'C', it indicates a cancellation. |
| **StockCode**  | Product (item) code. A 5-digit number uniquely assigned to each product. |
| **Description**  | Product name. |
| **Quantity**  | The number of each product per transaction. Numeric. |
| **InvoiceDate**  | Date and time of the transaction. Numeric. |
| **UnitPrice**  | Unit price in sterling. Numeric. |
| **CustomerID**  | Customer number, a 5-digit number uniquely assigned to each customer. |
| **Country**  | Name of the country where the customer resides. |

## III. Data preparation
```sql
#Import files
from google.colab import files

#Upload file
uploaded = files.upload()

#Import libraries
import pandas as pd
import io

#Load dataset
df = pd.read_excel(io.BytesIO(uploaded['ecommerce retail.xlsx']))
df

#Load Segmentation sheet
Segmentation = pd.read_excel(io.BytesIO(uploaded['ecommerce retail.xlsx']), sheet_name='Segmentation')

#Column information
df.info()

#Summary statistics of the dataset
df.describe()

#Remove NaN values in the CustomerID column
df = df.dropna(subset=['CustomerID'])

#Convert CustomerID data type to integer
df['CustomerID'] = df['CustomerID'].astype(int)

#Filter data
df1 = df[(df['Quantity'] > 0) & (df['UnitPrice'] > 0) & (df['Country'] == 'United Kingdom')]

#Convert InvoiceNo column to string type
df1['InvoiceNo'] = df1['InvoiceNo'].astype(str)

#Filter out rows where InvoiceNo starts with 'C'
df2 = df1[~df1['InvoiceNo'].str.startswith('C')]

#Calculate total price for each transaction
df2['Total Price'] = df2['UnitPrice'] * df2['Quantity']

df2

#Create a new table grouped by CustomerID
grouped_df2 = df2.groupby('CustomerID').agg(
    RecentInvoiceDate=('InvoiceDate', 'max'),
    Frequency=('CustomerID', 'count'),
    MonetaryValue=('Total Price', 'sum')
)

#Calculate the number of days since the last purchase per CustomerID
grouped_df2['Recency'] = (df2['InvoiceDate'].max() - grouped_df2['RecentInvoiceDate']).dt.days

grouped_df2

#Scoring
grouped_df2['recency_qcut'] = pd.qcut(grouped_df2['Recency'], q=5, labels=False)
grouped_df2['recency_rate'] = grouped_df2['recency_qcut'] + 1

grouped_df2['frequency_qcut'] = pd.qcut(grouped_df2['Frequency'], q=5, labels=False)
grouped_df2['frequency_rate'] = grouped_df2['frequency_qcut'] + 1

grouped_df2['monetary_qcut'] = pd.qcut(grouped_df2['MonetaryValue'], q=5, labels=False)
grouped_df2['monetary_rate'] = grouped_df2['monetary_qcut'] + 1

#Combine the three scored metrics
grouped_df2['RFM_score'] = grouped_df2['recency_rate'].astype(str) + grouped_df2['frequency_rate'].astype(str) + grouped_df2['monetary_rate'].astype(str)

grouped_df2

import matplotlib.pyplot as plt

#Count occurrences of each RFM_score value
rfm_counts = grouped_df2['RFM_score'].value_counts()

#Visualize RFM_score frequency using a bar chart
plt.figure(figsize=(10, 6))
plt.bar(rfm_counts.index, rfm_counts.values, color='skyblue', edgecolor='black')
plt.xlabel('RFM Score')
plt.ylabel('Frequency')
plt.title('Frequency of RFM Score')
plt.grid(True)
plt.xticks(rotation=90, fontsize=5, fontweight='bold')
plt.show()

#Create a table summarizing RFM_score frequency
rfm_frequency = grouped_df2['RFM_score'].value_counts().reset_index()
rfm_frequency.columns = ['RFM_score', 'Frequency of RFM_score']
rfm_frequency

Segmentation

#Split RFM Score column into individual values
Segmentation['RFM Score'] = Segmentation['RFM Score'].str.split(', ')
Segmentation = Segmentation.explode('RFM Score')

Segmentation

Segmentation['RFM Score'] = Segmentation['RFM Score'].str.split(',')
Segmentation = Segmentation.explode('RFM Score')

Segmentation

#Merge grouped_df2 and Segmentation tables
merged_df = pd.merge(grouped_df2, Segmentation, left_on='RFM_score', right_on='RFM Score', how='left')

merged_df

import seaborn as sns

#Plot distribution of customer segments
sns.countplot(x='Segment', data=merged_df)
plt.xticks(rotation=90)
plt.ylabel('Distribution')
plt.title('Distribution of 11 customer groups')
plt.show()

#Plot distribution of Recency
sns.distplot(merged_df['Recency'])
plt.xlabel('Recency (days)')
plt.ylabel('Distribution')
plt.title('Distribution of Recency')
plt.show()

#Plot distribution of Frequency
sns.distplot(merged_df['Frequency'])
plt.xlabel('Frequency (times)')
plt.ylabel('Distribution')
plt.title('Distribution of Frequency')
plt.show()

#Plot distribution of Monetary Value
sns.distplot(merged_df['MonetaryValue'])
plt.xlabel('Monetary Value ($)')
plt.ylabel('Distribution')
plt.title('Distribution of Monetary Value')
plt.show()

## IV. Insights
## V. Recommendation
