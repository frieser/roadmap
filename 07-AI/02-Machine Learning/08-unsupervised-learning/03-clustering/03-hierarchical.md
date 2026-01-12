---
tags: ['ai', 'roadmap']
---

## Summary

**Hierarchical Clustering**, specifically the **Agglomerative** (bottom-up) approach, is an unsupervised learning algorithm that builds a hierarchy of clusters. It starts by treating each data point as a single cluster and then successively merges the most similar pairs of clusters until all points belong to one giant cluster (or until a stopping criterion is met). The result is typically visualized using a **Dendrogram**, a tree-like diagram that records the sequences of merges and the distances at which they occurred.

## Detailed Explanation

### 1. Agglomerative (Bottom-Up) Process
1.  **Initialize**: Every data point is its own cluster ($N$ clusters for $N$ points).
2.  **Compute Distances**: Calculate the proximity matrix (distance between all clusters).
3.  **Merge**: Combine the two "closest" clusters into a single cluster.
4.  **Update**: Recalculate the distance between the new cluster and all remaining clusters.
5.  **Repeat**: Continue steps 3 and 4 until only one cluster remains.

### 2. Linkage Criteria
The definition of "closest" depends on the **linkage criterion** used:

*   **Single Linkage (Nearest Neighbor)**: Distance between the two closest points in different clusters.
    *   *Pros*: Can handle non-elliptical shapes.
    *   *Cons*: Prone to **chaining**, where clusters are joined by a single "bridge" of points.
*   **Complete Linkage (Farthest Neighbor)**: Distance between the two most distant points in different clusters.
    *   *Pros*: Produces compact, similarly sized clusters.
    *   *Cons*: Sensitive to outliers.
*   **Average Linkage (UPGMA)**: Average distance between all pairs of points in two clusters.
*   **Ward's Method**: Minimizes the increase in total **within-cluster variance** after merging. 
    *   *Characteristics*: Tends to create clusters of relatively equal size and spherical shape. It is often the default and most robust choice.

### 3. The Dendrogram
A **Dendrogram** is a visual tree representing the clustering process:
*   **Leaves**: Individual data points.
*   **Vertical Height**: Represents the distance (dissimilarity) between merged clusters.
*   **Cutting the tree**: You can choose the number of clusters by drawing a horizontal line across the dendrogram at a specific height.

### 4. Python Implementation

Using `scikit-learn` for the model and `scipy` for visualization:

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.cluster import AgglomerativeClustering
from scipy.cluster.hierarchy import dendrogram, linkage

# Sample Data
X = np.array([[1, 2], [1, 4], [1, 0],
              [4, 2], [4, 4], [4, 0]])

# 1. Scikit-learn Implementation
# 'ward' is default; 'euclidean' is the default metric
model = AgglomerativeClustering(n_clusters=2, linkage='ward')
labels = model.fit_predict(X)

# 2. Visualization with Scipy (Dendrogram)
linked = linkage(X, 'ward')

plt.figure(figsize=(10, 7))
dendrogram(linked,
            orientation='top',
            labels=range(1, 7),
            distance_sort='descending',
            show_leaf_counts=True)
plt.title("Hierarchical Clustering Dendrogram")
plt.show()
```

## Interview Questions

### 1. How does Hierarchical Clustering differ from K-Means?
K-Means requires specifying the number of clusters ($K$) upfront and produces a flat partition. Hierarchical clustering does not require $K$ in advance and produces a multi-level tree structure (hierarchy), allowing the user to decide the number of clusters after inspecting the results. However, Hierarchical clustering is generally more computationally expensive ($O(N^3)$ or $O(N^2 \log N)$) than K-Means.

### 2. What is the "chaining effect" in Single Linkage?
The chaining effect occurs when clusters are merged because of a few points that happen to be close to each other, even if the majority of the points in the two clusters are very far apart. This results in long, elongated clusters rather than compact groups.

### 3. How do you determine the optimal number of clusters from a dendrogram?
The common heuristic is to look for the largest vertical distance that does not intersect any horizontal merge line. Drawing a horizontal line through this "gap" gives the number of clusters corresponding to the vertical lines it intersects.

### 4. Why is Ward's Method popular?
Ward's method is popular because it behaves similarly to K-Means in its objective (minimizing variance) but provides the hierarchical structure. It tends to produce clusters that are more spherical and easier to interpret in many real-world applications.

### 5. Can Hierarchical Clustering handle outliers well?
It depends on the linkage. Single linkage is very sensitive to outliers as they can act as bridges between clusters. Complete linkage and Ward's method are generally more robust but can still be influenced if the outlier is significantly far from all other points.
