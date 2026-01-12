---
tags: ['ai', 'roadmap']
---

# Data Preparation with Scikit-Learn

## Summary
Data preparation is the process of cleaning and transforming raw data into a format suitable for machine learning models. In Scikit-Learn, this is handled through a consistent API of **Transformers** (`fit`, `transform`, `fit_transform`). Proper preparation prevents common pitfalls like data leakage, handles missing information, ensures numerical stability through scaling, and converts categorical data into model-readable formats. The ultimate goal is to build a robust **Pipeline** that automates these steps for both training and production inference.

## Detailed Explanation

### 1. Handling Missing Values (Imputation)
Real-world datasets often have missing entries. Scikit-Learn provides several strategies to fill these gaps without losing entire rows of data.

*   **`SimpleImputer`**: Fills missing values using basic statistics (mean, median, most frequent) or a constant value.
*   **`KNNImputer`**: Uses the k-Nearest Neighbors approach to fill missing values based on similar samples.
*   **`IterativeImputer`**: (Experimental/Multivariate) Models each feature with missing values as a function of other features.

```python
from sklearn.impute import SimpleImputer
import numpy as np

# Sample data with NaN
X = [[1, 2], [np.nan, 3], [7, 6]]

# Strategy: Mean
imputer = SimpleImputer(strategy='mean')
X_imputed = imputer.fit_transform(X)
```

### 2. Categorical Encoding
Machine learning models require numerical input. Categorical variables must be encoded.

*   **`OneHotEncoder`**: Creates binary columns for each category (dummy variables). Essential for nominal data where no order exists.
*   **`OrdinalEncoder`**: Assigns integers to categories. Best for ordinal data where order matters (e.g., "Low", "Medium", "High").
*   **`LabelEncoder`**: Typically used for encoding target labels (`y`), not features (`X`).

```python
from sklearn.preprocessing import OneHotEncoder, OrdinalEncoder

# Nominal data -> OneHot
ohe = OneHotEncoder(sparse_output=False, handle_unknown='ignore')
encoded = ohe.fit_transform([['Red'], ['Green'], ['Blue']])

# Ordinal data -> Ordinal
oe = OrdinalEncoder(categories=[['Low', 'Medium', 'High']])
ranks = oe.fit_transform([['Medium'], ['High'], ['Low']])
```

### 3. Feature Scaling
Algorithms that rely on distance metrics (like KNN, SVM, K-Means) or gradient descent (Linear Regression, Neural Nets) are sensitive to the scale of features.

*   **`StandardScaler`**: Centers data around 0 with unit variance ($z = (x - \mu) / \sigma$). Best when features follow a Gaussian distribution.
*   **`MinMaxScaler`**: Scales data to a specific range, usually [0, 1]. Preserves the shape of the distribution but is sensitive to outliers.
*   **`RobustScaler`**: Uses median and interquartile range (IQR). Excellent for data with significant outliers.

```python
from sklearn.preprocessing import StandardScaler, MinMaxScaler

data = [[-1, 2], [-0.5, 6], [0, 10], [1, 18]]

scaler = StandardScaler()
scaled_data = scaler.fit_transform(data)
```

### 4. Pipelines and ColumnTransformers
To prevent **Data Leakage** (using info from the test set during training), preprocessing should be bundled with the model.

*   **`ColumnTransformer`**: Applies different transformations to different columns (e.g., scale numeric, encode categorical).
*   **`Pipeline`**: Chains multiple steps into one object that behaves like a single estimator.

```python
from sklearn.compose import ColumnTransformer
from sklearn.pipeline import Pipeline
from sklearn.ensemble import RandomForestClassifier

# Define preprocessing for numerical and categorical tracks
preprocessor = ColumnTransformer(
    transformers=[
        ('num', StandardScaler(), ['age', 'fare']),
        ('cat', OneHotEncoder(), ['embarked', 'gender'])
    ])

# Create the full pipeline
clf = Pipeline(steps=[
    ('preprocessor', preprocessor),
    ('classifier', RandomForestClassifier())
])

# Fit the entire pipeline
# clf.fit(X_train, y_train)
```

## Interview Questions

**Q: What is Data Leakage and how do Scikit-Learn Pipelines prevent it?**
**A:** Data leakage occurs when information from outside the training dataset (like the mean of the test set) is used to create the model. Pipelines prevent this by ensuring that `fit` is only called on the training data. The transformations (like scaling) then use the parameters learned from the training set to `transform` the test set, maintaining strict separation.

**Q: When would you prefer `RobustScaler` over `StandardScaler`?**
**A:** You should use `RobustScaler` when the dataset contains significant outliers. `StandardScaler` uses the mean and standard deviation, which are highly influenced by extreme values, potentially squashing the majority of the data. `RobustScaler` uses the median and IQR, which are much more resistant to outliers.

**Q: What is the "Dummy Variable Trap" and how does Scikit-Learn handle it?**
**A:** The Dummy Variable Trap is a scenario where independent variables are multicollinear (one can be predicted from others), which can break some models like Linear Regression. You can handle this by dropping one category (using `drop='first'` in `OneHotEncoder`). However, for tree-based models (Random Forest, XGBoost), dropping a column is usually unnecessary and can even be detrimental.

**Q: What is the difference between `fit`, `transform`, and `fit_transform`?**
**A:** 
- `fit`: Calculates the parameters (e.g., mean/std for scaling) from the data.
- `transform`: Applies the transformation using the previously calculated parameters.
- `fit_transform`: Does both in one step. It is used on training data for efficiency, but only `transform` should be used on test/validation data.
