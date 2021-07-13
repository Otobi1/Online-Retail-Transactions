# Online-Retail-Transactions

## Data Collection

*Source and Description*: Data was sourced from the [UCI Machine Learining Repository](https://archive.ics.uci.edu/ml/datasets/Online+Retail+II). Dataset is a collection of the transaction of a retail store between 1st December 2009 and 9th December 2011.

*Objectives and Overview of Approaches to Analysis*: Objectives includes descriptive statistics, exploration of options for collaborative filtering (user-based and item-based) and clustering techniques for customer segementation.

*Data Loading*: The dataset consists in two parts, one on two tabs of the spreadsheet. Loading the data from each sheet requires the specification of the sheet name, in position 2 as follows.

```bash
df_09_10 = pd.read_excel('online_retail_II.xlsx', 'Year 2009-2010', index_col=None, na_values=['NA'])
```

The specification of the tab/sheet name is required for all the sheets/tabs in the workbook. After loading the dataset from each tab/sheet into a dataframe, you can place them into a list for concatenation as follows.

```bash
total_data = [data_09_10, data_10_11]

retail_data = pd.concat(total_data)
```

## Exploratory Data Analysis

*Checking and Handling Missing Values*: Asides using the standard "isnull().sum()" function of numpy, a better way to explore the proportions of missing data for each variable is to use the "plot_missing_value" function from "jcopml" library.

```bash
plot_missing_value(final_retail_data, return_df = True)
```

![the outcomes of the plot_missing_value function looks like this](https://github.com/Otobi1/Online-Retail-Transactions/blob/master/Snapshots/Screenshot%202021-07-05%20224555.jpg)

*Handling Outliers*: For transaction datasets, single extreme observations can skew the inferences drawn considerably. Hence, the need to check the distribution of the numerical variables and exclude extreme (rare) observations. The implementation of the Price variable is as follows. Same is for the Quantity variable, only need to swap out the variable name.

```bash
Q1_price = final_retail_data['Price'].quantile(0.25)
Q3_price = final_retail_data['Price'].quantile(0.75)

IQR_price = Q3_price - Q1_price

LB_price = float(Q1_price) - (1.5 * IQR_price)
UB_price = float(Q3_price) + (1.5 * IQR_price)

print ('Q1 = {}'.format(Q1_price))
print ('Q3 = {}'.format(Q3_price))

print ('IQR = Q3 - Q1 = {}'.format(IQR_price))
print ('lower bound = Q1 - 1.5 * IQR = {}'.format(LB_price))
print ('upper bound = Q1 - 1.5 * IQR = {}'.format(UB_price))
```

Further explorations, including plots, distributions and dataset peculiarities can be found in this [notebook](https://github.com/Otobi1/Online-Retail-Transactions/blob/master/Online_Retail_Transactions_EDA.ipynb). It also include various implementations of aggregation techniques using the "groupby" function.

## Collaborative Filtering

*Customer-Item Matrix*: first, we create a customer-item matrix to summarise the total quantity of each stock (item) each customer has purchased. The matrix will have a dimension equal to the number of the unique customers by the unique number of items (stock) to be purchased.

```bash
customer_item_matrix = clean_final_retail_data.pivot_table(index='Customer ID', columns='StockCode', values='Quantity', aggfunc='sum')
```

For collaborative filtering the number of items (stock) purchased is not as important as knowing whether a customer had purchased an item or not. Given that, we can map all stock purchases to 1 and those not purchased or returned to 0 using a lambda function as follows.

```bash
customer_item_matrix = customer_item_matrix.applymap(lambda x: 1 if x > 0 else 0)
```

*User-Based Collaborative Filtering*: a user-based collaborative filtering aims to explore the similarities and differences of each unique customer based what they've purchased within the customer-item similarity matrix. First, we use sklearn's cosine similarity to find the similarities between two or more non zero vectors. This [article](https://towardsdatascience.com/understanding-cosine-similarity-and-its-application-fd42f585296a) provides an interesting description of how cosine similarity works.

```bash
user_to_user_sim_matrix = pd.DataFrame(cosine_similarity(customer_item_matrix))
```

After that, we can explore the user to user similarity matrix and list out the specific items purchased by each unique customer. Subsequently, we can compare two customers on a small scale implementation and identify the products (stock/items) to be recommended to each customer based on the similarities of their purchase history.

*Item-Based Collaborative Filtering*: item-based collaborative filtering on the other hand focuses on the similarities in the items (stock). Sklearn's cosine similarity is applied to a transposed customer-item similarity matrix to create an item to item similarity matrix. Lastly, the most similar items can be listed for further exploration.

The implementation of the collaborative filtering can be found [here](https://github.com/Otobi1/Online-Retail-Transactions/blob/master/Online_Retail_Transactions_Collaborative_Filtering.ipynb)

Overall, collaborative filtering techniques are useful in building recommender systems, which is the foundation of several products from ad, ecommerce, movie, music recommendations.

## Customer Segmentation using K-Means Clustering based on Recency, Frequency and Monetary Value

*Estimating the Total Expenses per Customer*: first we read in the dataset and then create a new variable based on the unit price and the quantity of each item listed in the transaction dataset (feature engineering). Next, some descriptives to understand the data distribution including the proportions of transactions from each country tracked. Finally, we select only the data for the transactions only in the UK. We also restrict the transactions to those with quantities above 0.

*Restricting Data to One Year*: to create an RFM model, we need to choose an anchor variable, in this case, the invoice date, and restrict the dataset to a complete year or month as the case may be. Then further descriptives are explored within the single year period.

```bash
clean_final_retail_uk_data = clean_final_retail_uk_data[clean_final_retail_uk_data['InvoiceDate'] >= '2010-12-09']
```

*Recency*: to estimate the recency in the RFM model, we need to select a reference point (a specific date) to which the transactions will be compared.

```bash
now = dt.date(2011, 12, 9)

print (now)
```

```bash
recency_df = clean_final_retail_uk_data.groupby(by='CustomerID', as_index=False)['Date'].max()

recency_df.columns = ['CustomerID', 'LastPurchaseDate']

recency_df['Recency'] = recency_df['LastPurchaseDate'].apply(lambda x: (now - x).days)
```

*Frequency*: will be estimated by counting the number of invoices for each unique customer.

*Monetary Value*: estimated based on the total cost of each transaction summed for each unique customer.

*Merging - Single Dataset*: the recency, frequency and monetary variables are merged into a single dataset

*Handling Outliers*: for this model, it is important to effectively handle outliers as they could screw with the clustering algorithms. For instance, for recency, same frequency and monetary value as well.

```bash
Q1 = rfm_df.Recency.quantile(0.25)
Q3 = rfm_df.Recency.quantile(0.75)

IQR = Q3 - Q1

rfm_df = rfm_df[(rfm_df.Recency >= (Q1 - 1.5 * IQR)) & (rfm_df.Recency <= (Q3 + 1.5 * IQR))]
```

*Scaling*: following the handling outliers, it is important to scale the RFM variables to have instances between 0 and 1.

```bash
rfm_df['R'] = (rfm_df['Recency'] - rfm_df['Recency'].min()) / (rfm_df['Recency'].max() - rfm_df['Recency'].min())
```

*Hopkins Statistics*: aims to confirm the clustering tendency for unsupervised ML, given that clustering methods would return clusters even if the data does not contain any clusters. It enables the confirmation of inherent clusters in the dataset. Then you call the function on the datset and you get the proportion of the dataset that is suitable for clustering. Based on the hypothesis, whether the dataset is uniformly distributed and contains no meaningful clusters (H0) or where it is not uniformly distributed and contains meaningful clusters (H1), we reject HO if the [Hopkins stattistic](https://sushildeore99.medium.com/really-what-is-hopkins-statistic-bad1265df4b) is greater than 0.5 (essentially close to 1).

```bash
def hopkins(X):
  d = X.shape[1] # d = len(vars) # columns
  n = len(X) # rows
  m = int(0.1 * n)
  nbrs = NearestNeighbors(n_neighbors=1).fit(X.values)

  rand_X = sample(range(0, n, 1), m)

  ujd = []
  wjd = []
  for j in range(0, m):
    u_dist, _ = nbrs.kneighbors(uniform(np.amin(X, axis=0), np.amax(X, axis=0), d).reshape(1, -1), 2, return_distance=True)
    ujd.append(u_dist[0][1])
    w_dist, _ = nbrs.kneighbors(X.iloc[rand_X[j]].values.reshape(1, -1), 2, return_distance=True)
    wjd.append(w_dist[0][1])
    
  H = sum(ujd) / (sum(ujd) + sum(wjd))
  if isnan(H):
    print(ujd, wjd)
    H = 0

  return H
```

*Silhouette Scores*: [silhouette score](https://dzone.com/articles/kmeans-silhouette-score-explained-with-python-exam) is used to evaluate the quality of clusters created using algorithms such as K-Means (i.e., finding the optimum K for K-Means), in terms of how well samples are clustered with other samples that are similar to each other. The value of the Silhouette score varies from -1 to 1. If the score is 1, the cluster is dense and well-separated than other clusters. A value near 0 represents overlapping clusters with samples very close to the decision boundary of the neighboring clusters. A negative score [-1, 0] indicates that the samples might have got assigned to the wrong clusters.
