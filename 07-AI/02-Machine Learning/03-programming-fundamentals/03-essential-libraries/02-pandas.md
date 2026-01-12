---
tags: ['ai', 'roadmap', 'python', 'pandas']
---

# Pandas (Python Data Analysis Library)

## Summary
**Pandas** is the industry-standard Python library for data manipulation and analysis. It provides high-performance, easy-to-use data structures—specifically the **DataFrame** and **Series**—designed to work with relational or labeled data. Built on top of NumPy, it offers essential tools for data cleaning, transformation, and statistical analysis, making it an indispensable part of the AI/ML engineer's toolkit.

## Detailed Explanation

### 1. Core Data Structures
Pandas operates primarily on two data structures:
*   **Series**: A one-dimensional labeled array. Think of it as a single column in a table.
*   **DataFrame**: A two-dimensional, size-mutable, potentially heterogeneous tabular data structure. Think of it as a spreadsheet or a SQL table.

```python
import pandas as pd
import numpy as np

# Creating a Series
s = pd.Series([1, 3, 5, np.nan, 6, 8])

# Creating a DataFrame
df = pd.DataFrame({
    'Date': pd.date_range('20230101', periods=6),
    'Score': [10, 20, 15, 25, 30, 12],
    'Label': ['A', 'B', 'A', 'C', 'B', 'A']
})
```

### 2. Data Inspection
Before analyzing data, you must understand its structure:
*   `df.head()` / `df.tail()`: View the first or last N rows.
*   `df.info()`: Summary of data types and memory usage.
*   `df.describe()`: Statistical summary of numerical columns (mean, std, min, max, etc.).
*   `df.shape`: Returns the dimensions (rows, columns).

### 3. Selection and Indexing
Pandas provides powerful ways to slice and dice data:
*   **`df['col_name']`**: Select a single column.
*   **`.loc[]`**: Label-based selection (rows and columns).
*   **`.iloc[]`**: Integer-position based selection.
*   **Boolean Indexing**: `df[df['Score'] > 20]` filters rows where the condition is met.

### 4. Data Cleaning
Real-world data is often messy. Pandas excels at:
*   **Handling Missing Data**:
    *   `df.isna()`: Detect missing values.
    *   `df.fillna(value)`: Fill missing values with a specific value or strategy (mean, median).
    *   `df.dropna()`: Remove rows or columns with missing values.
*   **Duplicates**: `df.drop_duplicates()` removes redundant rows.
*   **Renaming**: `df.rename(columns={'old_name': 'new_name'})`.

### 5. Grouping and Aggregation
The **Split-Apply-Combine** pattern is implemented via `groupby()`:
```python
# Calculate average score per label
avg_scores = df.groupby('Label')['Score'].mean()
```

### 6. Merging and Joining
Combine multiple datasets:
*   **`pd.merge(df1, df2, on='key')`**: SQL-style joins (Inner, Left, Right, Outer).
*   **`pd.concat([df1, df2])`**: Stacking dataframes vertically or horizontally.

### 7. Time Series
Pandas was originally developed for financial time-series data:
*   `pd.to_datetime()`: Convert strings to datetime objects.
*   `.resample()`: Frequency conversion (e.g., converting daily data to monthly).

## Interview Questions

### 1. What is the difference between `.loc` and `.iloc`?
**Answer**: `.loc` is label-based, meaning you use the names of rows or columns to select data. `.iloc` is integer-index based, meaning you use the numerical position (starting from 0) to select data.

### 2. How do you handle missing values in a DataFrame?
**Answer**: You can use `.isna()` or `.isnull()` to find them. To handle them, you can either remove them using `.dropna()` or fill them with a value (like a constant, the mean, or the median) using `.fillna()`.

### 3. What does `inplace=True` do in many Pandas methods?
**Answer**: It modifies the DataFrame directly without returning a new object. However, in modern Pandas versions, it is increasingly recommended to avoid `inplace=True` and instead reassign the result (`df = df.method()`) for better code clarity and to avoid `SettingWithCopyWarning`.

### 4. How can you combine two DataFrames in Pandas?
**Answer**: You can use `pd.concat()` to stack them on top of each other or side-by-side, or `pd.merge()` to perform database-style joins based on common columns (keys).

### 5. What is "Vectorization" in Pandas?
**Answer**: Vectorization is the process of performing operations on entire arrays (or Series/DataFrames) at once, rather than looping through individual elements. This is significantly faster because it leverages optimized C and Fortran code via NumPy.
