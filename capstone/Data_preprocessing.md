# Methodology for Data Processing and Analysis

## Overview
This document outlines the step-by-step methodology employed to process and analyze the `aggregated_df` DataFrame. The methodology includes data preparation, inspection, cleaning, transformation, and final export.

## Data Processing Steps

### 1.1 Loading Data
- **Description**: The initial dataset was loaded into a DataFrame called `aggregated_df`. The data comprised multiple columns, including `Transaction Date`, `Sales Order No.`, `Customer Code`, `Inventory Code`, `Customer Name`, `Customer Category Desc`, `Inventory Desc`, `Price per qty`, `Qty`, `Total Base Amt`, among others.

### 1.2 Data Inspection
- **Objective**: To understand the structure and content of the dataset by inspecting data types, unique values, and missing values in each column.
- **Steps**:
  - **Transaction Date**: Checked for data types and ensured all entries were in the `datetime` format.
  - **Sales Order No.**: Verified the uniqueness of the `Sales Order No.` column, confirming it as an identifier.
  - **Customer Code**: Reviewed the `Customer Code` column to confirm its role as a categorical identifier.
  - **Inventory Code**: Inspected the `Inventory Code` to ensure it correctly represented different inventory items.
  - **Customer Name**: Analyzed customer names, identifying and correcting any inconsistencies in spelling.
  - **Customer Category Desc**: Reviewed the categories assigned to each customer, assessing the consistency and accuracy of category labels.
  - **Inventory Desc**: Examined the descriptions of inventory items, ensuring they were clear and appropriately categorized.
  - **Price per qty**: Checked for any anomalies in pricing, such as negative or zero values.
  - **Qty**: Assessed the quantity data for non-whole numbers and potential data entry errors.
  - **Total Base Amt**: Reviewed the `Total Base Amt` column to ensure all values were positive and consistent with the pricing and quantity data.

### 1.3 Data Cleaning and Transformation
- **Objective**: To clean and transform the dataset for subsequent analysis, ensuring consistency and accuracy.
- **Steps**:
  - **Date Conversion**: Converted the `Transaction Date` column to the `datetime` format to facilitate time-series analysis.
  - **Missing Values**: Ensured there were no missing values across all columns.
  - **Category Conversion**: Converted several categorical columns (`Customer Code`, `Inventory Code`, `Customer Name`, `Customer Category Desc`, `Inventory Desc`) to the `category` data type to optimize memory usage.
  - **Correction of Customer Names**: Identified and corrected misspellings or inconsistencies in `Customer Name`.
  - **Creation of New Columns**:
    - **Customer Category Broad**: Introduced a new column that grouped the original `Customer Category Desc` into broader categories (`Retail`, `Food Services`, `Institutional`, `Market`).
    - **Item Category**: Created another new column categorizing inventory descriptions into `Raw`, `RTE` (Ready-to-Eat), and `RTC` (Ready-to-Cook) based on certain keywords.

### 1.4 Memory Optimization
- **Objective**: To reduce the memory footprint of the DataFrame by converting appropriate columns to the `category` data type.
- **Steps**:
  - Converted `Customer Name`, `Customer Category Desc`, and `Inventory Desc` columns to `category` data type, significantly reducing memory usage.
  - Verified the memory savings by comparing memory usage before and after the conversion.

### 1.5 Final Data Inspection
- **Objective**: To verify that all transformations were correctly applied and that the data was ready for analysis.
- **Steps**:
  - Inspected the DataFrame structure and data types to confirm successful transformations.
  - Checked for any remaining anomalies or inconsistencies.
  - Verified that the newly created columns (`Customer Category Broad`, `Item Category`) accurately reflected the intended categorizations.

### 1.6 Exporting Processed Data
- **Objective**: To export the cleaned and processed data for further analysis or reporting.
- **Steps**:
  - Exported the full `aggregated_df` DataFrame to an S3 bucket as `aggregated_df.csv`.
  - Created and exported a version of the DataFrame without the `Customer Name` column as `no_customer_name_agg_df.csv`.

## Conclusion
This detailed methodology outlines the systematic approach taken to clean, process, and analyze the `aggregated_df` DataFrame. The steps described above ensure that the data is accurate, consistent, and optimized for further analysis or reporting. All transformations and checks were thoroughly validated to guarantee data integrity throughout the process.

# Data Dictionary for `no_customer_name_agg_df`

| **Column Name**              | **Data Type**          | **Description**                                                                                     |
|------------------------------|------------------------|-----------------------------------------------------------------------------------------------------|
| `Transaction Date`            | `datetime64[ns]`       | The date when the transaction occurred.                                                             |
| `Sales Order No.`             | `object`               | The unique identifier for the sales order.                                                          |
| `Customer Code`               | `category`             | The unique identifier for the customer.                                                             |
| `Inventory Code`              | `category`             | The unique identifier for the inventory item (SKU).                                                 |
| `Customer Category Desc`      | `category`             | A description of the customer category, such as "SUPERMARKET" or "RETAIL."                          |
| `Inventory Desc`              | `category`             | A description of the inventory item, such as "SP01 SKINLESS CHIX BREAST."                           |
| `Price per qty`               | `float64`              | The price per unit quantity of the inventory item.                                                  |
| `Qty`                         | `float64`              | The quantity of the inventory item purchased in the transaction.                                    |
| `Total Base Amt`              | `float64`              | The total amount for the transaction, calculated as `Price per qty` multiplied by `Qty`.            |
| `Customer Category Broad`     | `category`             | A broader categorization of the customer, such as "Retail", "Food Services", or "Institutional."    |
| `Item Category`               | `category`             | A categorization of the inventory item, such as "Raw", "RTE" (Ready-to-Eat), or "RTC" (Ready-to-Cook).|
