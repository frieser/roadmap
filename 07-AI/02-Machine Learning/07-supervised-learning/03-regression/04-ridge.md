---
tags: ['ai', 'roadmap']
---

## Summary
Ridge Regression, also known as **L2 Regularization**, is a technique used to analyze multiple regression data that suffer from multicollinearity. It introduces a small amount of bias to the regression estimates by adding a penalty term proportional to the square of the magnitude of coefficients. This helps to reduce the variance of the model and prevents overfitting, making it more robust to noise and highly correlated features.

## Detailed Explanation

### The L2 Penalty
Standard Ordinary Least Squares (OLS) regression aims to minimize the Residual Sum of Squares (RSS). However, when features are highly correlated or the number of features is large, the coefficients can become very large and sensitive to small changes in the training data.

Ridge regression modifies the OLS cost function by adding a penalty term:

$$J(\theta) = \text{RSS} + \alpha \sum_{j=1}^{p} \beta_j^2$$

Where:
- **RSS**: Residual Sum of Squares (the standard error term).
- **$\alpha$ (alpha)**: The regularization parameter (sometimes denoted as $\lambda$). It controls the trade-off between fitting the data well and keeping the weights small.
- **$\sum \beta_j^2$**: The L2 norm of the coefficient vector (excluding the intercept).

### Handling Multicollinearity
Multicollinearity occurs when independent variables are highly correlated. In such cases, the OLS estimates have high variance, meaning the model is very unstable. 
- By adding the L2 penalty, Ridge regression "shrinks" the coefficients towards zero (but never exactly zero).
- This shrinkage reduces the impact of correlated features and prevents the model from relying too heavily on any single feature, effectively stabilizing the estimates.

### Importance of Scaling
Since Ridge regression penalizes the magnitude of the coefficients, it is **crucial** to scale the features (e.g., using Standardization) before fitting the model. Otherwise, features with larger scales would be penalized more than those with smaller scales, regardless of their actual importance.

### Python Implementation
Using `scikit-learn`, Ridge regression is straightforward to implement:

```python
import numpy as np
from sklearn.linear_model import Ridge
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_squared_error

# Sample data
X = np.array([[1, 2], [2, 3], [3, 4], [4, 5], [5, 6]])
y = np.array([2, 4, 6, 8, 10])

# 1. Scale features
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# 2. Split data
X_train, X_test, y_train, y_test = train_test_split(X_scaled, y, test_size=0.2, random_state=42)

# 3. Fit Ridge Regression (alpha=1.0 is default)
ridge_reg = Ridge(alpha=1.0)
ridge_reg.fit(X_train, y_train)

# 4. Predict and Evaluate
predictions = ridge_reg.predict(X_test)
print(f"Coefficients: {ridge_reg.coef_}")
print(f"MSE: {mean_squared_error(y_test, predictions)}")
```

## Interview Questions

### Q: What is the main difference between Ridge and Lasso regression?
**A:** The primary difference is the type of penalty. Ridge uses **L2 regularization** (sum of squared coefficients), which shrinks coefficients towards zero but rarely makes them exactly zero. Lasso uses **L1 regularization** (sum of absolute values), which can shrink some coefficients to exactly zero, performing automatic feature selection.

### Q: How does the alpha parameter affect the model?
**A:** As $\alpha$ increases, the penalty for large coefficients grows, causing the coefficients to shrink more towards zero, which increases bias but reduces variance (simplifies the model). When $\alpha = 0$, Ridge regression becomes equivalent to standard OLS regression.

### Q: Why is Ridge regression better than OLS when multicollinearity is present?
**A:** In the presence of multicollinearity, the OLS solution matrix $(X^T X)$ becomes nearly singular, leading to extremely large and unstable coefficients. Ridge regression adds a positive value $(\alpha I)$ to this matrix before inversion, ensuring it is non-singular and providing more stable, lower-variance estimates.

### Q: Does Ridge regression perform feature selection?
**A:** No. Unlike Lasso, Ridge regression does not set coefficients to zero. It keeps all features in the model but reduces their influence by shrinking their weights. If feature selection is required, Lasso or Elastic Net would be better choices.

### Q: When would you use Ridge regression over Lasso?
**A:** Ridge is generally preferred when you expect most features to have at least a small effect on the outcome, or when you have many features that are highly correlated with each other. Lasso is better when you suspect only a few features are actually relevant.
