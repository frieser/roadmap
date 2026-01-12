---
tags: ['ai', 'roadmap']
---

## Summary

**Decision Trees** are non-parametric supervised learning methods used for classification and regression. They predict the value of a target variable by learning simple decision rules inferred from data features. **Random Forest** is an ensemble learning method that operates by constructing a multitude of decision trees at training time and outputting the class that is the mode of the classes (classification) or mean prediction (regression) of the individual trees. It is one of the most popular and powerful machine learning algorithms due to its robustness and ability to handle high-dimensional data without extensive feature engineering.

## Detailed Explanation

### Decision Trees

A decision tree is a flowchart-like structure where each internal node represents a "test" on an attribute, each branch represents the outcome of the test, and each leaf node represents a class label.

#### 1. Splitting Criteria
To build a tree, we need to decide which feature to split on at each node. Two common metrics are used:

- **Gini Impurity**: Measures the frequency at which a randomly chosen element from the set would be incorrectly labeled if it was randomly labeled according to the distribution of labels in the subset.
  $$Gini(p) = 1 - \sum_{i=1}^{J} p_i^2$$
- **Entropy (Information Gain)**: Measures the amount of uncertainty or disorder in the data. Information Gain is the reduction in entropy after a dataset is split on an attribute.
  $$Entropy(p) = -\sum_{i=1}^{J} p_i \log_2 p_i$$

#### 2. Overfitting and Pruning
Decision trees are prone to **overfitting**, where the tree becomes too complex and captures noise in the training data. **Pruning** is the process of removing sections of the tree that provide little power to classify instances. This can be done via:
- **Pre-pruning**: Stopping the tree growth early (e.g., setting `max_depth` or `min_samples_split`).
- **Post-pruning**: Removing branches from a fully grown tree.

### Random Forest and Ensemble Methods

**Ensemble methods** combine multiple machine learning models to create a more powerful model. Random Forest uses two main techniques:

#### 1. Bagging (Bootstrap Aggregating)
Bagging involves training multiple decision trees on different random subsets of the training data, sampled with replacement (bootstrapping). The final prediction is made by averaging the predictions (regression) or taking a majority vote (classification). This reduces the **variance** of the model without increasing the bias.

#### 2. Feature Randomness (The "Random" in Random Forest)
In a standard decision tree, each node is split using the best feature among all available features. In a Random Forest, each node is split using the best feature among a **random subset of features**. This decorrelates the trees, ensuring that even if one feature is a very strong predictor, the trees don't all look the same.

### Python Implementation

Using `scikit-learn`:

```python
import numpy as np
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.tree import DecisionTreeClassifier
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score

# Load dataset
data = load_iris()
X, y = data.data, data.target
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=42)

# 1. Decision Tree
dt_clf = DecisionTreeClassifier(criterion='gini', max_depth=3)
dt_clf.fit(X_train, y_train)
dt_pred = dt_clf.predict(X_test)
print(f"Decision Tree Accuracy: {accuracy_score(y_test, dt_pred):.4f}")

# 2. Random Forest
rf_clf = RandomForestClassifier(n_estimators=100, criterion='entropy', random_state=42)
rf_clf.fit(X_train, y_train)
rf_pred = rf_clf.predict(X_test)
print(f"Random Forest Accuracy: {accuracy_score(y_test, rf_pred):.4f}")

# Feature Importances in Random Forest
for name, score in zip(data.feature_names, rf_clf.feature_importances_):
    print(f"Feature: {name}, Score: {score:.4f}")
```

## Interview Questions

**Q: What is the main difference between a Decision Tree and a Random Forest?**
**A:** A Decision Tree is a single model that can easily overfit if not pruned. A Random Forest is an ensemble of many Decision Trees trained on different data subsets and feature subsets, which significantly reduces overfitting and improves generalizability.

**Q: Why do we use 'Gini' or 'Entropy' in Decision Trees?**
**A:** They are impurity measures used to evaluate the quality of a split. Gini is computationally faster as it doesn't involve logarithmic functions, while Entropy can sometimes produce slightly more balanced trees. In practice, the choice rarely impacts performance significantly.

**Q: What is 'Out-of-Bag' (OOB) error in Random Forest?**
**A:** Since Random Forest uses bootstrapping (sampling with replacement), about 1/3 of the data is left out for each tree. This "out-of-bag" data can be used as a validation set to estimate the model's performance without needing a separate cross-validation set.

**Q: How does Random Forest handle high-dimensional data?**
**A:** It handles high-dimensional data well by randomly selecting a subset of features at each split. This prevents the model from being dominated by a few strong features and allows it to capture complex interactions between many variables.

**Q: What are the hyperparameters you would tune in a Random Forest?**
**A:** Key hyperparameters include `n_estimators` (number of trees), `max_features` (size of the random subsets of features), `max_depth` (maximum depth of each tree), and `min_samples_leaf` (minimum samples required at a leaf node).
