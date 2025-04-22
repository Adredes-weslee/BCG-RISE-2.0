# Methodology for Customer Segmentation Using RFM Analysis and K-Means Clustering

## Overview

This document outlines the step-by-step methodology used to perform customer segmentation using Recency, Frequency, and Monetary (RFM) analysis combined with K-Means clustering. The goal is to identify distinct customer segments based on their purchasing behaviors and analyze these segments in relation to customer categories and item categories.

## 1. Data Preparation

### 1.1 Selection of Relevant Features

- **Objective**: To select key features that represent customer purchasing behaviors for segmentation analysis.
- **Features Selected**:
  - `Customer Code`
  - `Recency`: Number of days since the last purchase.
  - `Frequency`: Total number of transactions.
  - `Monetary`: Total monetary value of the transactions.
  - `Total Quantity`: Total quantity of items purchased.

### 1.2 Data Scaling

- **Objective**: To scale the selected features to ensure they are on a comparable scale and reduce the impact of outliers.
- **Method**: `RobustScaler` was used, which scales features using statistics that are robust to outliers (e.g., median and interquartile range).

## 2. RFM Segmentation

### 2.1 RFM Binning

- **Objective**: To categorize customers into segments based on their Recency, Frequency, and Monetary values using RFM binning.
- **Steps**:
  - **RFM Scoring**: Each customer was assigned a score for Recency, Frequency, and Monetary based on their percentile rank. Scores ranged from 1 (lowest) to 5 (highest).
  - **RFM Segmentation**: Customers were grouped into segments based on their combined RFM scores, creating distinct customer profiles (e.g., `Champions`, `Loyal Customers`, `At Risk`, etc.).

### 2.2 RFM Segment Assignment

- **Objective**: To assign each customer to an RFM segment based on their RFM scores.
- **Steps**:
  - **RFM Concatenation**: Combined the individual R, F, and M scores into a single string to create unique segment identifiers.
  - **Segment Assignment**: Mapped these combined scores to predefined customer segments.

## 3. K-Means Clustering

### 3.1 First Run of K-Means

#### 3.1.1 Determining the Optimal Number of Clusters

- **Objective**: To identify the optimal number of clusters for the K-Means algorithm.
- **Method**:
  - Computed Inertia and Silhouette scores for cluster counts ranging from 2 to 12.
  - Used Elbow and Silhouette plots to visually assess the optimal number of clusters.
- **Result**: 3 clusters were chosen as the optimal number for the first run.

#### 3.1.2 Clustering and Assignment of Labels

- **Objective**: To segment customers based on their purchasing behaviors.
- **Method**: Applied K-Means clustering with 3 clusters.
- **Outcome**: Each customer was assigned a cluster label.

### 3.2 Second Run of K-Means on Cluster 1

#### 3.2.1 Further Segmentation of Cluster 1

- **Objective**: To further segment Cluster 1 from the first run into more granular sub-clusters.
- **Method**:
  - Repeated the process of determining the optimal number of clusters within Cluster 1.
  - Selected 3 clusters as the optimal number for this second run.

#### 3.2.2 Combining and Renaming Clusters

- **Objective**: To combine the results from both runs of K-Means clustering.
- **Method**:
  - Combined the clusters from the second run with those from the first run.
  - Renumbered the cluster labels for sequential representation.

## 4. Merging RFM Segments with K-Means Clusters

### 4.1 Integration of RFM Segments

- **Objective**: To integrate RFM segmentation with K-Means clusters to enrich the segmentation analysis.
- **Method**: Merged the original RFM DataFrame with the combined DataFrame containing K-Means cluster labels based on the `Customer Code`.

### 4.2 Analysis and Visualization

#### 4.2.1 Crosstab Analysis

- **Objective**: To explore the relationship between RFM segments and K-Means clusters.
- **Method**: Created a crosstab to visualize the distribution of customers across RFM segments and K-Means clusters.

#### 4.2.2 Heatmaps

- **Objective**: To visualize the overlap and differences between RFM segments and K-Means clusters.
- **Method**: Generated heatmaps based on the crosstab results.

## 5. Customer and Item Category Analysis

### 5.1 Customer Category Analysis

- **Objective**: To analyze customer category descriptions in relation to RFM segments and K-Means clusters.
- **Method**:
  - Grouped data by customer category and segment.
  - Counted occurrences within each segment and visualized the distribution using bar plots.

### 5.2 Item Category Analysis

- **Objective**: To analyze item categories in relation to RFM segments and K-Means clusters.
- **Method**:
  - Grouped data by item category and segment.
  - Counted occurrences within each segment and visualized the distribution using bar plots and heatmaps.

## 6. Exporting Results

### 6.1 Export to CSV

- **Objective**: To export the final segmented data for further analysis or reporting.
- **Method**:
  - Exported the final DataFrame containing customer segmentation data, including customer and item categories, to a CSV file.
  - Uploaded the CSV file to an S3 bucket for storage.

## Conclusion

This detailed methodology outlines the approach taken to perform customer segmentation using RFM analysis combined with K-Means clustering. The integration of customer and item category analysis provides a comprehensive understanding of customer behaviors within each segment, offering valuable insights for targeted marketing strategies.

# Data Dictionary for `cust_broad`

| **Column Name**              | **Data Type**          | **Description**                                                                                     |
|------------------------------|------------------------|-----------------------------------------------------------------------------------------------------|
| `Customer Code`              | `object`               | The unique identifier for the customer.                                                             |
| `Recency`                    | `int64`                | The number of days since the customer's last purchase.                                              |
| `Frequency`                  | `int64`                | The total number of purchases made by the customer.                                                 |
| `Monetary`                   | `float64`              | The total monetary value of the customer's purchases.                                               |
| `Total Quantity`             | `float64`              | The total quantity of items purchased by the customer.                                              |
| `R`                          | `category`             | The recency score for the customer, used in RFM analysis.                                           |
| `F`                          | `category`             | The frequency score for the customer, used in RFM analysis.                                         |
| `M`                          | `category`             | The monetary score for the customer, used in RFM analysis.                                          |
| `Score`                      | `int64`                | The combined RFM score for the customer, calculated by summing the R, F, and M scores.              |
| `RFM_Segment`                | `object`               | The customer segment based on RFM analysis, such as "New_Customers" or "Champions."                 |
| `KMeans_Segment`             | `int64`                | The segment assigned to the customer based on K-Means clustering.                                   |
| `Customer Category Desc`     | `object`               | A description of the customer category, such as "RETAIL" or "WHOLESALER."                           |
| `Item Category`              | `object`               | A categorization of the inventory item, such as "Raw", "RTE" (Ready-to-Eat), or "RTC" (Ready-to-Cook).|
