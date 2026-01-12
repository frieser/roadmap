---
tags: ['ai', 'roadmap']
---

## Summary
Linear Regression is a fundamental supervised learning algorithm used to predict a continuous numerical output based on one or more input features. It establishes a linear relationship between the independent variables (features) and the dependent variable (target) by fitting a mathematical equation of the form $Y = \beta_0 + \beta_1X_1 + \dots + \beta_nX_n + \epsilon$. The primary goal is to find the coefficients ($\beta$) that minimize the error between the predicted and actual values.

## Detailed Explanation

### Simple vs Multiple Linear Regression
- **Simple Linear Regression**: Involves a single independent variable to predict the target. The model is a straight line: $Y = \beta_0 + \beta_1X + \epsilon$.
- **Multiple Linear Regression**: Extends the concept to multiple independent variables: $Y = \beta_0 + \sum_{i=1}^{n} \beta_iX_i + \epsilon$. This allows the model to capture more complex relationships by considering multiple factors simultaneously.

### Ordinary Least Squares (OLS)
OLS is the most common method for estimating the unknown parameters in a linear regression model. It works by minimizing the **Sum of Squared Residuals (SSR)**, where a residual is the difference between the observed value and the predicted value.
- **Objective Function**: $J(\beta) = \sum_{i=1}^{m} (y_i - \hat{y}_i)^2$
- **Mathematical Solution**: $\hat{\beta} = (X^T X)^{-1} X^T y$ (Normal Equation).

### Core Assumptions
For Linear Regression to provide reliable and unbiased results, several assumptions must hold:
1. **Linearity**: The relationship between features and target is linear.
2. **Independence**: Observations are independent of each other (no autocorrelation).
3. **Homoscedasticity**: The variance of residual errors is constant across all levels of the independent variables.
4. **Normality of Residuals**: For any fixed value of X, Y is normally distributed (important for hypothesis testing).
5. **No Multicollinearity**: Independent variables should not be highly correlated with each other.

### Evaluation Metrics
- **Mean Squared Error (MSE)**: Average of the squared differences between actual and predicted values.
- **Root Mean Squared Error (RMSE)**: Square root of MSE, providing error in the same units as the target variable.
- **R-squared ($R^2$)**: The proportion of variance in the dependent variable that is predictable from the independent variables.
- **Adjusted R-squared**: A modified version of $R^2$ that accounts for the number of predictors in the model, preventing overestimation when adding irrelevant features.

### Python Implementation

Using `scikit-learn` for a standard workflow:

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.linear_model import LinearRegression
from sklearn.metrics import mean_squared_error, r2_score

# 1. Generate Sample Data
X = np.array([[1], [2], [3], [4], [5]])
y = np.array([1, 2.1, 2.9, 4.2, 5.1])

# 2. Initialize and Train the Model
model = LinearRegression()
model.fit(X, y)

# 3. Make Predictions
y_pred = model.predict(X)

# 4. Evaluate the Model
print(f"Coefficients: {model.coef_}")
print(f"Intercept: {model.intercept_}")
print(f"MSE: {mean_squared_error(y, y_pred):.4f}")
print(f"R-squared: {r2_score(y, y_pred):.4f}")

# 5. Visualization
plt.scatter(X, y, color='blue', label='Actual Data')
plt.plot(X, y_pred, color='red', label='Regression Line')
plt.xlabel('X (Feature)')
plt.ylabel('y (Target)')
plt.legend()
plt.show()
```

## Interview Questions

**Q: What happens to the model if the assumption of homoscedasticity is violated?**
**A:** If homoscedasticity is violated (Heteroscedasticity), the OLS estimators remain unbiased, but they are no longer the Best Linear Unbiased Estimators (BLUE). The standard errors will be unreliable, leading to incorrect p-values and confidence intervals.

**Q: Explain the difference between R-squared and Adjusted R-squared.**
**A:** $R^2$ always increases (or stays the same) when new features are added, even if they are irrelevant. Adjusted $R^2$ penalizes the model for adding unnecessary features by incorporating the number of predictors. It only increases if the new feature improves the model more than would be expected by chance.

**Q: How do you detect and handle multicollinearity?**
**A:** Multicollinearity can be detected using the **Variance Inflation Factor (VIF)**. A VIF > 5 or 10 indicates high correlation. To handle it, you can remove highly correlated features, use dimensionality reduction (like PCA), or apply regularization techniques (Ridge/Lasso).

**Q: What is the significance of the Intercept term?**
**A:** The intercept ($\beta_0$) represents the expected value of Y when all independent variables (X) are zero. While it might not always have a practical interpretation in all domains, it is mathematically necessary to ensure the residuals have a mean of zero.

**Q: How does Linear Regression handle outliers?**
**A:** Linear Regression is highly sensitive to outliers because it minimizes the *squared* errors. A single extreme outlier can significantly pull the regression line toward it, skewing the coefficients. Techniques to handle this include using Robust Regression, removing outliers, or using transformations (like Log transform).
