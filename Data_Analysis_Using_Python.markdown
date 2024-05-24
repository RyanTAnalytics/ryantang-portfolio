---
jupyter:
  kernelspec:
    display_name: Python 3
    language: python
    name: python3
  language_info:
    codemirror_mode:
      name: ipython
      version: 3
    file_extension: .py
    mimetype: text/x-python
    name: python
    nbconvert_exporter: python
    pygments_lexer: ipython3
    version: 3.12.3
  nbformat: 4
  nbformat_minor: 2
---

::: {.cell .markdown}
### Data Analysis of Brazilian E-Commerce Dataset with Python (Pandas)

Let\'s use Pandas to explore and analyze some data. The public dataset
was taken from the [Brazilian e-commerce platform,
Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce). It
has been filtered to only contain 2017 data.

**Scenario:**

Olist, a Brazilian company providing a marketplace integration platform
for small and medium-sized businesses, seeks to leverage data analytics
to optimize its sales strategies. The company has access to a
comprehensive public dataset containing various aspects of their
e-commerce activities, including customer information, order details,
product listings, seller performance, and more.

*Objectives:*

1.  Analyze the changes in total sales over time in 2017.
2.  Evaluate the performance of different categories to identify
    top-selling items and underperformers.
3.  Identify which regions generate the most sales to inform sales
    strategies.
4.  Assess seller performance to provide insights and recommendations
    for improving seller ratings and sales.

*Research Questions:*

1.  How do total sales fluctuate over the year and week?
2.  Which categories of products are most popular and least popular?
3.  Which regions generate the most sales from customers?
4.  Which regions contain the highest performing sellers?
:::

::: {.cell .markdown}
#### Phase 1: Data Exploration

Let\'s start by only looking at the `olist_order_items_2017.csv` data
and explore its properties.
:::

::: {.cell .code}
``` python
import pandas as pd
import matplotlib.pyplot as plt

#Reading in the data
order_items = pd.read_csv('olist_order_items_2017.csv')
```
:::

::: {.cell .markdown}
##### Phase 1.1: Inspecting the DataFrame {#phase-11-inspecting-the-dataframe}
:::

::: {.cell .code}
``` python
# Using head() to show the first 5 rows
order_items.head()
```
:::

::: {.cell .code}
``` python
# Using tails() to show the last 5 rows
order_items.tail()
```
:::

::: {.cell .code}
``` python
# Using shape to view number of rows and columns
order_items.shape
```
:::

::: {.cell .code}
``` python
# Using info to view column names/types and more
order_items.info()
```
:::

::: {.cell .markdown}
##### Phase 1.2: Data Summaries {#phase-12-data-summaries}
:::

::: {.cell .code}
``` python
# Using describe to view descriptive statistics
order_items.describe()
```
:::

::: {.cell .code}
``` python
# Using unique() to look at categorical values
order_items['product_category'].unique()
```
:::

::: {.cell .code}
``` python
# Using value_counts() to reveal product categories with highest sales
order_items['product_category'].value_counts()
```
:::

::: {.cell .code}
``` python
# Using value_counts() to reveal sellers with highest sales
order_items['seller_id'].value_counts()
```
:::

::: {.cell .markdown}
#### Phase 2: Data Transformation & Wrangling {#phase-2-data-transformation--wrangling}

We have looked at the ordered items in the Olist e-commerce platform.
Let\'s look at when these orders were placed.

Let\'s look into the `olist_orders_2017.csv` dataset which seems to
contain some missing values.
:::

::: {.cell .code}
``` python
# Reading in the orders dataset
orders = pd.read_csv('olist_orders_2017.csv')
```
:::

::: {.cell .code}
``` python
# Checking its info
orders.info()
```
:::

::: {.cell .markdown}
##### Phase 2.1: Identifying Missing Data {#phase-21-identifying-missing-data}
:::

::: {.cell .code}
``` python
# Using isnull() and sum() to count the number of missing values
orders.isnull().sum()
```
:::

::: {.cell .markdown}
#### Phase 2.2: Filling in Missing Data {#phase-22-filling-in-missing-data}
:::

::: {.cell .code}
``` python
# Make a copy to protect original data
orders_copy = orders.copy()

# Remove the rows which really have all missing values
orders.dropna(how='all', inplace=True)

# Fill in missing values 
orders.fillna(value={'order_status':orders['order_status'].mode()[0],
                     'approval_lag':orders['approval_lag'].median()}, inplace=True)

#Check nulls
orders.isnull().sum()
```
:::

::: {.cell .code}
``` python
# Retrieve the original DataFrame if needed
# orders =  orders_copy.copy()
```
:::

::: {.cell .markdown}
##### Phase 2.2: Renaming Columns {#phase-22-renaming-columns}
:::

::: {.cell .code}
``` python
# Rename functions to be more mnemonic 
old_and_new = {'product_category':'category', 'shipping_limit_date':'ship_date'}

order_items.rename(columns=old_and_new, inplace=False)
```
:::

::: {.cell .markdown}
##### Phase 2.3: Dropping Columns {#phase-23-dropping-columns}
:::

::: {.cell .code}
``` python
# drop the irrelevant columns from order_items
order_items.drop(['shipping_limit_date', 'freight_value'], axis='columns', inplace=True)

# drop the irrelevant columns from orders
orders.drop(['order_delivered_carrier_date', 'order_delivered_customer_date', 'order_estimated_delivery_date'], 
            axis='columns', inplace=True)
```
:::

::: {.cell .markdown}
##### Phase 2.4: Merging Data {#phase-24-merging-data}

Since we need retain **all** the data in the product_ids column within
the order_items dataset, we will use a `RIGHT JOIN` here.
:::

::: {.cell .code}
``` python
# Create a DataFrame to record products ordered, category and when they were purchased.
ordered_products = pd.merge(orders, order_items, how='right', on='order_id')
```
:::

::: {.cell .markdown}
For our analysis to meet the objectives effectively, we would also need
to bring in data from customer and seller data.
:::

::: {.cell .code}
``` python
# Reading in the datasets
customers = pd.read_csv('olist_customers_2017.csv')
sellers = pd.read_csv('olist_sellers_2017.csv')
```
:::

::: {.cell .code}
``` python
# Check the columns for customers
customers.columns
```
:::

::: {.cell .markdown}
Similarly, we want to retain the product_ids column. Therefore, we will
use a `LEFT JOIN` here.
:::

::: {.cell .code}
``` python
# Merge the ordered_products with customers to find out who the product is sold to
merged_df = pd.merge(ordered_products, customers, how='left', on='customer_id')
```
:::

::: {.cell .markdown}
Finally, we will use a `LEFT JOIN` here as well to retain the
product_ids column.
:::

::: {.cell .code}
``` python
# Merge the newly merged DataFrame with sellers to find out who the product is sold by
merged_df = pd.merge(merged_df, sellers, how='left', on='seller_id')
```
:::

::: {.cell .code}
``` python
# View the merged dataset
merged_df
merged_df.columns
```
:::

::: {.cell .markdown}
##### Phase 2.5: Cleaning Up {#phase-25-cleaning-up}

After merging, we seem to have a wide variety of columns, many of which
are irrelevant to our objective or are not needed for our analysis.
Therefore, we will drop them.
:::

::: {.cell .code}
``` python
# Drop columns and save into a new DataFrame
products_sold = merged_df.drop(['customer_id', 'order_approved_at', 'approval_lag', 'order_item_id',
                                 'seller_id', 'customer_unique_id', 'customer_zip_code_prefix', 
                                  'customer_city', 'seller_zip_code_prefix', 'seller_city'],axis='columns')

# Looking into the new dataframe
products_sold.head(10)
```
:::

::: {.cell .code}
``` python
#Checking data types
products_sold.dtypes
```
:::

::: {.cell .markdown}
After inspecting the datatype, it seems that the
`order_purchase_timestamp` has been incorrectly assigned as an a
categorical object. Let\'s convert it into the correct format.
:::

::: {.cell .code}
``` python
# Convert to DateTime using to_datetime() method
products_sold['order_purchase_timestamp']= pd.to_datetime(products_sold['order_purchase_timestamp'])
```
:::

::: {.cell .markdown}
**Get Date Information**

Now we can use DateTime functions to get more information about the
purchase date.
:::

::: {.cell .code}
``` python
# Month of Order
products_sold['order_month'] = products_sold['order_purchase_timestamp'].dt.month

# Day of Week 
products_sold['order_day'] = products_sold['order_purchase_timestamp'].dt.weekday

# Day Name
products_sold['order_day_name'] = products_sold['order_purchase_timestamp'].dt.day_name()

# Hour of the day
products_sold['order_hour'] = products_sold['order_purchase_timestamp'].dt.hour
```
:::

::: {.cell .markdown}
To make our analysis more fruitful and straightforward (clearer and less
complicated), we can further breakdown the categories.
:::

::: {.cell .code}
``` python
# Fill in missing values in category as 'Other'
products_sold['product_category'].fillna('Other', inplace=True)

# Rename category as subcategory
products_sold.rename(columns={'product_category':'subcategory'}, inplace=True)

# Define a function to break down the categories further
def set_category(sub_category):
    if 'fashion' in sub_category:
        return 'fashion'
    elif 'furniture' in sub_category:
        return 'furniture'
    elif 'tool' in sub_category:
        return 'tools'
    elif 'home' in sub_category or sub_category in ['housewares','bed_bath_table']:
        return 'home'
    elif sub_category in ['consoles_games', 'computers', 'computers_accessories', 'electronics', 'tablets_printing_image']:
        return 'electronics'
    elif sub_category in ['audio','music','musical_instruments']:
        return 'audio'
    elif sub_category in ['baby','toys', 'cool_stuff']:
        return 'kids'
    else:
        return 'other category'
    
```
:::

::: {.cell .code}
``` python
# Apply the function and create a new column based on the subcategory
products_sold['category'] = products_sold['subcategory'].apply(set_category)

# Check the unique values in the category column
products_sold['category'].unique()
```
:::

::: {.cell .code}
``` python
# Check the value counts
products_sold['category'].value_counts()
```
:::

::: {.cell .markdown}
##### Saving the DataFrame
:::

::: {.cell .code}
``` python
# Saving the merged data to a new file called products_sold_2017.csv, without adding an index column
products_sold.to_csv('products_sold_2017.csv', index=False)
```
:::

::: {.cell .markdown}
#### Phase 3: Data Analysis

Great! Our data is now properly explored, transformed, and wrangled.

Now it is time to do some analysis.
:::

::: {.cell .code}
``` python
# Check the categories values, including missing values
products_sold['category'].value_counts(dropna=False)
```
:::

::: {.cell .markdown}
##### Phase 3.1: Grouping and Summarizing {#phase-31-grouping-and-summarizing}

As you can see above, we have been able to count the number of order
items by each category.

We can summarize the data by category, sub_category, month, day of order
and even time of order!
:::

::: {.cell .code}
``` python
# Grouping by category, then count the number of order_ids for each category
products_sold.groupby('category')['order_id'].count()
```
:::

::: {.cell .code}
``` python
# Summary values of sales by month
products_sold.groupby('order_month')['price'].agg(['count', 'sum','mean','min','max'])
```
:::

::: {.cell .code}
``` python
# Summary values of sales by day 
products_sold.groupby('order_day_name')['price'].agg(['count', 'sum','mean','min','max'])
```
:::

::: {.cell .markdown}
#### Phase 4: Data Visualization

Visualization 1: Total Sales by Month
:::

::: {.cell .code}
``` python
plotdata = products_sold.groupby('order_month')['price'].sum()
plotdata.plot(title='Total Sales by Month in 2017')
plt.xlabel('Month')
plt.ylabel('Total Sales')
plt.show()
```
:::

::: {.cell .markdown}
Visualization 2: Fluctuations of Total Sales throughout the Week by
Month
:::

::: {.cell .code}
``` python
plotdata = products_sold.groupby(['order_month', 'order_day_name'])['price'].count().unstack()

# Reorder the columns to match the order of the days of the week
day_order = ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday']
plotdata = plotdata[day_order]

# Plot the data
plotdata.plot(title="Fluctuations of Total Sales throughout the Week by Month")
plt.xlabel('Month')
plt.ylabel('Total Sales')
plt.legend(title='Day of Week', labels=day_order)
plt.show()
```
:::

::: {.cell .markdown}
Visualization 3: Total Sales by Category
:::

::: {.cell .code}
``` python
plotdata = products_sold.groupby(['category'])['price'].count()
plotdata.plot(kind='barh', title='Total Sales by Category')
plt.xlabel('Category')
plt.ylabel('Total Sales')
plt.show()
```
:::

::: {.cell .markdown}
Visualization 4: Top 10 Performing States
:::

::: {.cell .code}
``` python
plotdata = products_sold.groupby(['customer_state'])['price'].sum()
plotdata.sort_values().tail(10).plot(kind='barh', title='Top 10 Performing States')
plt.xlabel('Total Sales')
plt.ylabel('Customer State')
plt.show()
```
:::

::: {.cell .markdown}
Visualization 5: Top 10 Performing Sellers
:::

::: {.cell .code}
``` python
plotdata = products_sold.groupby(['seller_state'])['price'].sum()
plotdata.sort_values().tail(10).plot(kind='barh', title='Top 10 Performing Sellers')
plt.xlabel('Total Sales')
plt.ylabel('Seller State')
plt.show()
```
:::

::: {.cell .markdown}
#### Phase 5: Conclusions & Suggestions {#phase-5-conclusions--suggestions}

1.  How do total sales fluctuate over the year and week?

-   Based on Visualization 1, it seems that total sales starts out low
    and increases rather gradually over the year. However, there is a
    large spike at the end of the year, peaking in November. A similar
    trend can be seen in Visualization 2, where total sales gradually
    increase over the week, generally peaking on Friday for all months.
    There is a significant spike on Friday in November, where total
    sales almost double that of other days of the week.
-   Suggestions:
    -   Given the spike in November, particularly on Fridays, it's clear
        that Black Friday and Cyber Monday are crucial periods. Plan
        extensive marketing campaigns and promotions targeting these
        dates.
    -   Extend special promotions through December to maintain the
        momentum from the November peak, focusing on holiday shopping.
    -   Implement regular "Flash Sale Fridays" with special discounts
        and promotions to capitalize on the natural peak in sales.
    -   Increase inventory for high-demand products well in advance of
        November to avoid stockouts.

1.  Which categories of products are most popular and least popular?

-   Based on Visualization 3, it seems that products in the home
    category brings in the most sales individually,followed by kids,
    furniture, and electronics respectively. Though, the categories
    could be broken down further to reveal more interesting insights.
-   Suggestions:
    -   Highlight home category products prominently on the homepage and
        in marketing campaigns.
    -   Align home category promotions with seasonal events such as
        spring cleaning, back-to-school, and holiday decorating
    -   Ensure adequate stock levels for high-demand products in the
        home, kids, furniture, and electronics categories, especially
        during peak sales periods.

1.  Which regions generate the most sales from customers?

-   Based on Visualization 4, it seems that highest total sales come
    overwhelmingly from customers in Sao Paulo compared to other states
    in Brazil like Rio de Janeiro and Minas Gerais. This is not
    surprising given Sao Paulo is the most populous and developed state
    in Brazil
-   Suggestions:
    -   Increase investment in digital and traditional advertising
        specifically targeting São Paulo. Utilize platforms like Google
        Ads, Facebook, and Instagram with geo-targeting to reach São
        Paulo residents.
    -   Consider establishing local warehouses or distribution centers
        in São Paulo to reduce delivery times and improve logistics
        efficiency.

1.  Which regions contain the highest performing sellers?

-   Based on Visualization 5, highest performing sellers in terms of
    total sales also seem to be from Sao Paulo, followed by Parana and
    Minas Gerais.
-   Suggestions:
    -   Work with logistics partners to offer more efficient and
        cost-effective shipping solutions for sellers, especially in
        high-performing regions like São Paulo, Paraná, and Minas
        Gerais.
    -   Provide tools and training for better inventory management,
        ensuring sellers can meet demand without overstocking.
:::
