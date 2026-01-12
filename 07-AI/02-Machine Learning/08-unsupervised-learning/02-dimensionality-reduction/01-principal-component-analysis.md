---
tags: ['ai', 'roadmap']
---

## Summary
**Principal Component Analysis (PCA)** is a powerful unsupervised machine learning technique used for **dimensionality reduction**. It transforms a large set of variables into a smaller one that still contains most of the information (variance) from the original set. By creating new, uncorrelated variables called **Principal Components**, PCA helps simplify data, reduce noise, and visualize high-dimensional datasets.

## Detailed Explanation

### 1. Core Concept
PCA aims to find the directions (Principal Components) where the data varies the most. These components are linear combinations of the original features and are orthogonal (perpendicular) to each other, ensuring they are uncorrelated.

### 2. Mathematical Steps
The process typically involves:
1.  **Standardization**: Scale features to have a mean of 0 and a variance of 1. This prevents features with larger scales from dominating the components.
2.  **Covariance Matrix**: Calculate the covariance between all pairs of features to understand how they vary together.
3.  **Eigen-decomposition**:
    -   **Eigenvectors**: Define the directions of the new feature space.
    -   **Eigenvalues**: Define the magnitude (amount of variance) along each eigenvector.
4.  **Feature Vector**: Sort eigenvalues in descending order and choose the top $k$ eigenvectors to form a transformation matrix.
5.  **Recasting**: Project the original data onto the new principal component axes.

### 3. Variance Preservation
The "Explained Variance Ratio" tells us how much information each principal component holds. Often, a "Scree Plot" is used to visualize this and decide how many components to keep (e.g., retaining 95% of total variance).

### 4. Python Implementation
Using `scikit-learn` is the standard way to implement PCA in Python.

```python
import numpy as np
import pandas as pd
from sklearn.decomposition import PCA
from sklearn.preprocessing import StandardScaler
import matplotlib.pyplot as plt

# 1. Generate/Load Data
# Creating a dummy dataset with 3 correlated features
np.random.seed(42)
x = np.random.rand(100)
data = pd.DataFrame({
    'Feature1': x + np.random.normal(0, 0.1, 100),
    'Feature2': x * 2 + np.random.normal(0, 0.1, 100),
    'Feature3': x * -0.5 + np.random.normal(0, 0.1, 100)
})

# 2. Standardization (Crucial Step)
scaler = StandardScaler()
scaled_data = scaler.fit_transform(data)

# 3. Apply PCA
pca = PCA(n_components=2) # Reduce from 3D to 2D
principal_components = pca.fit_transform(scaled_data)

# 4. Create DataFrame for results
pca_df = pd.DataFrame(data=principal_components, columns=['PC1', 'PC2'])

# 5. Analysis
print(f"Explained Variance Ratio: {pca.explained_variance_ratio_}")
print(f"Total Variance Explained: {sum(pca.explained_variance_ratio_):.2%}")

# Visualization
plt.scatter(pca_df['PC1'], pca_df['PC2'])
plt.xlabel('Principal Component 1')
plt.ylabel('Principal Component 2')
plt.title('PCA Projection')
plt.show()
```

## Interview Questions

### 1. Why do we need to normalize/standardize data before PCA?
PCA maximizes variance. If one feature has a range of [0, 1000] and another [0, 1], PCA will be biased towards the first feature because its numerical variance is much higher, even if it's less informative. Standardization ensures all features contribute equally.

### 2. What are Principal Components?
Principal Components are new, uncorrelated variables created as linear combinations of the original features. The first component (PC1) captures the maximum possible variance, and each subsequent component captures the remaining variance while being orthogonal to previous ones.

### 3. How do you decide the number of components to keep?
Common methods include:
- **Kaiser Criterion**: Keep components with eigenvalues $> 1$.
- **Scree Plot**: Look for the "elbow" where the explained variance starts to level off.
- **Cumulative Explained Variance**: Keep enough components to reach a threshold (e.g., 90% or 95%).

### 4. Is PCA a feature selection or feature extraction technique?
It is **feature extraction**. Unlike feature selection, which keeps a subset of original features, PCA creates *new* features that are combinations of the original ones.

### 5. What is the "Curse of Dimensionality"?
It refers to phenomena that arise when analyzing data in high-dimensional spaces. As dimensionality increases, the volume of the space increases so fast that the available data becomes sparse, making statistical significance harder to achieve and distance metrics less meaningful. PCA helps mitigate this.
