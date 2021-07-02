# Online-Retail-Transactions

## Data Collection

*Source and Description*:

*Objectives and Overview of Approaches to Analysis*:

*Data Loading*:

- Loading from two sheets in the same workbook
- Merging data from the two sheets into one dataframe

## Exploratory Data Analysis

*Checking and Handling Missing Values*:

- Checking for proportions and occurence using the jcopml's plot missing value function

*Handling Outliers*:

- Exploring outliers in the numeric variables

*Plots and Distributions*:

- Exploring the relationship between the numeric variables
- Identifying the dataset (transcation distribution) by country
- Checking the unique values per variable

*Dataset Peculiarities*:

- Exploring negative quantity transactions
- Exploring zero price transactions
- Aggregating transaction quantity and price by invoic date

*Clean Dataset*:

- Saving the clean data for further use

## Collaborative Filtering

*Customer-Item Matrix*:

- Using pivot tables to match the customers and the items they have bought.
- Exploring the shape, unique values, mapping the stock purchased to 1 and items not purchased to 0

*User-Based Collaborative Filtering*:

- User to user matrix via cosine similarity
- Listing items bought by specific users
- Comparing listed items with between 2 users and generating list of items to recommend to either users.

*Item-Based Collaborative Filtering*:

- Item to item matrix via cosine similarity
- Listing top 10 similar items

## Customer Segmentation using K-Means Clustering based on Recency, Frequency and Monetary Value

*Estimating the Total Expenses per Customer*:

*Descriptives*:

*Restricting Data to One Year*:

*Recency*: including last purchase date

*Frequency*: how many invoices per customer

*Monetary Value*: sum of total expenses per customer.

*Merging - Single Dataset*: on CustomerID

*Handling Outliers*:

*Scaling*:

*Distribution Plots*:

*Hopkins Statistics*:

*Dendrogram*:

*Clustering*:

*Finding the Optimum K for K-Means*:

*Silhouette Scores and Inertia*:

*Hierarchical Clustering*: to find the initial seeds

*Modelling*:

*Categorical Plots*:

*Relational Plots*:

*Pairplots*:
