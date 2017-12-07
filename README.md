## **Heroes of Pymoli Data Analysis**

#### OBSERVED TRENDS
   ##### 1) Over 80% of all players are males.  Males made the majority of purchases and contributed to over 80% of the total revenue.
   ##### 2) Regardless of gender, 20-24 year olds have made the most number of purchases and spent the most on this game. 
   ##### 3) The most profitable items all had high item prices.  Despite this, multiple purchases were made for these items.  
```python
# Obtain dependencies
import pandas as pd
import numpy as np

# Read CSV file
pymoli = pd.read_json('purchase_data.json')
```
### **Player Count**
```python
# Get a unique number of players by 'SN' from the file
unique_users = pymoli['SN'].nunique()
total_players = pd.DataFrame({'Total Players': [unique_users]})
total_players
```

### Purchasing Analysis (Total)
```python
# From the file, calculate metrics for Purchasing Analysis
unique_items = pymoli['Item Name'].nunique()
avg_price = pymoli['Price'].mean()
num_purchases = pymoli['Item Name'].count()
total_revenue = pymoli['Price'].sum()

# Create dataframe with metrics
purchasing_analysis_summary = pd.DataFrame(
        {'Number of Unique Items': [unique_items],
         'Average Price': [avg_price],
         'Number of Purchases': [num_purchases],
         'Total Revenue': [total_revenue],
        } )

# format columns
purchasing_analysis_summary['Average Price'] = purchasing_analysis_summary['Average Price'].map("$ {:,.2f}".format)
purchasing_analysis_summary['Total Revenue'] = purchasing_analysis_summary['Total Revenue'].map("$ {:,.2f}".format)


purchasing_analysis_summary[['Number of Unique Items','Average Price', 'Number of Purchases', 'Total Revenue']]
```
### Gender Demographics
```python
# Group by Gender and get Percentage of Players based on Gender Totals
gender_groupby = pymoli.groupby('Gender')['SN'].nunique().reset_index()
gender_groupby['Percentage of Players'] = 100 * gender_groupby['SN']/gender_groupby['SN'].sum()
gender_summary = gender_groupby[['Gender', 'Percentage of Players','SN' ]].sort_values(['Percentage of Players'],ascending = False)
gender_summary = gender_summary.reset_index(drop=True)

# format columns
gender_summary['Percentage of Players'] = gender_summary['Percentage of Players'].map("{:,.2f}".format)

# set Index to Gender
gender_demos_summary = gender_summary.set_index('Gender')
gender_demos_summary = gender_demos_summary.rename(columns = {'SN': 'Total Count'}) 
gender_demos_summary
```
### Purchasing Analysis (Gender)
```python
# Group by Gender and get summary of data
gender_demos = pymoli.groupby('Gender')["Price"].agg(['sum', 'mean','count']).sort_values(['sum'],ascending = False).reset_index()

# Calculate normalized values
gender_normal_f= gender_demos[['sum']].iloc[1,0]/gender_summary[['SN']].iloc[1,0]
gender_normal_m= gender_demos[['sum']].iloc[0,0]/gender_summary[['SN']].iloc[0,0]
gender_normal_o = gender_demos[['sum']].iloc[2,0]/gender_summary[['SN']].iloc[2,0]
# Add Normalized Totals column to summary table
gender_summary['Normalized Totals'] = [gender_normal_m, gender_normal_f, gender_normal_o]

# Merge Table with gender summary information and gender demographic information.  Rename Columns
merge_gender = pd.merge(gender_summary, gender_demos, on= 'Gender')
merge_gender_df = merge_gender.rename(columns = {'count': 'Purchase Count', 
                                               'mean' : 'Average Purchase Price', 
                                               'sum' : 'Total Purchase Value' })

# format columns
merge_gender_df['Average Purchase Price'] = merge_gender_df['Average Purchase Price'].map("$ {:,.2f}".format)
merge_gender_df['Total Purchase Value'] = merge_gender_df['Total Purchase Value'].map("$ {:,.2f}".format)
merge_gender_df['Normalized Totals'] = merge_gender_df['Normalized Totals'].map("$ {:,.2f}".format)

# set Index to Gender
purchasing_analysis_gender = merge_gender_df.set_index('Gender')
purchasing_analysis_gender[['Purchase Count', 'Average Purchase Price', 'Total Purchase Value', 'Normalized Totals']]
```
### Age Demographics
```python
#Create the bins for Age Buckets
age_bins = [0, 9, 14, 19, 24, 29, 34, 39, 44]

# Create the labels for bins
age_groups = ['<10', '10-14', '15-19', '20-24', '25-29', '30-34', '35-39', '40+' ]

# Add column of bins based on Age
pymoli["Age Groups"] = pd.cut(pymoli["Age"],age_bins, labels=age_groups)

#Calculate total count and percentage of those counts by age groups
age_groupby = pymoli.groupby('Age Groups')['SN'].nunique().reset_index()
age_groupby['Percentage of Players'] = 100 * age_groupby['SN']/gender_groupby['SN'].sum()
age_summary = age_groupby[['Age Groups', 'Percentage of Players','SN' ]].sort_values(['Age Groups'])
age_summary = age_summary.reset_index(drop=True)

# format columns
age_summary['Percentage of Players'] = age_summary['Percentage of Players'].map("{:,.2f}".format)

# set Indext to Age Groups
age_demos_summary = age_summary.set_index('Age Groups')
age_demos_summary = age_demos_summary.rename(columns = {'SN': 'Total Count'}) 
age_demos_summary
```
### Purchasing Analysis (Age)

```python
# Group by Age Groups and get summary of data
age_demos = pymoli.groupby('Age Groups')['Price'].agg(['sum', 'mean','count']).reset_index()
age_demos

# Calculate normalized value for each Age Group
age_normal_0 = age_demos[['sum']].iloc[0,0]/age_summary[['SN']].iloc[0,0]
age_normal_1 = age_demos[['sum']].iloc[1,0]/age_summary[['SN']].iloc[1,0]
age_normal_2 = age_demos[['sum']].iloc[2,0]/age_summary[['SN']].iloc[2,0]
age_normal_3 = age_demos[['sum']].iloc[3,0]/age_summary[['SN']].iloc[3,0]
age_normal_4 = age_demos[['sum']].iloc[4,0]/age_summary[['SN']].iloc[4,0]
age_normal_5 = age_demos[['sum']].iloc[5,0]/age_summary[['SN']].iloc[5,0]
age_normal_6 = age_demos[['sum']].iloc[6,0]/age_summary[['SN']].iloc[6,0]
age_normal_7 = age_demos[['sum']].iloc[7,0]/age_summary[['SN']].iloc[7,0]

# Add column with Normalized Totals
age_summary['Normalized Totals'] = [age_normal_0, age_normal_1, age_normal_2, age_normal_3, age_normal_4, age_normal_5,
                                    age_normal_6, age_normal_7]

# format column
age_summary['Normalized Totals'] = age_summary['Normalized Totals'].map("$ {:,.2f}".format)

# Merge Table of age summary info with age demographic info and rename columns
merge_age = pd.merge(age_summary, age_demos, on= 'Age Groups')
merge_age_df = merge_age.rename(columns = {'count': 'Purchase Count', 
                                               'mean' : 'Average Purchase Price', 
                                               'sum' : 'Total Purchase Value' })

# format columns
merge_age_df['Average Purchase Price'] = merge_age_df['Average Purchase Price'].map("$ {:,.2f}".format)
merge_age_df['Total Purchase Value'] = merge_age_df['Total Purchase Value'].map("$ {:,.2f}".format)

# set Index to Age Groups
purchasing_analysis_age = merge_age_df.set_index('Age Groups')
purchasing_analysis_age[['Purchase Count', 'Average Purchase Price', 'Total Purchase Value', 'Normalized Totals']]
```
### Top Spenders
```python
# get list of top 5 spenders
top5_spenders = pymoli.groupby('SN')['Price'].sum().sort_values(ascending = False).nlargest(5).reset_index()
top5_spenders_SN = top5_spenders[['SN']]

# merge top 5 spenders with original data set to obtain summary data 
merge_top5_spenders = pd.merge(top5_spenders_SN, pymoli, on = 'SN', how = 'left')
merge_top5_spenders = merge_top5_spenders.groupby('SN')['Price'].agg(['sum', 'mean','count']).sort_values(['sum'], ascending= False)

# rename columns
top_spenders = merge_top5_spenders.rename(columns = {'count': 'Purchase Count', 'mean' : 'Average Purchase Price', 
                                                      'sum' : 'Total Purchase Value' })
# format columns
top_spenders['Average Purchase Price'] = top_spenders['Average Purchase Price'].map("$ {:,.2f}".format)
top_spenders['Total Purchase Value'] = top_spenders['Total Purchase Value'].map("$ {:,.2f}".format)

top_spenders[['Purchase Count', 'Average Purchase Price', 'Total Purchase Value']]
```
### Most Popular Items
```python
# get list of top 5 most popular items by count
items_bycount = pymoli.groupby(['Item ID','Item Name'])['Price'].agg(['count','sum','mean']).sort_values(['count'], ascending= False).reset_index()
top5_items = items_bycount[:5]

# rename columns
top_items = top5_items.rename(columns = {'count': 'Purchase Count', 'mean' : 'Item Price', 
                                                    'sum' : 'Total Purchase Value' })
# format columns
top_items['Item Price'] = top_items['Item Price'].map("$ {:,.2f}".format)
top_items['Total Purchase Value'] = top_items['Total Purchase Value'].map("$ {:,.2f}".format)

# set Index to Item ID and Item Name
top_items_df = top_items.set_index(['Item ID', 'Item Name'])
top_items_df[['Purchase Count', 'Item Price', 'Total Purchase Value']]
```
### Most Profitable Items

```python
# get list of top 5 most profitable items by sum
items_bysum = pymoli.groupby(['Item ID','Item Name'])['Price'].agg(['count','sum','mean']).sort_values(['sum'], ascending= False).reset_index()
top5sum_items = items_bysum[:5]

# rename columns
profitable_items = top5sum_items.rename(columns = {'count': 'Purchase Count', 'mean' : 'Item Price', 
                                                    'sum' : 'Total Purchase Value' })
# format columns
profitable_items['Item Price'] = profitable_items['Item Price'].map("$ {:,.2f}".format)
profitable_items['Total Purchase Value'] = profitable_items['Total Purchase Value'].map("$ {:,.2f}".format)

# set Index to Item ID and Item Name
profitable_items_df = profitable_items.set_index(['Item ID', 'Item Name'])
profitable_items_df[['Purchase Count', 'Item Price', 'Total Purchase Value']]
```
