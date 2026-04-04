# RFM Customer Segmentation for Holiday Campaigns in Global Retail | Python
  
Author: [LUU MY HOA]  
Date: April 2026  
Tools Used: Python  

## 📑 Table of Contents  
1. [Background & Overview](#1-background--overview) 
2. [Dataset Description & Data Structure](#2-dataset-description--data-structure)
3. [Data Cleaning & Preprocessing](#3-data-cleaning--preprocessing)
4. [Apply RFM Model](#4-apply-rfm-model)
5. [Visualization & Analysis](#5-visualization--analysis)
6. [Final Conclusion & Recommendations](#6-final-conclusion--recommendations)

## 1. Background & Overview  

This project focuses on customer segmentation for a global company (SuperStore) to support customer appreciation during the Christmas and New Year period. In addition, the segmentation is designed not only for this campaign but also to support long-term customer management strategies.

### 📌 Business Context:
- SuperStore is a UK-based online retail company
- The company mainly sells unique all-occasion gifts, with many customers being wholesalers
- During the Christmas and New Year season, the Marketing team plans to run a “Thank You” campaign to appreciate customers who have supported the company over time
- The Marketing team requested support from the Data Analytics team to segment customers
- SuperStore already has an RFM-based segmentation mapping for customer classification

### ❓ Main Business Questions:
- How are customers distributed across different RFM segments?
- Which groups should be selected for the holiday “Thank You” campaign?
- Which customer segments show potential to become loyal customers in the future?
- How does customer behavior across segments differ between UK and international markets?

### 👤 Who is this project for?
- Marketing Teams: Design targeted campaigns for different customer groups
- Data & Business Analysts: Learn how to build an automated segmentation using Python
- Decision-makers: Understand customer behavior and support long-term strategy

### 📊 Why RFM?
RFM (Recency, Frequency, Monetary) is a simple and effective method to segment customers based on their purchasing behavior:
- Recency (R): How recently a customer made a purchase
- Frequency (F): How often a customer makes purchases
- Monetary (M): How much a customer spends

## 2. Dataset Description & Data Structure  

### 📂 Data Source  
- Source: Online Retail dataset (SuperStore)
- Format: .xlsx
- Content: 2 sheets (transactional data, segmentation)
- Size:  541,910 rows × 8 columns (sheet 1), 12 rows x 2 columns (sheet 2)

### 📌 Data Structure 
Sheet 1 : The dataset consists of transactional records, where each row represents a product a product within an invoice.

<details> <summary> The table schema </summary>
  
| Column Name | Description |
| :--------- | :------------|
| InvoiceNo | Invoice number. If it starts with “C”, it indicates a cancellation |
| StockCode | Product code |
| Description | Product name |
| Quantity | Number of items purchased per transaction |
| InvoiceDate | Date and time when the transaction was created |
| UnitPrice | Price per unit (in GBP) |
| CustomerID| Customer identifier |
| Country | Country where the customer is located |
</details>
  
Sheet 2: Segmentation mapping defines business rules for mapping RFM scores to customer segments. Each RFM score is a 3-digit combination representing a combination of Recency (R), Frequency (F), and Monetary (M), where each dimension is scored from 1 to 5.
<details> <summary>Full table</summary>
  
| Segment   | RFM Score      |
| :--- | :--- |
| Champions            | 555, 554, 544, 545, 454, 455, 445 |
| Loyal                 | 543, 444, 435, 355, 354, 345, 344, 335 |
| Potential Loyalist    | 553, 551, 552, 541, 542, 533, 532, 531, 452, 451, 442, 441, 431, 453, 433, 432, 423, 353, 352, 351, 342, 341, 333, 323 |
| New Customers        | 512, 511, 422, 421, 412, 411, 311   |
| Promising             | 525, 524, 523, 522, 521, 515, 514, 513, 425, 424, 413, 414, 415, 315, 314, 313 |
| Need Attention       | 535, 534, 443, 434, 343, 334, 325, 324 |
| About To Sleep       | 331, 321, 312, 221, 213, 231, 241, 251 |
| At Risk            | 255, 254, 245, 244, 253, 252, 243, 242, 235, 234, 225, 224, 153, 152, 145, 143, 142, 135, 134, 133, 125, 124           |
| Cannot Lose Them  | 155, 154, 144, 214, 215, 115, 114, 113       |
| Hibernating Customers| 332, 322, 233, 232, 223, 222, 132, 123, 122, 212, 211   |
| Lost Customers| 111, 112, 121, 131, 141, 151  |

</details>

### 🔗 Relationship Between Tables  

- RFM metrics are calculated from transactional data (Sheet 1)  
- The resulting RFM scores are mapped to segments (Sheet 2)  
- This mapping enables customer classification for targeted marketing strategies

## 3. Data Cleaning & Preprocessing

| Problem | Data Cleaning & Preprocessing |
|:--------|:-----------------------------|
| Duplicate records | Remove exact duplicate rows to avoid double-counting |
| InvoiceNo starts with 'C' | Remove canceled transactions (not actual purchases) |
| Quantity <= 0 | Remove as these likely represent returns or invalid entries |
| UnitPrice <= 0 | Remove to ensure valid revenue calculation |
| Missing CustomerID | Remove, as RFM analysis requires customer-level identification |
| CustomerID data type | Convert to integer for consistency |
| Missing revenue metric | Create Revenue = Quantity × UnitPrice |
| Undefined date range | Identify min/max InvoiceDate to define analysis period |

<details> <summary>Code Python</summary>
  
```python
  
df_clean = df.copy()

# Remove duplicates
df_clean = df_clean.drop_duplicates()

# Remove canceled invoices
df_clean = df_clean[~df_clean['InvoiceNo'].astype(str).str.startswith('C')]

# Filter valid transactions
df_clean = df_clean[(df_clean['Quantity'] > 0) & (df_clean['UnitPrice'] > 0)]

# Remove missing customers
df_clean = df_clean[df_clean['CustomerID'].notna()]

# Convert data types
df_clean['CustomerID'] = df_clean['CustomerID'].astype(int)

# Create revenue
df_clean['Revenue'] = df_clean['Quantity'] * df_clean['UnitPrice']

print(f" Raw data: {df.shape[0]:,} rows × {df.shape[1]} columns")

print(f" After cleaning: {df_clean.shape[0]:,} rows ({df_clean.shape[0]/df.shape[0]*100:.1f}% retained)")

print(f" Unique customers: {df_clean['CustomerID'].nunique():,}")

print(f" Date range: {df_clean['InvoiceDate'].min().date()} → {df_clean['InvoiceDate'].max().date()}")
```

</details>

### 📊 Data Summary

Raw data: 541,909 rows × 8 columns

After cleaning: 392,692 rows (72.5% retained)

Unique customers: 4,338
   
Date range: 2010-12-01 → 2011-12-09
   
## 4. Apply RFM Model
- REFERENCE_DATE is automatically calculated. REFERENCE_DATE is set to 1 day after the latest transaction date to ensure Recency is always ≥ 1
- Recency is calculated as the number of days since the customer's last purchase relative to the REFERENCE_DATE. A smaller Recency value indicates a more recent and active customer
- Frequency is defined as the number of unique invoices (transactions) per customer
- Monetary represents total revenue per customer (in GBP)

<details> <summary>Code Python</summary>
  
```python
# Identify the most recent date in your data
latest_date = df_clean['InvoiceDate'].max()

# Set reference date (1 day after the latest transaction)
REFERENCE_DATE = latest_date + dt.timedelta(days=1)

# Cal R, F, M
rfm = df_clean.groupby('CustomerID').agg(
    Recency   = ('InvoiceDate', lambda x: (REFERENCE_DATE - x.max()).days),
    Frequency = ('InvoiceNo',   'nunique'),
    Monetary  = ('Revenue',     'sum')
).reset_index()

print(f"\n📊 RFM Summary:")
print(rfm[['Recency','Frequency','Monetary']].describe().round(2).to_string())
```
</details>

### 📊 RFM Summary

|  | Recency | Frequency | Monetary |
|---|---|---|---|
| count | 4,338.00 | 4,338.00 | 4,338.00 |
| mean | 92.54 | 4.27 | 2,048.69 |
| std | 100.01 | 7.70 | 8,985.23 |
| min | 1.00 | 1.00 | 3.75 |
| 25% | 18.00 | 1.00 | 306.48 |
| 50% | 51.00 | 2.00 | 668.57 |
| 75% | 142.00 | 5.00 | 1,660.60 |
| max | 374.00 | 209.00 | 280,206.02 |

*The summary statistics provide an overview of customer behavior distribution before segmentation.*

### RFM SCORING — Quintile (scale 1–5) 
- Each metric is divided into 5 equal-sized groups using quintiles (qcut). This ensures a balanced distribution of customers across score levels
- Recency is reversed (lower is better), while Frequency and Monetary are positively scored  
- Ranking is applied before qcut to handle duplicate values and ensure stable binning

<details> <summary>Code Python</summary>
  
```python 
# Recency: smaller = more recent = BETTER → reverse labels
rfm['R_Score'] = pd.qcut(rfm['Recency'].rank(method='first'), 5, labels=[5,4,3,2,1]).astype(int)

# Frequency & Monetary: larger = better → ascending labels
rfm['F_Score'] = pd.qcut(rfm['Frequency'].rank(method='first'), 5, labels=[1,2,3,4,5]).astype(int)
rfm['M_Score'] = pd.qcut(rfm['Monetary'].rank(method='first'), 5, labels=[1,2,3,4,5]).astype(int)

rfm['RFM_Score'] = rfm['R_Score'].astype(str) + rfm['F_Score'].astype(str) + rfm['M_Score'].astype(str)
```
</details>

### SEGMENT DICTIONARY
- The SEGMENT DICTIONARY defines customer segments based on the business rules provided in Sheet 2 of the dataset.

<details> <summary>Code Python</summary>
  
```python 
segment_dict = {
    'Champions': {'555','554','544','545','454','455','445'},
    'Loyal': {'543','444','435','355','354','345','344','335'},
    'Potential Loyalist': {'553','551','552','541','542','533','532','531','452','451','442',
                           '441','431','453','433','432','423','353','352','351','342','341','333','323'},
    'New Customers': {'512','511','422','421','412','411','311'},
    'Promising': {'525','524','523','522','521','515','514','513','425','424','413','414','415','315','314','313'},
    'Need Attention': {'535','534','443','434','343','334','325','324'},
    'About To Sleep': {'331','321','312','221','213','231','241','251'},
    'At Risk': {'255','254','245','244','253','252','243','242','235','234','225','224','153',
                '152','145','143','142','135','134','133','125','124'},
    'Cannot Lose Them': {'155','154','144','214','215','115','114','113'},
    'Hibernating Customers': {'332','322','233','232','223','222','132','123','122','212','211'},
    'Lost Customers': {'111','112','121','131','141','151'}
}
```
</details>

### MAPPING RFM SCORE AND SEGMENT
<details> <summary>Code Python</summary>
  
```python 
# Hàm gán segment dựa theo bảng
def assign_segment(rfm_code):
    for segment, codes in segment_dict.items():
        if rfm_code in codes:
            return segment
    return 'Others'

# Áp dụng vào dataframe
rfm['Segment'] = rfm['RFM_Score'].apply(assign_segment)

seg_stats = (
    rfm.groupby('Segment')
    .agg(
        Count       = ('CustomerID', 'count'),
        Avg_Recency = ('Recency',    'mean'),
        Avg_Freq    = ('Frequency',  'mean'),
        Avg_Mon     = ('Monetary',   'mean'),
        Total_Rev   = ('Monetary',   'sum'),
    )
    .reset_index()
    .sort_values('Count', ascending=False)
    .reset_index(drop=True)
)

print(f"\n[7] Segment distribution:")
print(seg_stats[['Segment','Count','Avg_Recency','Avg_Freq','Avg_Mon','Total_Rev']].round(2).to_string(index=False))
```
</details>

### 📊 Segment Distribution Summary

| Segment | Count | Avg_Recency | Avg_Freq | Avg_Mon | Total_Rev |
|---|---|---|---|---|---|
| Champions | 831 | 11.28 | 12.14 | 6,716.97 | 5,581,802.32 |
| Hibernating Customers | 697 | 148.82 | 1.56 | 409.71 | 285,570.76 |
| Lost Customers | 489 | 274.80 | 1.07 | 199.37 | 97,491.54 |
| Loyal | 427 | 36.35 | 5.34 | 2,389.32 | 1,020,240.32 |
| At Risk | 425 | 143.17 | 3.76 | 1,776.45 | 754,990.25 |
| Potential Loyalist | 405 | 26.30 | 2.52 | 538.55 | 218,114.26 |
| Need Attention | 286 | 33.29 | 3.12 | 1,630.18 | 466,230.62 |
| About To Sleep | 285 | 86.06 | 1.28 | 275.95 | 78,644.84 |
| New Customers | 267 | 28.22 | 1.07 | 222.28 | 59,347.96 |
| Promising | 133 | 25.08 | 1.33 | 895.49 | 119,099.52 |
| Cannot Lose Them | 93 | 236.61 | 2.34 | 2,211.58 | 205,676.50 |

*This table summarizes customer distribution and average RFM metrics across segments.*

## 5. Visualization & Analysis

## 6. Final Conclusion & Recommendations  

👉🏻 Based on the insights and findings above, we would recommend the to consider the following:  

📍 Key Takeaways:  
✔️ Recommendation 1  
✔️ Recommendation 2  
✔️ Recommendation 3

