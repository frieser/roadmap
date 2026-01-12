---
tags: ['ai', 'roadmap']
---

## Summary
Dimensionality reduction is a feature engineering technique used to reduce the number of input variables in a dataset. By transforming high-dimensional data into a lower-dimensional space, it helps mitigate the "curse of dimensionality," reduces computational costs, simplifies models, and facilitates data visualization while retaining as much relevant information as possible.

## Detailed Explanation

### **Why Dimensionality Reduction?**
1. **The Curse of Dimensionality**: As the number of features increases, the volume of the space increases so fast that the available data becomes sparse, making it difficult for models to find patterns.
2. **Computational Efficiency**: Fewer features mean faster training and inference times.
3. **Overfitting Prevention**: Reducing noise and redundant features helps the model generalize better.
4. **Visualization**: Humans can only perceive up to 3 dimensions. Reducing data to 2D or 3D allows for visual exploratory data analysis.

---

### **1. Principal Component Analysis (PCA)**
PCA is an **unsupervised linear** technique that finds the directions (principal components) along which the variance of the data is maximized. It projects the data onto these components.

*   **Best for**: General noise reduction and feature extraction when labels are unavailable.
*   **Key Property**: The first principal component captures the most variance, the second captures the most remaining variance, and so on.

#### **Python Example (PCA)**
```python
from sklearn.decomposition import PCA
from sklearn.datasets import load_iris
import pandas as pd

# Load data
data = load_iris()
X = data.data

# Initialize PCA - reduce 4 features to 2
pca = PCA(n_components=2)
X_reduced = pca.fit_transform(X)

print(f"Original shape: {X.shape}")
print(f"Reduced shape: {X_reduced.shape}")
print(f"Explained variance ratio: {pca.explained_variance_ratio_}")
```

---

### **2. Linear Discriminant Analysis (LDA)**
LDA is a **supervised linear** technique. Unlike PCA, which focuses on variance, LDA focuses on maximizing the distance between different classes and minimizing the variance within each class.

*   **Best for**: Dimensionality reduction as a pre-processing step for classification tasks.
*   **Key Property**: It requires class labels to compute the transformation.

#### **Python Example (LDA)**
```python
from sklearn.discriminant_analysis import LinearDiscriminantAnalysis as LDA
from sklearn.datasets import load_iris

# Load data
data = load_iris()
X, y = data.data, data.target

# Initialize LDA - reduce to (n_classes - 1) dimensions
lda = LDA(n_components=2)
X_lda = lda.fit_transform(X, y)

print(f"LDA Reduced shape: {X_lda.shape}")
```

---

### **3. t-Distributed Stochastic Neighbor Embedding (t-SNE)**
t-SNE is a **non-linear, unsupervised** technique primarily used for **visualization**. It maps high-dimensional data into 2D or 3D such that similar points in high-dimensional space remain close in the low-dimensional map.

*   **Best for**: Visualizing clusters and non-linear manifolds.
*   **Key Property**: Preserves local structure but often distorts global relationships. It is computationally expensive.

#### **Python Example (t-SNE)**
```python
from sklearn.manifold import TSNE
from sklearn.datasets import load_digits

# Load complex data (64 features)
digits = load_digits()
X = digits.data

# Initialize t-SNE
tsne = TSNE(n_components=2, perplexity=30, n_iter=1000, random_state=42)
X_embedded = tsne.fit_transform(X)

print(f"t-SNE shape: {X_embedded.shape}")
```

---

### **Comparison Table**

| Technique | Type | Goal | Use Case |
| :--- | :--- | :--- | :--- |
| **PCA** | Unsupervised, Linear | Maximize Variance | Feature extraction, Noise reduction |
| **LDA** | Supervised, Linear | Maximize Class Separability | Pre-classification dimensionality reduction |
| **t-SNE** | Unsupervised, Non-linear | Preserve Local Structure | Visualization, Cluster discovery |

---

## Interview Questions

**Q: What is the main difference between PCA and LDA?**
**A:** PCA is an unsupervised technique that focuses on finding directions of maximum variance in the data, regardless of class labels. LDA is a supervised technique that seeks to find a feature space that maximizes the separability between known classes.

**Q: When would you prefer t-SNE over PCA?**
**A:** t-SNE is preferred when the data has non-linear relationships and the primary goal is visualization or discovering clusters. PCA is preferred for general dimensionality reduction, noise filtering, and as a pre-processing step for linear models because it is faster and preserves global structure better.

**Q: What is "Explained Variance Ratio" in PCA?**
**A:** It represents the proportion of the dataset's total variance that lies along each principal component. It helps in deciding how many components to keep (e.g., keeping enough components to cover 95% of the variance).

**Q: Does dimensionality reduction always improve model performance?**
**A:** Not necessarily. While it can prevent overfitting and reduce noise, it also involves losing some information. If too many dimensions are removed, the model might underfit. It should be validated using cross-validation.
