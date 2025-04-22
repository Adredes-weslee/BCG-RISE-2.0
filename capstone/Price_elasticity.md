# Methodology for Price Elasticity Analysis and Optimization

## Overview
This document outlines the step-by-step methodology employed to analyze price elasticity and optimize pricing strategies using the `merged_with_sku_ab` DataFrame. The methodology is divided into several parts, each covering specific aspects of the elasticity calculation, optimization, and analysis.

## Part 1: Price Elasticity Calculation

### 1.1 Data Preparation
- **Description**: The dataset was prepared by merging relevant SKUs into pairs for which cross-price elasticities and individual price elasticities were calculated. The key columns included `SKU A`, `SKU B`, `Price Elasticity SKU A`, `Price Elasticity SKU B`, and `Cross-Price Elasticity`.

### 1.2 Elasticity Calculation
- **Objective**: To calculate the price elasticity of demand for each SKU and the cross-price elasticity between paired SKUs.
- **Steps**:
  - **Price Elasticity**: Calculated the percentage change in quantity demanded relative to the percentage change in price for SKU A and SKU B.
  - **Cross-Price Elasticity**: Measured the impact of a price change in SKU A on the quantity demanded for SKU B, and vice versa.
  - **Formula**: 
    \[
    \text{Elasticity} = \frac{\%\ \text{Change in Quantity Demanded}}{\%\ \text{Change in Price}}
    \]
  - **Interpretation**:
    - Elasticity values greater than 1 indicate high sensitivity (elastic demand).
    - Elasticity values less than 1 indicate low sensitivity (inelastic demand).

## Part 2: Revenue Optimization

### 2.1 Model Setup
- **Objective**: To create an optimization model using Gurobi to determine the optimal price adjustments for SKUs that maximize total revenue.
- **Steps**:
  - **Model Initialization**: Created a Gurobi model named `price_optimization`.
  - **Decision Variables**: Defined decision variables for the percentage change in price for SKU A and SKU B, with bounds between -50% and 50%.
  - **Objective Function**: Set the objective function to maximize total revenue, which is calculated based on the new prices and quantities after applying the elasticity effects.
  - **Code snippet**:
    ```python
    # Create the Gurobi model
    model = gp.Model('price_optimization')

    # Define decision variables for the percentage change in price for SKU A and SKU B
    price_change_a = model.addVar(lb=-0.5, ub=0.5, vtype=GRB.CONTINUOUS, name="price_change_a")
    price_change_b = model.addVar(lb=-0.5, ub=0.5, vtype=GRB.CONTINUOUS, name="price_change_b")
    ```

### 2.2 Revenue Function Construction
- **Objective**: To build a revenue function that accounts for both own-price elasticity and cross-price elasticity, ensuring non-negativity constraints on quantities.
- **Steps**:
  - **Revenue Calculation**: For each SKU pair, calculated the new prices and quantities based on the defined elasticities.
  - **Constraints**: Added non-negativity constraints to ensure that the quantities remain positive.
  - **Total Revenue**: The revenue contributions from both SKUs were added to the total revenue function.
  - **Code snippet**:
    ```python
    # Loop through the rows in the dataframe to build the revenue function
    for index, row in merged_with_sku_ab.iterrows():
        # Calculate the new prices and quantities
        new_price_a = sku_a_price * (1 + price_change_a)
        new_price_b = sku_b_price * (1 + price_change_b)
        new_qty_a = sku_a_qty * (1 + sku_a_elasticity * price_change_a + cross_price_elasticity * price_change_b)
        new_qty_b = sku_b_qty * (1 + sku_b_elasticity * price_change_b + cross_price_elasticity * price_change_a)
        # Add non-negativity constraints and revenue contributions
        model.addConstr(new_qty_a >= 0, f"non_negativity_a_{index}")
        model.addConstr(new_qty_b >= 0, f"non_negativity_b_{index}")
        total_revenue += new_price_a * new_qty_a + new_price_b * new_qty_b
    ```

### 2.3 Optimization Execution
- **Objective**: To execute the optimization model and find the optimal price adjustments that maximize total revenue.
- **Steps**:
  - **Model Optimization**: The Gurobi model was optimized to find the solution that maximizes total revenue.
  - **Result Analysis**: After optimization, the results were analyzed to observe the impact of price adjustments on revenue.
  - **Code snippet**:
    ```python
    # Optimize the model
    model.optimize()

    # Output the results and calculate revenue changes
    if model.status == GRB.OPTIMAL:
        # Process results here
    ```

## Part 3: Optimization by Customer Category

### 3.1 Category-Specific Optimization
- **Objective**: To perform the optimization separately for each customer category (e.g., Supermarket, Retail) to understand category-specific impacts on revenue.
- **Steps**:
  - **Data Separation**: The dataset was separated into different customer categories.
  - **Category Optimization**: The optimization was performed independently for each category using the same methodology as described in Part 2.
  - **Code snippet**:
    ```python
    # Function to perform the optimization for a given dataframe
    def optimize_prices_for_category(df):
        # Create the Gurobi model and optimize as before
    ```

### 3.2 Result Aggregation
- **Objective**: To aggregate and compare the optimization results across customer categories.
- **Steps**:
  - **Data Concatenation**: The optimized results from each customer category were concatenated into a single DataFrame for comparison.
  - **Revenue Summary**: The total revenue change was summarized and compared across categories.
  - **Code snippet**:
    ```python
    # Concatenate the results from both categories
    final_revenue_changes_df = pd.concat([supermarket_revenue_changes_df, retail_revenue_changes_df])

    # Group by 'Customer Category' and sum the 'Total Change in Revenue'
    revenue_summary = final_revenue_changes_df.groupby('Customer Category')['Total Change in Revenue'].sum().reset_index()
    ```

## Part 4: Data Export

### 4.1 Exporting Optimized Data
- **Objective**: To export the final optimized data to a CSV file for further analysis or reporting.
- **Steps**:
  - **Data Export**: The final DataFrame containing the optimization results was exported to an S3 bucket in CSV format.
  - **Code snippet**:
    ```python
    # Convert DataFrame to CSV and upload to S3
    final_revenue_changes_df.to_csv(csv_buffer, index=False)
    s3_client.put_object(Bucket=s3_bucket, Key=s3_prefix, Body=csv_buffer.getvalue())
    ```

## Conclusion
This detailed methodology outlines the systematic approach taken to calculate price elasticity, optimize pricing strategies, and analyze the impacts on revenue across different customer categories. The steps described above ensure that the analysis was thorough, accurate, and aligned with business objectives. All transformations, optimizations, and checks were validated to guarantee data integrity and reliable results.

# Data Dictionary for `final_revenue_changes_df`

| **Column Name**                 | **Data Type**          | **Description**                                                                                                          |
|---------------------------------|------------------------|--------------------------------------------------------------------------------------------------------------------------|
| `Customer Category`             | `object`               | The category of the customer (e.g., Supermarket, Retail).                                                                |
| `Item Category`                 | `object`               | The category of the item (e.g., Raw, RTE, RTC).                                                                          |
| `SKU A`                         | `float64`              | The stock-keeping unit (SKU) code for the first product in the pair.                                                     |
| `SKU A Desc`                    | `object`               | The description of SKU A (e.g., SP01 SKINLESS CHIX BREAST).                                                              |
| `Original Price A`              | `float64`              | The original price of SKU A.                                                                                             |
| `Original Quantity A`           | `float64`              | The original quantity of SKU A sold.                                                                                     |
| `Optimized Price A`             | `float64`              | The optimized price for SKU A after applying the price elasticity model.                                                 |
| `Optimized Quantity A`          | `float64`              | The optimized quantity of SKU A sold after price optimization.                                                           |
| `SKU B`                         | `float64`              | The stock-keeping unit (SKU) code for the second product in the pair.                                                    |
| `SKU B Desc`                    | `object`               | The description of SKU B (e.g., SP02 CHIX BREAST).                                                                       |
| `Original Price B`              | `float64`              | The original price of SKU B.                                                                                             |
| `Original Quantity B`           | `float64`              | The original quantity of SKU B sold.                                                                                     |
| `Optimized Price B`             | `float64`              | The optimized price for SKU B after applying the price elasticity model.                                                 |
| `Optimized Quantity B`          | `float64`              | The optimized quantity of SKU B sold after price optimization.                                                           |
| `Original Revenue A`            | `float64`              | The revenue generated from SKU A at the original price.                                                                  |
| `Optimized Revenue A`           | `float64`              | The revenue generated from SKU A at the optimized price.                                                                 |
| `Change in Revenue A`           | `float64`              | The change in revenue from SKU A due to the price optimization.                                                          |
| `Original Revenue B`            | `float64`              | The revenue generated from SKU B at the original price.                                                                  |
| `Optimized Revenue B`           | `float64`              | The revenue generated from SKU B at the optimized price.                                                                 |
| `Change in Revenue B`           | `float64`              | The change in revenue from SKU B due to the price optimization.                                                          |
| `Total Change in Revenue`       | `float64`              | The total change in revenue for the SKU pair (SKU A and SKU B) after price optimization.                                 |
| `Cross-Price Elasticity`        | `float64`              | The cross-price elasticity of demand between SKU A and SKU B.                                                            |
| `Price Elasticity SKU A`        | `float64`              | The price elasticity of demand for SKU A.                                                                                |
| `Price Elasticity SKU B`        | `float64`              | The price elasticity of demand for SKU B.                                                                                |
| `Product Pair Type`             | `object`               | The type of relationship between SKU A and SKU B (e.g., complements - close, substitutes).                               |
| `SKU A Latest Transaction Date` | `object`               | The date of the latest transaction for SKU A.                                                                            |
| `SKU B Latest Transaction Date` | `object`               | The date of the latest transaction for SKU B.                                                                            |
