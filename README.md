# grocery-store-sales-sql-analysis
FoodYum, a grocery store chain based in the United States, aims to maintain a diverse product range across various price points to cater to a broad customer base, especially as food costs continue to rise. This project focuses on analyzing their product catalog to ensure data quality and extract valuable insights.

---

# SQL Data Analysis: FoodYum Grocery Store Sales

## Project Overview

FoodYum, a grocery store chain based in the United States, aims to maintain a diverse product range across various price points to cater to a broad customer base, especially as food costs continue to rise. This project focuses on analyzing their product catalog to ensure data quality and extract valuable insights.

## Data

The analysis utilizes a `products` table containing comprehensive records of items stocked by FoodYum, including details from their last full year of the loyalty program.

| Column Name        | Criteria                                                                                                                                                                                                                               |
| :----------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `product_id`       | Nominal. The unique identifier of the product. Missing values are not possible.                                                                                                                                                        |
| `product_type`     | Nominal. The product category (Produce, Meat, Dairy, Bakery, Snacks). Missing values should be replaced with "Unknown".                                                                                                                |
| `brand`            | Nominal. The brand of the product (one of 7 values). Missing values should be replaced with "Unknown".                                                                                                                                 |
| `weight`           | Continuous. The weight of the product in grams (positive, rounded to 2 decimal places). Missing values should be replaced with the overall median weight.                                                                            |
| `price`            | Continuous. The price in US dollars (positive, rounded to 2 decimal places). Missing values should be replaced with the overall median price.                                                                                        |
| `average_units_sold` | Discrete. The average number of units sold each month (positive integer). Missing values should be replaced with 0.                                                                                                                  |
| `year_added`       | Nominal. The year the product was first added to stock. Missing values should be replaced with 2022.                                                                                                                                   |
| `stock_location`   | Nominal. The warehouse location (A, B, C, or D). Missing values should be replaced with "Unknown".                                                                                                                                     |

---

## Task 1: Identify Missing `year_added` Values

**Objective:** Determine the count of products where the `year_added` value is missing. This is crucial for understanding the extent of a known data bug from 2022.

**SQL Query:**

```sql
SELECT COUNT(*) AS missing_year
FROM products
WHERE year_added IS NULL;
```

**Output:**

```
 missing_year
0          170
```

**Insight:** There are **170 products** with missing `year_added` values, confirming the presence of the data bug.

---

## Task 2: Data Cleaning and Preparation

**Objective:** Clean the product data according to the specified criteria, handling missing values and ensuring data integrity without modifying the original table. This step creates a clean dataset for subsequent analysis.

**SQL Query:**

```sql
SELECT
    product_id,
    COALESCE(product_type, 'Unknown') AS product_type,
    COALESCE(NULLIF(REPLACE(brand, '-', ''), ''), 'Unknown') AS brand,
    COALESCE(CAST(REGEXP_REPLACE(weight, '[^\\d.]', '', 'g') AS DECIMAL(10, 2)), (SELECT PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY CAST(REGEXP_REPLACE(weight, '[^\\d.]', '', 'g') AS DECIMAL(10, 2))) FROM products)) AS weight,
    COALESCE(CAST(price AS DECIMAL(10, 2)), (SELECT PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY price) FROM products)) AS price,
    COALESCE(average_units_sold, 0) AS average_units_sold,
    COALESCE(year_added, 2022) AS year_added,
    COALESCE(UPPER(stock_location), 'Unknown') AS stock_location
FROM products;
```

**Explanation of Cleaning Logic:**

* **`COALESCE(column, 'default_value')`**: Used extensively to replace `NULL` values with specified defaults ("Unknown", 0, or 2022).
* **`NULLIF(REPLACE(brand, '-', ''), '')`**: Specifically handles the `brand` column by first removing hyphens and then converting empty strings (resulting from only hyphens) to `NULL` before `COALESCE` can replace them with "Unknown". This ensures robust handling of potentially messy brand data.
* **Median Imputation for `weight` and `price`**:
    * **`REGEXP_REPLACE(weight, '[^\\d.]', '', 'g')`**: This addresses potential non-numeric characters in the `weight` column by removing anything that isn't a digit or a decimal point before casting.
    * **`CAST(... AS DECIMAL(10, 2))`**: Ensures correct data types for numerical columns.
    * **`PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY column) FROM products`**: This subquery dynamically calculates the median value for `weight` and `price` across the entire `products` table. This approach ensures that missing numerical values are filled with a statistically robust central tendency measure, minimizing distortion from outliers.
* **`UPPER(stock_location)`**: Standardizes `stock_location` to uppercase for consistency.

---

## Conclusion and Future Work

This project demonstrates proficiency in SQL for essential data engineering tasks, including complex data cleaning, imputation, and preparation for business analysis. By addressing data quality issues upfront, we ensure that subsequent analyses (e.g., pricing strategy optimization or sales performance evaluation) are based on reliable information.

**Potential Future Analysis:**

* Analyze product profitability by `product_type` and `brand`.
* Investigate the impact of `year_added` on product `price` or `average_units_sold`.
* Identify optimal stocking levels based on `average_units_sold` and `stock_location` efficiency.
* Develop a dashboard for key product metrics using a BI tool.

---
