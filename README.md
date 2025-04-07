# [Python] Customer Segmentation Project

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
```

```sql
#Load dataset
df = pd.read_excel(io.BytesIO(uploaded['ecommerce retail.xlsx']))
df
```
![Image](https://github.com/user-attachments/assets/7f8050c3-d7b2-426a-8070-a2ee82e0eafb)

```sql
#Load Segmentation sheet
Segmentation = pd.read_excel(io.BytesIO(uploaded['ecommerce retail.xlsx']), sheet_name='Segmentation')
```
```sql
#Column information
df.info()
```
<img width="1100" alt="Image" src="https://github.com/user-attachments/assets/70d24fba-6543-4f41-bbaa-fede90173d50" />

```sql
#Statistics summary of the dataset
df.describe()
```
<img width="1100" alt="Image" src="https://github.com/user-attachments/assets/ef39622f-cdcd-4d1a-b0fb-dadc00484443" />

```sql
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
```
<img width="1232" alt="Image" src="https://github.com/user-attachments/assets/64c834c2-7250-4449-8cf7-031f334fdef8" />

```sql
#Create a new table grouped by CustomerID
grouped_df2 = df2.groupby('CustomerID').agg(
    RecentInvoiceDate=('InvoiceDate', 'max'),
    Frequency=('CustomerID', 'count'),
    MonetaryValue=('Total Price', 'sum')
)

#Calculate the number of days since the last purchase per CustomerID
grouped_df2['Recency'] = (df2['InvoiceDate'].max() - grouped_df2['RecentInvoiceDate']).dt.days

grouped_df2
```
<img width="1232" alt="Image" src="https://github.com/user-attachments/assets/0099adef-ba66-4104-a312-8b4e2c37409d" />

```sql
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
```
<img width="1391" alt="Image" src="https://github.com/user-attachments/assets/cbe4c190-c4cd-4274-8549-5caef563af6c" />

```sql
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
```
<img width="875" alt="Image" src="https://github.com/user-attachments/assets/585e38fe-8c77-4c22-9f6f-b59a77965e2d" />

```sql
#Create a table summarizing RFM_score frequency
rfm_frequency = grouped_df2['RFM_score'].value_counts().reset_index()
rfm_frequency.columns = ['RFM_score', 'Frequency of RFM_score']
rfm_frequency
```
<img width="1089" alt="Image" src="https://github.com/user-attachments/assets/a8faf8de-4a10-434d-a71e-b2473c656622" />

```sql
#Split RFM Score column into individual values
Segmentation['RFM Score'] = Segmentation['RFM Score'].str.split(', ')
Segmentation = Segmentation.explode('RFM Score')
Segmentation
```
<img width="1089" alt="Image" src="https://github.com/user-attachments/assets/040c9bc4-d6d4-490a-b341-2559c9f946e6" />

```sql
#Merge grouped_df2 and Segmentation tables
merged_df = pd.merge(grouped_df2, Segmentation, left_on='RFM_score', right_on='RFM Score', how='left')
merged_df
```
<img width="1412" alt="Image" src="https://github.com/user-attachments/assets/28e3fded-b4e9-47db-bf8c-f3adbe069076" />

```sql
import seaborn as sns

#Plot distribution of customer segments
sns.countplot(x='Segment', data=merged_df)
plt.xticks(rotation=90)
plt.ylabel('Distribution')
plt.title('Distribution of 11 customer groups')
plt.show()
```
<img width="1216" alt="Image" src="https://github.com/user-attachments/assets/74a67a95-2b46-457a-928b-d0ca28cf7715" />

```sql
#Plot distribution of Recency
sns.distplot(merged_df['Recency'])
plt.xlabel('Recency (days)')
plt.ylabel('Distribution')
plt.title('Distribution of Recency')
plt.show()
```
<img width="1216" alt="Image" src="https://github.com/user-attachments/assets/9e376f6f-08b3-4897-a93a-63905dfe9138" />

```sql
#Plot distribution of Frequency
sns.distplot(merged_df['Frequency'])
plt.xlabel('Frequency (times)')
plt.ylabel('Distribution')
plt.title('Distribution of Frequency')
plt.show()
```
<img width="1216" alt="Image" src="https://github.com/user-attachments/assets/5e6449f3-9ba7-44cd-8e75-ddc241d19c9f" />

```sql
#Plot distribution of Monetary Value
sns.distplot(merged_df['MonetaryValue'])
plt.xlabel('Monetary Value ($)')
plt.ylabel('Distribution')
plt.title('Distribution of Monetary Value')
plt.show()
```
<img width="1216" alt="Image" src="https://github.com/user-attachments/assets/15319ee4-4584-4775-9758-d4706e247269" />

## IV. Insights
- The three customer segments with the highest proportion are "New Customers", "At Risk", and "Hibernating Customers".

- For the "New Customers" segment, we can implement strategies to encourage repeat purchases, such as "running ads" and "offering discounts for the next order".

- A common characteristic of the remaining two segments is that they do not return to make another purchase. This could be due to product/service quality issues or other factors causing customer dissatisfaction. To address this, we can conduct surveys to identify problems and implement solutions to extend the customer lifetime value.

- Additionally, we can focus on specific strategies for each customer segment, as outlined in the table in the **Recommendations** section.
  
## V. Recommendations
| Segments                | Characteristics                                                                 | Recommendations                                                                                     |
|-------------------------|-------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------|
| **Champions**           | - New transactions <br> - Frequent buyers <br> - Highest spenders <br> - Very loyal <br> - Generous spending <br> - Likely to buy again soon | - Take care and encourage them to return more: <br> + Offer loyalty programs with personalized benefits <br> + Recommend higher-value items and product combos based on order history |
| **Loyal**               | - Very frequent buyers <br> - Medium to high spending                         | - Implement membership ranking strategies <br> - Introduce new products                         |
| **Potential Loyalist**  | - New transactions <br> - Multiple purchases but not frequent <br> - Medium to high spending | - Upsell high-value products <br> - Gather feedback and implement engagement strategies         |
| **New Customers**       | - Recently made their first purchase <br> - Infrequent buyer <br> - Low cart value | - Run ads <br> - Offer discounts for next purchase                                             |
| **Promising**           | - Purchased recently <br> - Infrequent buyer <br> - Large purchase power      | - Run ads <br> - Implement membership programs <br> - Offer exclusive perks <br> - Recommend higher-value items and product combos |
| **Need Attention**      | - Haven’t purchased for a while <br> - Moderate purchase frequency <br> - Low to moderate cart value | - Create limited-time promotions <br> - Suggest products based on past purchases               |
| **About To Sleep**      | - Long time since last purchase <br> - Low purchase frequency <br> - High cart value | - Personalized interactions via email and phone <br> - Offer vouchers, discounts, and exclusive perks |
| **At Risk**             | - Haven’t returned for a long time <br> - Used to be a frequent buyer <br> - Medium to high cart value | - Conduct surveys to identify the root cause and implement solutions                          |
| **Cannot Lose Them**    | - Haven’t returned for a long time <br> - Used to be a frequent buyer <br> - High cart value | - Offer exclusive privileges such as early access to new products and special discounts        |
| **Hibernating Customers** | - Very long time since last purchase <br> - Low purchase frequency <br> - Low cart value | - Personalized interactions via email and phone <br> - Offer vouchers, discounts, and exclusive perks |
| **Lost Customers**      | - Very long time since last purchase <br> - Very low purchase frequency <br> - Behavior: Exploratory purchases, one-time buyers who compare products/services | - Implement policies for free product trials                                                  |

