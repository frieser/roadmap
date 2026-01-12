---
tags: ['ai', 'roadmap']
---

## Summary
Singular Value Decomposition (SVD) is a powerful matrix factorization technique that decomposes any $m \times n$ matrix $A$ into three matrices: $A = U\Sigma V^T$. It generalizes the eigendecomposition to non-square matrices and provides a way to identify the most important directions (features) in data. In machine learning, SVD is the mathematical foundation for techniques like Principal Component Analysis (PCA), Latent Semantic Analysis (LSA), and various forms of data compression and noise reduction.

## Detailed Explanation

### Mathematical Definition
For any real $m \times n$ matrix $A$, the SVD is defined as:
$$A = U \Sigma V^T$$

Where:
- **$U$** ($m \times m$): An orthogonal matrix whose columns are the **left singular vectors**. These are the eigenvectors of $AA^T$.
- **$\Sigma$** ($m \times n$): A diagonal matrix containing the **singular values** $\sigma_i$, which are non-negative and usually sorted in descending order. These are the square roots of the eigenvalues of $A^T A$ (or $AA^T$).
- **$V^T$** ($n \times n$): The transpose of an orthogonal matrix $V$ whose columns are the **right singular vectors**. These are the eigenvectors of $A^T A$.

### Applications in Machine Learning

#### 1. Dimensionality Reduction (PCA)
Principal Component Analysis (PCA) is often implemented using SVD. If we center our data matrix $X$ (subtract the mean), the SVD of $X$ provides the principal components directly. The columns of $V$ are the principal directions, and the singular values in $\Sigma$ tell us how much variance is captured by each component.

#### 2. Image Compression
An image can be treated as a matrix of intensity values. By performing SVD and keeping only the top $k$ singular values (and their corresponding vectors), we can reconstruct a "low-rank approximation" of the image. This significantly reduces the storage size while maintaining the essential visual features.

#### 3. Noise Reduction
Since the smaller singular values often represent noise or less significant details, "truncating" the SVD (setting small $\sigma_i$ to zero) effectively filters out noise from the data.

### Python Implementation (NumPy)

The following example demonstrates how to perform SVD and use it for image compression using NumPy.

```python
import numpy as np
import matplotlib.pyplot as plt

# 1. Create a dummy "image" (100x100 matrix)
image = np.zeros((100, 100))
image[20:80, 20:80] = 1  # A simple white square on black background

# 2. Perform SVD
U, S, Vt = np.linalg.svd(image)

# 3. Reconstruct using only the top k singular values
def reconstruct(U, S, Vt, k):
    # S is returned as a 1D array of singular values
    S_k = np.diag(S[:k])
    U_k = U[:, :k]
    Vt_k = Vt[:k, :]
    return U_k @ S_k @ Vt_k

# Example reconstructions
image_k1 = reconstruct(U, S, Vt, 1)
image_k5 = reconstruct(U, S, Vt, 5)

print(f"Original shape: {image.shape}")
print(f"Singular values found: {len(S)}")
```

## Interview Questions

**Q: What is the geometric interpretation of SVD?**
**A:** SVD decomposes a linear transformation into three steps: a rotation ($V^T$), a scaling along the axes ($\Sigma$), and a second rotation ($U$). It essentially describes how a unit sphere in the input space is stretched into an ellipsoid in the output space.

**Q: Why is SVD preferred over eigendecomposition for certain tasks?**
**A:** Eigendecomposition only works for square matrices and requires them to be diagonalizable. SVD works for **any** $m \times n$ matrix, making it more robust for real-world data which is rarely square or perfectly behaved.

**Q: How do you choose the number of singular values to keep for compression?**
**A:** A common method is to look at the "Scree Plot" (plot of singular values) or calculate the cumulative energy/variance: $\sum_{i=1}^k \sigma_i^2 / \sum \sigma_i^2$. We usually keep enough components to capture 90-99% of the variance.

**Q: What is the relationship between SVD and the Moore-Penrose Pseudoinverse?**
**A:** SVD provides a stable way to calculate the pseudoinverse $A^+$. If $A = U \Sigma V^T$, then $A^+ = V \Sigma^+ U^T$, where $\Sigma^+$ is obtained by taking the reciprocal of each non-zero element on the diagonal of $\Sigma$ and transposing the matrix.
