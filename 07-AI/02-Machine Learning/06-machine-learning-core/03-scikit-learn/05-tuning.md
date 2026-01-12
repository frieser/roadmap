---
tags: ['ai', 'roadmap']
---

## Summary
Hyperparameter tuning is the process of finding the optimal set of parameters for a machine learning model that are not directly learned during training. Scikit-learn provides robust tools for this, primarily through cross-validated search strategies like **GridSearchCV** (exhaustive) and **RandomizedSearchCV** (sampling-based). For larger datasets or complex parameter spaces, **Successive Halving** techniques offer a significantly faster alternative by iteratively discarding low-performing candidates.

## Detailed Explanation

### 1. Introduction to Hyperparameters
Unlike model weights (learned from data), **hyperparameters** are set before training (e.g., `C` in SVM, `max_depth` in Random Forest). Tuning them is critical to prevent overfitting and maximize predictive performance.

### 2. Exhaustive Grid Search (`GridSearchCV`)
`GridSearchCV` performs an exhaustive search over a manually specified subset of the hyperparameter space. It evaluates every possible combination in the provided grid.

**Pros:** Guaranteed to find the best combination within the grid.
**Cons:** Computationally expensive; search time grows exponentially with the number of parameters.

```python
from sklearn.model_selection import GridSearchCV
from sklearn.svm import SVC

# Define the model
svc = SVC()

# Define the parameter grid
param_grid = {
    'C': [0.1, 1, 10, 100],
    'kernel': ['linear', 'rbf'],
    'gamma': ['scale', 'auto']
}

# Initialize and fit
grid_search = GridSearchCV(svc, param_grid, cv=5, scoring='accuracy')
grid_search.fit(X_train, y_train)

print(f"Best Params: {grid_search.best_params_}")
print(f"Best Score: {grid_search.best_score_}")
```

### 3. Randomized Parameter Optimization (`RandomizedSearchCV`)
Instead of trying every combination, `RandomizedSearchCV` samples a fixed number of parameter settings from specified distributions.

**Pros:** More efficient for high-dimensional spaces; user controls the computational budget via `n_iter`.
**Cons:** Might miss the absolute "best" point if the budget is too low.

```python
from sklearn.model_selection import RandomizedSearchCV
from scipy.stats import loguniform, randint
from sklearn.ensemble import RandomForestClassifier

# Define distributions
param_dist = {
    'n_estimators': randint(10, 200),
    'max_depth': [None, 10, 20, 30],
    'min_samples_split': randint(2, 11)
}

# Initialize and fit
random_search = RandomizedSearchCV(
    RandomForestClassifier(), 
    param_distributions=param_dist, 
    n_iter=20, 
    cv=5
)
random_search.fit(X_train, y_train)
```

### 4. Successive Halving (Efficient Tuning)
Introduced as an experimental feature, `HalvingGridSearchCV` and `HalvingRandomSearchCV` use a "tournament" approach. They start with many candidates using few resources (samples) and iteratively prune the worst performers while increasing resources for the survivors.

```python
from sklearn.experimental import enable_halving_search_cv  # Required
from sklearn.model_selection import HalvingGridSearchCV

# Faster than standard GridSearch for large datasets
halving_search = HalvingGridSearchCV(
    svc, param_grid, factor=3, resource='n_samples'
)
halving_search.fit(X_train, y_train)
```

### 5. Practical Tips & Best Practices
- **Pipelines**: Always wrap your search in a `Pipeline` to ensure preprocessing is included in cross-validation folds, preventing data leakage.
- **Nested Cross-Validation**: Use it to get an unbiased estimate of model performance when tuning hyperparameters.
- **Log-Uniform Scaling**: For parameters like `C` or `alpha` that span multiple orders of magnitude, use `loguniform` distributions in Randomized Search.
- **Refit**: Set `refit=True` (default) to automatically retrain the best model on the entire dataset after the search.

## Interview Questions

**Q: What is the main difference between Grid Search and Random Search?**
**A:** Grid Search exhaustively evaluates every combination in a specified grid, making it thorough but slow. Random Search samples a fixed number of combinations from distributions, making it more efficient for large parameter spaces and often finding a "good enough" solution much faster.

**Q: Why is it important to use Cross-Validation during hyperparameter tuning?**
**A:** Without cross-validation, you might pick parameters that overfit a specific train-test split. CV ensures the selected parameters generalize well across different subsets of the data.

**Q: How does Successive Halving improve tuning efficiency?**
**A:** It allocates more resources (like training samples) only to the most promising parameter combinations, quickly discarding those that perform poorly on smaller subsets of data.

**Q: What is "Data Leakage" in the context of hyperparameter tuning?**
**A:** Data leakage occurs if preprocessing (like scaling) is done on the entire dataset before cross-validation. This allows information from the validation fold to "leak" into the training process. Using Scikit-Learn `Pipelines` within the search prevents this.
