---
tags: ['ai', 'roadmap']
---

## Summary
**Unsupervised Learning** is a branch of machine learning that deals with finding hidden patterns and structures in data that has not been labeled, categorized, or classified. Unlike supervised learning, there is no "correct" answer provided during training; the model learns to identify inherent relationships within the input data itself.

## Detailed Explanation

### Unlabeled Data
In unsupervised learning, the input data consists of examples without corresponding labels. The goal is not to predict a target value, but to explore the data and find some structure. This is particularly useful for large datasets where manual labeling is too expensive or impractical.

### Finding Patterns
The algorithm analyzes the data to discover interesting properties or relationships. These patterns can be used for:
- **Grouping**: Finding natural segments in the data.
- **Association**: Finding rules that describe large portions of your data (e.g., people that buy X also tend to buy Y).
- **Anomaly Detection**: Identifying data points that are significantly different from the rest.

### Clustering vs. Dimensionality Reduction
These are the two most common tasks in unsupervised learning:

#### 1. Clustering
Clustering involves grouping data points into sets (clusters) based on their similarities. Points within a cluster should be more similar to each other than to points in other clusters.
- **Use Cases**: Customer segmentation, image compression, document grouping.
- **Common Algorithms**: K-Means, Hierarchical Clustering, DBSCAN, Gaussian Mixture Models.

#### 2. Dimensionality Reduction
Dimensionality reduction is the process of reducing the number of random variables under consideration by obtaining a set of principal variables. It aims to simplify the data without losing its essential information.
- **Use Cases**: Data visualization, noise filtering, feature extraction, speeding up other ML algorithms.
- **Common Algorithms**: Principal Component Analysis (PCA), t-SNE, Singular Value Decomposition (SVD), Autoencoders.

**Key Difference**: Clustering seeks to find "types" or "groups" of data points, while dimensionality reduction seeks to find a more efficient "representation" or "view" of the entire dataset.

## Interview Questions

### 1. What is the fundamental difference between supervised and unsupervised learning?
The main difference is the presence of labels. **Supervised learning** trains on input-output pairs to learn a mapping, while **unsupervised learning** works with unlabeled data to find inherent structures without explicit guidance.

### 2. When would you use unsupervised learning instead of supervised learning?
You use unsupervised learning when you don't have labeled data, or when your goal is to explore the data to find patterns, segments, or outliers rather than predicting a specific target.

### 3. How do you know if an unsupervised learning model is performing well?
Since there are no ground-truth labels, evaluation is often indirect or qualitative. For clustering, you can use metrics like the **Silhouette Score** or the **Elbow Method**. For dimensionality reduction, you can measure the **proportion of variance explained**. Often, the final "test" is whether the results provide actionable insights for a downstream task.

### 4. What is an example of "Association" in unsupervised learning?
A classic example is **Market Basket Analysis**, where a retailer discovers that customers who buy diapers are also likely to buy beer. This helps in store layout and cross-selling strategies.

### 5. Can Unsupervised Learning be used for Supervised Learning tasks?
Yes. Techniques like **Pre-training** (using an unsupervised model to learn features before training a supervised model) or **Feature Extraction** (using dimensionality reduction to create better inputs for a classifier) are common in deep learning and traditional ML pipelines.
