---
tags: ['ai', 'roadmap']
---

## Summary
**Polynomial Regression** is a form of regression analysis in which the relationship between the independent variable $x$ and the dependent variable $y$ is modeled as an $n$-th degree polynomial. It is used when the data shows a non-linear relationship that a simple straight line cannot capture.

## Detailed Explanation

### Non-linear Relationships
While standard Linear Regression assumes a straight-line relationship ($y = mx + b$), many real-world phenomena are curved. For example:
- The growth rate of a population over time.
- The relationship between engine speed and fuel consumption.
- Yield of a chemical reaction based on temperature.

### How it Works
Polynomial regression "curves" the line by adding higher-order terms of the predictors as new features. For a single feature $x$, the model becomes:
$$y = w_0 + w_1x + w_2x^2 + w_3x^3 + \dots + w_dx^d + \epsilon$$

Even though the relationship between $x$ and $y$ is non-linear, the model is still technically **linear** because it is linear in terms of the coefficients ($w_0, w_1, \dots$). We are essentially performing multiple linear regression on a transformed feature set.

### Degree of Polynomial
The **degree** ($d$) determines the flexibility of the model:
- **$d=1$**: Simple linear regression (Underfitting if data is curved).
- **$d=2$**: Quadratic (Parabola).
- **$d=3$**: Cubic.
- **High $d$**: Can fit complex shapes but risks **Overfitting**.

### Overfitting Risk
As the degree of the polynomial increases, the model becomes more complex and can follow the noise in the training data too closely. This leads to:
- **Low Bias**: High accuracy on training data.
- **High Variance**: Poor generalization to unseen data.

To find the optimal degree, techniques like **Cross-Validation** or **Learning Curves** are used to balance the bias-variance tradeoff.

### Python Implementation
In `scikit-learn`, we use `PolynomialFeatures` to transform our data before fitting it with `LinearRegression`.

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.preprocessing import PolynomialFeatures
from sklearn.linear_model import LinearRegression
from sklearn.pipeline import Pipeline

# Generate some non-linear data
X = np.sort(np.random.rand(40, 1) * 5, axis=0)
y = np.sin(X).ravel() + np.random.normal(0, 0.1, X.shape[0])

# Create a pipeline: Polynomial Features -> Linear Regression
degree = 3
model = Pipeline([
    ("poly_features", PolynomialFeatures(degree=degree)),
    ("linear_regression", LinearRegression())
])

model.fit(X, y)

# Predict
X_test = np.linspace(0, 5, 100)[:, np.newaxis]
y_pred = model.predict(X_test)

plt.scatter(X, y, color='blue', label='Actual Data')
plt.plot(X_test, y_pred, color='red', label=f'Polynomial Degree {degree}')
plt.legend()
plt.show()
```

## Interview Questions

1. **Is Polynomial Regression a linear or non-linear model?**
   It is a **linear model** in terms of its parameters ($w$). Although it models non-linear relationships in the data, the mathematical optimization problem (finding the weights) remains linear.

2. **What is the main drawback of high-degree Polynomial Regression?**
   **Overfitting**. High-degree polynomials are extremely sensitive to outliers and noise, which leads to high variance and poor performance on new data.

3. **How do you choose the best degree for a polynomial model?**
   The best approach is to use **Cross-Validation**. By testing different degrees on validation sets, you can identify the degree that minimizes error without overfitting. You can also use **Regularization** (Ridge or Lasso) to penalize large coefficients in high-degree models.

4. **Why is feature scaling important for Polynomial Regression?**
   When you create polynomial features (e.g., $x, x^2, x^3$), the range of values grows exponentially. If $x=100$, then $x^3=1,000,000$. This huge difference in scale can cause numerical issues and make the model unstable if features aren't normalized or standardized.
