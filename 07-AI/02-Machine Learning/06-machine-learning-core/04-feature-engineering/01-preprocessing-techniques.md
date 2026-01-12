---
tags: ['ai', 'ml', 'roadmap']
---

# Data Preprocessing Techniques

## Summary
Data preprocessing is a critical step in the machine learning pipeline that involves transforming raw, messy data into a clean, structured format suitable for modeling. Since most real-world data is incomplete, inconsistent, or noisy, preprocessing ensures that models can effectively learn patterns without being misled by errors or scale differences. Key operations include cleaning, transformation, dimensionality reduction, and handling missing values or outliers.

## Detailed Explanation

### 1. Data Cleaning
Data cleaning involves identifying and correcting errors, inconsistencies, and noise in the dataset.
*   **Noise Reduction**: Smoothing data to remove random variations (e.g., using moving averages).
*   **Deduplication**: Identifying and removing duplicate records.
*   **Consistency Checks**: Ensuring data types and units are consistent across the dataset.

### 2. Data Transformation
Transformation adapts data into a range or format that algorithms can process efficiently.

#### **Scaling and Normalization**
*   **Standardization (Z-score Scaling)**: Rescales data to have a mean of 0 and a standard deviation of 1.
*   **Normalization (Min-Max Scaling)**: Rescales data to a specific range, usually [0, 1].

```python
import pandas as pd
from sklearn.preprocessing import StandardScaler, MinMaxScaler

data = pd.DataFrame({'age': [25, 30, 35, 40, 45], 'salary': [50000, 60000, 70000, 80000, 150000]})

# Standardization
scaler = StandardScaler()
data['std_salary'] = scaler.fit_transform(data[['salary']])

# Min-Max Scaling
min_max = MinMaxScaler()
data['norm_age'] = min_max.fit_transform(data[['age']])
```

#### **Encoding Categorical Variables**
*   **One-Hot Encoding**: Creates binary columns for each category (good for nominal data).
*   **Ordinal Encoding**: Assigns integers to categories (good for ordered data).

```python
from sklearn.preprocessing import OneHotEncoder, OrdinalEncoder

df = pd.DataFrame({'color': ['red', 'blue', 'green'], 'size': ['S', 'M', 'L']})

# One-Hot Encoding
ohe = OneHotEncoder(sparse_output=False)
colors_encoded = ohe.fit_transform(df[['color']])

# Ordinal Encoding
oe = OrdinalEncoder(categories=[['S', 'M', 'L']])
df['size_encoded'] = oe.fit_transform(df[['size']])
```

### 3. Handling Missing Values
Missing data can lead to biased models or errors during training.
*   **Deletion**: Removing rows (listwise) or columns with missing values.
*   **Imputation**: Filling gaps with statistical measures (Mean, Median, Mode) or using algorithms like KNN.

```python
import numpy as np
from sklearn.impute import SimpleImputer

X = np.array([[1, 2], [np.nan, 3], [7, 6]])

# Mean Imputation
imputer = SimpleImputer(strategy='mean')
X_imputed = imputer.fit_transform(X)
```

### 4. Handling Outliers
Outliers are extreme values that deviate significantly from the rest of the data.
*   **Detection**:
    *   **Z-score**: Points with |Z| > 3.
    *   **IQR (Interquartile Range)**: Points outside [Q1 - 1.5 * IQR, Q3 + 1.5 * IQR].
*   **Treatment**: Capping (Winsorizing), trimming, or transforming (log scale).

```python
# IQR Detection and Trimming
Q1 = data['salary'].quantile(0.25)
Q3 = data['salary'].quantile(0.75)
IQR = Q3 - Q1

lower_bound = Q1 - 1.5 * IQR
upper_bound = Q3 + 1.5 * IQR

trimmed_data = data[(data['salary'] >= lower_bound) & (data['salary'] <= upper_bound)]
```

### 5. Data Reduction
Reduces the volume of data while maintaining its integrity.
*   **Feature Selection**: Selecting the most relevant features (e.g., recursive feature elimination).
*   **Dimensionality Reduction**: Techniques like **Principal Component Analysis (PCA)** to project data into a lower-dimensional space.

```python
from sklearn.decomposition import PCA

pca = PCA(n_components=1)
reduced_features = pca.fit_transform(data[['age', 'salary']])
```

## Interview Questions

**Q: What is the difference between Normalization and Standardization?**
**A:** Normalization (Min-Max) scales data into a fixed range (usually 0 to 1), making it sensitive to outliers. Standardization (Z-score) centers data around a mean of 0 with a unit standard deviation; it is more robust to outliers and assumes a Gaussian distribution.

**Q: When should you use One-Hot Encoding versus Label/Ordinal Encoding?**
**A:** Use One-Hot Encoding for nominal data (no inherent order, like colors) to prevent the model from assuming a mathematical relationship between categories. Use Ordinal Encoding for data with a clear rank (like "Small", "Medium", "Large").

**Q: How does the presence of outliers affect Machine Learning models?**
**A:** Outliers can skew statistical measures (like mean and variance), leading to biased parameter estimates in linear models (Linear/Logistic Regression). They can also increase training time or cause poor generalization in algorithms like K-Means clustering.

**Q: Why is it important to perform feature scaling before applying PCA?**
**A:** PCA seeks to maximize variance. If features are on different scales (e.g., age in years vs. income in dollars), PCA will be dominated by the feature with the largest numerical range, regardless of its actual importance to the variance.

**Q: What is "Data Imputation" and what are the risks?**
**A:** Data Imputation is the process of replacing missing data with substituted values. While it prevents data loss, the risk is introducing bias if the "missingness" is not random, or underestimating the uncertainty/variance of the dataset by using simple averages.
