---
tags: ['ai', 'roadmap']
---

## Summary
**Logistic Regression** is a fundamental supervised learning algorithm used primarily for **classification** tasks. Despite its name, it is used for predicting the probability of a discrete outcome (e.g., 0 or 1, Yes or No) rather than a continuous value. It works by applying the **Sigmoid function** to a linear combination of input features, mapping any real-valued number into a range between 0 and 1, which represents the probability of the positive class.

## Detailed Explanation

### The Sigmoid Function
The core of Logistic Regression is the **Sigmoid (or Logistic) function**, which squashes the output of a linear equation into a probability:

$$\sigma(z) = \frac{1}{1 + e^{-z}}$$

Where $z = w^T x + b$ (the linear part).
- As $z \to \infty$, $\sigma(z) \to 1$
- As $z \to -\infty$, $\sigma(z) \to 0$
- At $z = 0$, $\sigma(z) = 0.5$ (the default decision threshold)

### Cost Function: Log Loss (Binary Cross-Entropy)
Linear Regression uses Mean Squared Error (MSE), but in Logistic Regression, MSE results in a non-convex cost function with many local minima. Instead, we use **Log Loss** (also known as Binary Cross-Entropy):

$$J(w) = -\frac{1}{m} \sum_{i=1}^{m} [y^{(i)} \log(\hat{y}^{(i)}) + (1 - y^{(i)}) \log(1 - \hat{y}^{(i)})]$$

This function penalizes confident wrong predictions exponentially, ensuring a convex surface for optimization via **Gradient Descent**.

### Binary vs. Multiclass Classification
1. **Binary Classification**: Predicting between two classes (0 or 1).
2. **Multiclass Classification**:
   - **One-vs-Rest (OvR)**: Trains $N$ separate binary classifiers (one for each class vs. all others).
   - **Multinomial (Softmax)**: Generalizes Logistic Regression to multiple classes directly using the Softmax function, which ensures the sum of all class probabilities is 1.

### Python Implementation (Scikit-Learn)
Below is a practical example using `scikit-learn` to classify the Iris dataset (subset for binary classification).

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.linear_model import LogisticRegression
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, confusion_matrix

# Load data (Binary: Setosa vs Others)
iris = load_iris()
X = iris.data[:, :2]  # Using only first two features for visualization
y = (iris.target != 0).astype(int) 

# Split data
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Initialize and train model
model = LogisticRegression()
model.fit(X_train, y_train)

# Predict probabilities and classes
y_probs = model.predict_proba(X_test)[:, 1]
y_pred = model.predict(X_test)

print(f"Accuracy: {accuracy_score(y_test, y_pred):.2f}")
print("Confusion Matrix:")
print(confusion_matrix(y_test, y_pred))

# Visualization of decision boundary
x_min, x_max = X[:, 0].min() - 0.5, X[:, 0].max() + 0.5
y_min, y_max = X[:, 1].min() - 0.5, X[:, 1].max() + 0.5
xx, yy = np.meshgrid(np.arange(x_min, x_max, 0.02), np.arange(y_min, y_max, 0.02))
Z = model.predict(np.c_[xx.ravel(), yy.ravel()])
Z = Z.reshape(xx.shape)

plt.contourf(xx, yy, Z, alpha=0.3)
plt.scatter(X[:, 0], X[:, 1], c=y, edgecolors='k')
plt.xlabel('Sepal length')
plt.ylabel('Sepal width')
plt.title('Logistic Regression Decision Boundary')
plt.show()
```

## Interview Questions

**Q: Why do we use the Log Loss function instead of Mean Squared Error (MSE) in Logistic Regression?**
**A:** MSE is not used because the resulting cost function would be non-convex (wavy) due to the non-linear Sigmoid transformation. This makes it difficult for Gradient Descent to find the global minimum. Log Loss is convex, ensuring that Gradient Descent can converge to the global optimum.

**Q: What are the main assumptions of Logistic Regression?**
**A:** 1. The target variable is categorical (Binary/Multiclass). 2. Observations are independent. 3. Little or no multicollinearity among independent variables. 4. Linearity of independent variables and log-odds (logit).

**Q: How does Logistic Regression handle non-linear decision boundaries?**
**A:** Standard Logistic Regression creates a linear decision boundary. To handle non-linearity, you must use **Feature Engineering** (e.g., polynomial features) or **Kernel tricks** (though kernels are more common in SVMs) to project the data into a higher-dimensional space where it is linearly separable.

**Q: What is the difference between One-vs-Rest (OvR) and Multinomial Logistic Regression?**
**A:** OvR trains one binary classifier per class (Class A vs Not A, Class B vs Not B), whereas Multinomial (or Softmax) regression models all classes simultaneously using a single loss function and ensures class probabilities sum to 1.
