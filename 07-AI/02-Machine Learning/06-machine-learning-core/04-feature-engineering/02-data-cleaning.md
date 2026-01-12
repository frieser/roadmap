---
tags: ['ai', 'roadmap']
---

## Summary
**Data Cleaning** is the fundamental process of identifying and correcting (or removing) errors, inconsistencies, and inaccuracies in a dataset to improve its quality. In the AI/ML pipeline, it is often the most time-consuming but critical step, as the principle of **"Garbage In, Garbage Out"** dictates that even the most sophisticated models will perform poorly if trained on "dirty" data.

## Detailed Explanation

### Handling Duplicates
Duplicate records can occur due to data entry errors, scraping glitches, or merging multiple datasets. They can bias the model by giving undue weight to specific observations.

**Python Implementation:**
```python
import pandas as pd

# Create a sample dataframe with duplicates
data = {'id': [1, 2, 2, 3], 'value': ['A', 'B', 'B', 'C']}
df = pd.DataFrame(data)

# Check for duplicates
print(f"Duplicates found: {df.duplicated().sum()}")

# Remove duplicates (keeping the first occurrence)
df_cleaned = df.drop_duplicates()
```

### Inconsistent Data
Inconsistencies often involve variations in naming conventions, formats, or units. For example, a "Country" column might contain "USA", "U.S.A.", and "United States".

**Strategies:**
1. **String Normalization:** Converting to lowercase and removing leading/trailing whitespace.
2. **Format Standardization:** Using `pd.to_datetime()` for dates or converting all units to a common metric.
3. **Mapping/Replacement:** Using dictionaries to consolidate categorical values.

**Python Implementation:**
```python
# Standardizing string data
df['city'] = df['city'].str.strip().str.lower()

# Mapping inconsistent labels
city_mapping = {
    'nyc': 'new york',
    'ny': 'new york',
    'n.y.c': 'new york'
}
df['city'] = df['city'].replace(city_mapping)
```

### Noise Reduction
**Noise** refers to random errors or variance in data that obscures the underlying pattern. This often manifests as outliers or high-frequency fluctuations in time-series data.

**Techniques:**
1. **Binning:** Smoothing data by consulting "neighborhood" values.
2. **Outlier Removal (IQR Method):**
   - Calculate $Q1$ (25th percentile) and $Q3$ (75th percentile).
   - $IQR = Q3 - Q1$.
   - Outliers are values outside $[Q1 - 1.5 \times IQR, Q3 + 1.5 \times IQR]$.
3. **Smoothing:** Using moving averages for trend analysis.

**Python Implementation (Outlier Removal):**
```python
# Interquartile Range (IQR) method
Q1 = df['price'].quantile(0.25)
Q3 = df['price'].quantile(0.75)
IQR = Q3 - Q1

lower_bound = Q1 - 1.5 * IQR
upper_bound = Q3 + 1.5 * IQR

# Filter the data
df_no_noise = df[(df['price'] >= lower_bound) & (df['price'] <= upper_bound)]
```

## Interview Questions

**Q: What is the impact of not removing duplicates before training an ML model?**
**A:** Duplicates can lead to **overfitting** because the model sees the same information multiple times and may treat it as a more significant pattern than it actually is. It also skews evaluation metrics if duplicates are present in both training and test sets.

**Q: How do you handle "Noise" in a numerical dataset?**
**A:** Noise can be handled via **binning** (grouping values into ranges), **regression** (fitting a function to the data), or **outlier detection** (using Z-scores or IQR) to remove or cap extreme values that don't represent the general trend.

**Q: Describe a strategy to clean inconsistent categorical data when you have thousands of unique entries.**
**A:** For large-scale inconsistencies, manual mapping is infeasible. Instead, use **Fuzzy Matching** (e.g., Levenshtein distance) to group similar strings or **Clustering** algorithms to identify groups of strings that likely represent the same entity.

**Q: When should you NOT remove an outlier?**
**A:** You should keep outliers if they represent **genuine, albeit rare, phenomena** that the model needs to learn (e.g., fraud detection, rare disease diagnosis). Only remove outliers that are clearly the result of measurement error or data corruption.
