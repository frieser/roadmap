---
tags: ['ai', 'roadmap']
---

## Summary
Lasso (Least Absolute Shrinkage and Selection Operator) is a linear regression technique that uses **L1 regularization**. It adds a penalty proportional to the sum of the absolute values of the coefficients to the loss function. This not only prevents overfitting by penalizing large coefficients but also performs **automatic feature selection** by shrinking less important feature coefficients to exactly zero.

## Detailed Explanation

### 1. The L1 Penalty
Lasso Regression modifies the standard Ordinary Least Squares (OLS) objective function by adding an L1 regularization term:

$$Cost = \text{RSS} + \alpha \sum_{j=1}^{p} |w_j|$$

Where:
- **RSS** (Residual Sum of Squares) is the standard OLS loss ($\sum (y_i - \hat{y}_i)^2$).
- $\alpha$ (alpha) is the regularization strength (hyperparameter).
- $w_j$ are the feature coefficients.

### 2. Feature Selection Property
The defining characteristic of Lasso is its ability to produce **sparse models**. 

In high-dimensional datasets where many features are irrelevant or redundant, Lasso's geometric constraint (an L1-ball or "diamond" shape in 2D) tends to intersect the RSS contours at the axes. This mathematical property forces the coefficients of less important features to become exactly zero, effectively removing them from the model. This makes Lasso an embedded method for feature selection.

### 3. Ridge vs. Lasso
- **Ridge (L2)**: Penalizes the square of coefficients ($w^2$). It shrinks coefficients toward zero but never makes them exactly zero. It is better when most features are useful or when features are highly correlated.
- **Lasso (L1)**: Penalizes the absolute value of coefficients ($|w|$). It can zero out coefficients. It is better when only a few features are truly influential (sparsity) or when you need a simpler, more interpretable model.

### 4. Python Implementation

Regularization is sensitive to the scale of features, so **Standardization** is mandatory.

```python
import numpy as np
from sklearn.linear_model import Lasso
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_squared_error

# 1. Generate synthetic data
np.random.seed(42)
X = np.random.rand(100, 10)  # 100 samples, 10 features
# Only the first 2 features are actually relevant
y = 5 * X[:, 0] + 3 * X[:, 1] + np.random.randn(100) * 0.1 

# 2. Split and Scale
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

# 3. Initialize and fit Lasso
# alpha controls the penalty strength
lasso = Lasso(alpha=0.1)
lasso.fit(X_train_scaled, y_train)

# 4. Evaluate
y_pred = lasso.predict(X_test_scaled)
print(f"MSE: {mean_squared_error(y_test, y_pred):.4f}")

# 5. Inspect Coefficients
for i, coef in enumerate(lasso.coef_):
    print(f"Feature {i} coefficient: {coef:.4f}")

# Note: Features 2-9 will likely have coefficients close or equal to zero.
```

## Interview Questions

### Q: Why does Lasso Regression perform feature selection while Ridge does not?
**A:** This is due to the geometry of the constraint. Lasso uses an L1 penalty $\sum |w|$, which has a "pointy" diamond shape in the parameter space. When minimizing the loss, the optimal solution is mathematically more likely to occur at a vertex of this diamond, where one or more coordinates (coefficients) are exactly zero. Ridge uses an L2 penalty $\sum w^2$, which is circular, so the solution rarely hits an axis exactly.

### Q: What is the impact of the $\alpha$ parameter in Lasso?
**A:** $\alpha$ is the hyperparameter that controls the trade-off between fitting the data and keeping the model simple. 
- If **$\alpha = 0$**, it becomes standard OLS regression (no regularization).
- As **$\alpha$ increases**, the penalty for large coefficients grows, pushing more of them to exactly zero. This increases bias but decreases the variance of the model.
- If **$\alpha$ is too high**, the model may underfit, eventually setting all coefficients to zero.

### Q: When is Lasso Regression preferred over Ridge Regression?
**A:** Lasso is preferred when you suspect that only a subset of your features are actually useful (sparsity). It simplifies the model and improves interpretability by performing automatic feature selection. Ridge is generally better when you have many features that all contribute small effects, or when features are highly collinear (Lasso might arbitrarily pick one from a group of correlated features).

### Q: Is it necessary to scale features before applying Lasso?
**A:** **Yes.** Regularization penalties are applied to the absolute magnitude of coefficients. If features are on different scales (e.g., one in millions, another in decimals), the penalty will unfairly suppress features with larger numerical ranges regardless of their actual predictive power. Standardization ensures that the penalty is applied uniformly across all features.
