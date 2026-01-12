---
tags: ['ai', 'roadmap']
---

## Summary
K-Nearest Neighbors (KNN) is a simple, non-parametric, and lazy supervised learning algorithm used for both classification and regression. It works by identifying the $K$ closest data points to a query individual and assigning a label based on the majority vote (classification) or average value (regression). Because it stores the entire training dataset, its prediction phase is computationally expensive, making it best suited for smaller datasets.

## Detailed Explanation

### 1. The "Lazy Learner" Concept
Unlike "eager learners" (e.g., Logistic Regression, Decision Trees) that build a generalized model during the training phase, KNN is a **lazy learner**. It does not perform any training in the traditional sense; instead, it simply stores the training data and postpones all computation until a prediction is requested.
- **Pros**: Quick training phase, can easily adapt to new data.
- **Cons**: High memory usage and slow prediction time (inference is $O(n \cdot d)$ where $n$ is the number of samples and $d$ is the number of dimensions).

### 2. Distance Metrics
The effectiveness of KNN depends heavily on how "closeness" is measured. Common metrics include:
- **Euclidean Distance**: The "straight-line" distance between two points. Most common for continuous variables.
  $$d(x, y) = \sqrt{\sum_{i=1}^n (x_i - y_i)^2}$$
- **Manhattan Distance**: The sum of absolute differences. Useful when data has discrete or binary attributes (grid-like movement).
  $$d(x, y) = \sum_{i=1}^n |x_i - y_i|$$
- **Minkowski Distance**: A generalized form that includes both Euclidean and Manhattan.
  $$d(x, y) = \left(\sum_{i=1}^n |x_i - y_i|^p\right)^{1/p}$$
  (where $p=1$ is Manhattan and $p=2$ is Euclidean).

### 3. Choosing the Optimal K
Selecting the right value for $K$ is a critical hyperparameter tuning step:
- **$K=1$**: The model will capture noise and outliers, leading to **overfitting**.
- **Large $K$**: The decision boundary becomes smoother, but it may lose local patterns, leading to **underfitting**.
- **Rule of Thumb**: A common starting point is $K = \sqrt{n}$, where $n$ is the number of samples.
- **Tie-breaking**: It is common to choose an **odd number** for $K$ in binary classification to avoid ties.

### 4. Feature Scaling
Since KNN relies on distance, features with larger scales (e.g., Salary in thousands vs. Age in years) will dominate the distance calculation. **Normalization** (Min-Max) or **Standardization** (Z-score) is mandatory before applying KNN to ensure all features contribute equally.

### Python Implementation (Scikit-Learn)
```python
from sklearn.neighbors import KNeighborsClassifier
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.datasets import load_iris

# 1. Load data
iris = load_iris()
X, y = iris.data, iris.target

# 2. Split into training and testing sets
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# 3. Feature Scaling (CRITICAL for KNN)
scaler = StandardScaler()
X_train = scaler.fit_transform(X_train)
X_test = scaler.transform(X_test)

# 4. Initialize and fit KNN
# Using K=5 and Euclidean distance (p=2)
knn = KNeighborsClassifier(n_neighbors=5, metric='minkowski', p=2)
knn.fit(X_train, y_train)

# 5. Predict and Evaluate
accuracy = knn.score(X_test, y_test)
print(f"Model Accuracy: {accuracy * 100:.2f}%")

# Predict a new sample
new_sample = [[5.1, 3.5, 1.4, 0.2]]
new_sample_scaled = scaler.transform(new_sample)
prediction = knn.predict(new_sample_scaled)
print(f"Prediction for sample {new_sample}: {iris.target_names[prediction][0]}")
```

## Interview Questions

**Q: Why is KNN called a non-parametric algorithm?**
**A:** It is called non-parametric because it doesn't make any underlying assumptions about the distribution of the data. It doesn't try to find a specific mathematical function (like a line or a plane) that fits the data; instead, the model structure is determined by the training data itself.

**Q: What is the impact of outliers on KNN?**
**A:** KNN is sensitive to outliers, especially when $K$ is small (e.g., $K=1$). Since it relies on the nearest neighbors, an outlier close to a query point can easily skew the classification. Increasing $K$ can help mitigate this by averaging out the noise, but data cleaning is usually the best approach.

**Q: How do you handle categorical features in KNN?**
**A:** KNN requires numerical input for distance calculations. Categorical features must be converted using One-Hot Encoding or Label Encoding. For binary features, Manhattan distance is often more appropriate than Euclidean.

**Q: How does the "Curse of Dimensionality" affect KNN?**
**A:** As the number of dimensions (features) increases, the volume of the space grows exponentially, making the data points sparse. In high-dimensional space, all points become nearly equidistant from each other, which degrades the concept of "nearest neighbor." Dimensionality reduction (like PCA) is often used before KNN to combat this.
