import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

# Load the data from the CSV files
orders_df = pd.read_csv('customer_orders.csv')
payments_df = pd.read_csv('payments (1).csv')

# Convert order_date and payment_date to datetime
orders_df['order_date'] = pd.to_datetime(orders_df['order_date'])
payments_df['payment_date'] = pd.to_datetime(payments_df['payment_date'])

# Merge the orders and payments dataframes on order_id
merged_df = pd.merge(orders_df, payments_df, on='order_id')

# Extract year and month from order_date
merged_df['order_year_month'] = merged_df['order_date'].dt.to_period('M')

# Create a cohort column based on the first purchase month of each customer
merged_df['cohort'] = merged_df.groupby('customer_id')['order_date'].transform('min').dt.to_period('M')

# Calculate the number of months since the first purchase for each order
merged_df['months_since_first_purchase'] = (merged_df['order_year_month'] - merged_df['cohort']).apply(lambda x: x.n)

# Create a pivot table to count the number of unique customers in each cohort who made repeat purchases in subsequent months
cohort_pivot = merged_df.pivot_table(index='cohort', columns='months_since_first_purchase', values='customer_id', aggfunc='nunique')

# Plot the cohort analysis heatmap
plt.figure(figsize=(12, 8))
sns.heatmap(cohort_pivot, annot=True, fmt='.0f', cmap='Blues')
plt.title('Customer Retention by Cohort')
plt.xlabel('Months Since First Purchase')
plt.ylabel('Cohort (First Purchase Month)')
plt.show()
