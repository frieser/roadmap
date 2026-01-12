---
tags: ['ai', 'roadmap']
---

## Summary
Descriptive statistics provide a concise summary of the characteristics of a dataset, focusing on describing the observed data rather than making inferences about a larger population. In the context of AI and Machine Learning, descriptive statistics are fundamental during the Exploratory Data Analysis (EDA) phase to understand data distributions, identify outliers, and inform feature engineering decisions.

## Detailed Explanation

### 1. Measures of Central Tendency
These metrics describe the "center" or typical value of a distribution.

*   **Mean**: The arithmetic average (sum of values divided by count). It is highly sensitive to outliers.
*   **Median**: The middle value in a sorted dataset. If the number of observations is even, it's the average of the two central values. It is robust to outliers.
*   **Mode**: The most frequent value in the dataset. It is particularly useful for categorical data.

### 2. Measures of Dispersion (Spread)
These metrics describe how spread out the data points are.

*   **Variance ($\sigma^2$)**: The average of the squared differences from the Mean. It quantifies how much the data varies.
*   **Standard Deviation ($\sigma$)**: The square root of the variance. It is easier to interpret as it is in the same units as the data itself.
*   **Interquartile Range (IQR)**: The difference between the 75th percentile (Q3) and the 25th percentile (Q1). It measures the spread of the middle 50% of the data and is used in box plots to identify outliers.

### 3. Shape of Distribution
These metrics describe the symmetry and "peakedness" of the data.

*   **Skewness**: Measures the asymmetry of the probability distribution.
    *   **Positive Skew (Right-skewed)**: The tail on the right side is longer. Mean > Median.
    *   **Negative Skew (Left-skewed)**: The tail on the left side is longer. Mean < Median.
*   **Kurtosis**: Measures the "tailedness" of the distribution.
    *   **Leptokurtic**: High kurtosis (sharp peak, heavy tails).
    *   **Platykurtic**: Low kurtosis (flat peak, thin tails).
    *   **Mesokurtic**: Normal distribution (kurtosis ≈ 3 or 0 depending on the definition).

### Python (Pandas) Implementation

```python
import pandas as pd
import numpy as np
from scipy.stats import skew, kurtosis

# Create a sample dataset
data = {
    'feature_a': [10, 12, 12, 13, 15, 18, 20, 25, 100], # Contains an outlier (100)
    'feature_b': [50, 51, 52, 50, 49, 51, 50, 52, 51]
}
df = pd.DataFrame(data)

# Basic Descriptive Statistics using Pandas
description = df.describe()
print("Pandas describe():\n", description)

# Specific Metrics
for col in df.columns:
    print(f"\n--- Statistics for {col} ---")
    print(f"Mean: {df[col].mean():.2f}")
    print(f"Median: {df[col].median():.2f}")
    print(f"Mode: {df[col].mode()[0]}")
    print(f"Variance: {df[col].var():.2f}")
    print(f"Std Dev: {df[col].std():.2f}")
    print(f"Skewness: {skew(df[col]):.2f}")
    print(f"Kurtosis: {kurtosis(df[col]):.2f}")

# Identifying Outliers using IQR
Q1 = df['feature_a'].quantile(0.25)
Q3 = df['feature_a'].quantile(0.75)
IQR = Q3 - Q1
lower_bound = Q1 - 1.5 * IQR
upper_bound = Q3 + 1.5 * IQR

outliers = df[(df['feature_a'] < lower_bound) | (df['feature_a'] > upper_bound)]
print(f"\nOutliers detected in feature_a:\n", outliers)
```

## Interview Questions

**Q: Why is the Median often preferred over the Mean in real-world datasets?**
**A:** The Mean is sensitive to outliers, which can pull the average in one direction and misrepresent the "typical" value (e.g., average income in a neighborhood with one billionaire). The Median is robust because it only depends on the relative ordering of values, making it a better measure of central tendency for skewed distributions.

**Q: What does a high Standard Deviation indicate about your data?**
**A:** A high standard deviation indicates that the data points are spread out over a wide range of values, suggesting high variability. Conversely, a low standard deviation means the data points tend to be close to the mean.

**Q: How do you detect outliers using descriptive statistics?**
**A:** Common methods include calculating Z-scores (checking if a value is more than 3 standard deviations from the mean) or using the Interquartile Range (IQR). In the IQR method, any value smaller than $Q1 - 1.5 \times IQR$ or larger than $Q3 + 1.5 \times IQR$ is typically flagged as an outlier.

**Q: If a distribution is right-skewed, where do the Mean and Median sit relative to each other?**
**A:** In a right-skewed (positively skewed) distribution, the tail is on the right. The Mean is typically greater than the Median because the high-value outliers in the right tail pull the Mean upward more than they affect the Median.
