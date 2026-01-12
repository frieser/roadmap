---
tags: ['ai', 'roadmap']
---

## Summary
**Model Selection** in Scikit-Learn refers to the suite of tools and techniques used to compare, validate, and choose the best machine learning model or hyperparameter configuration. While simple training-test splits provide a baseline, advanced techniques like **Cross-Validation** and **Grid Search** ensure that models generalize well to unseen data and are not merely overfitting to a specific subset.

## Detailed Explanation

Model selection is a critical stage in the machine learning workflow. It involves evaluating multiple candidate models (e.g., Logistic Regression vs. Random Forest) or multiple configurations of a single model (hyperparameter tuning) to find the one that performs best on out-of-sample data.

### 1. Cross-Validation (CV)
Cross-validation is a statistical method used to estimate the skill of machine learning models. It is more robust than a single train-test split because it uses multiple folds of the data, ensuring every data point is used for both training and validation.

*   **K-Fold CV**: The dataset is split into *k* groups (folds). The model is trained *k* times, each time using a different fold as the test set and the remaining *k-1* folds as the training set.
*   **Stratified K-Fold**: A variation of K-Fold that ensures each fold has approximately the same percentage of samples of each target class as the complete set. This is essential for classification tasks with imbalanced classes.
*   **ShuffleSplit**: Randomly shuffles the data and yields a user-defined number of independent train/test splits.

### 2. Performance Metrics (Scoring)
Scikit-Learn provides a `scoring` parameter in most model selection tools (like `cross_val_score`) to define the evaluation criterion.

*   **Classification Metrics**:
    *   `accuracy`: Proportion of correct predictions.
    *   `precision` / `recall` / `f1`: Critical for imbalanced datasets.
    *   `roc_auc`: Area under the Receiver Operating Characteristic curve.
*   **Regression Metrics**:
    *   `neg_mean_squared_error`: Mean squared error (negative because Scikit-Learn maximizes scores).
    *   `r2`: Coefficient of determination.

### 3. Hyperparameter Tuning
Hyperparameters are settings that are not learned by the model during training (e.g., the depth of a tree).

*   **GridSearchCV**: Exhaustively searches through a specified subset of hyperparameters. It is guaranteed to find the best combination within the grid but can be computationally expensive.
*   **RandomizedSearchCV**: Samples a fixed number of hyperparameter settings from specified distributions. It is often much faster than Grid Search and typically finds a solution that is just as good.

### Python Implementation Example

```python
import numpy as np
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import cross_val_score, GridSearchCV, train_test_split
from sklearn.svm import SVC
from sklearn.metrics import classification_report

# Load dataset
data = load_breast_cancer()
X, y = data.data, data.target

# 1. Basic Cross-Validation
# Evaluating a Support Vector Classifier with 5 folds
model = SVC(kernel='linear', C=1)
cv_scores = cross_val_score(model, X, y, cv=5, scoring='accuracy')
print(f"CV Accuracy: {cv_scores.mean():.4f} (+/- {cv_scores.std() * 2:.4f})")

# 2. Hyperparameter Tuning with GridSearchCV
param_grid = {
    'C': [0.1, 1, 10, 100],
    'gamma': ['scale', 'auto'],
    'kernel': ['rbf', 'poly']
}

# n_jobs=-1 uses all available CPU cores
grid_search = GridSearchCV(SVC(), param_grid, cv=5, scoring='f1', n_jobs=-1)
grid_search.fit(X, y)

print(f"Best Hyperparameters: {grid_search.best_params_}")
print(f"Best F1-Score: {grid_search.best_score_:.4f}")

# 3. Final Model Assessment
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
best_model = grid_search.best_estimator_
y_pred = best_model.predict(X_test)

print("\nDetailed Classification Report on Test Set:")
print(classification_report(y_test, y_pred))
```

## Interview Questions

**Q: Why is Cross-Validation preferred over a single Train-Test split?**
**A:** A single split might be lucky or unlucky depending on which samples end up in the test set, leading to a high-variance estimate of model performance. Cross-validation averages performance over multiple splits, providing a more stable and reliable estimate of how the model will perform on unseen data.

**Q: What is the "Grid Search" bottleneck, and how do you solve it?**
**A:** The bottleneck is computational cost; as the number of hyperparameters and their possible values increase, the number of combinations grows exponentially. This can be solved by using `RandomizedSearchCV`, which samples a subset of the space, or by using "Halving" versions of these searches (`HalvingGridSearchCV`) which discard poorly performing candidates early.

**Q: Explain the difference between `KFold` and `StratifiedKFold`.**
**A:** `KFold` simply divides the data into blocks without looking at the labels. `StratifiedKFold` ensures that each fold contains the same proportion of classes as the original dataset. You should almost always use `StratifiedKFold` for classification to avoid folds that might be missing a minority class entirely.

**Q: What is "Data Leakage" in the context of Cross-Validation?**
**A:** Data leakage occurs when information from outside the training dataset is used to create the model. In CV, a common mistake is performing feature scaling or preprocessing on the *entire* dataset before splitting. This allows information from the validation fold to "leak" into the training folds. The solution is to use **Pipelines** that ensure preprocessing is only fit on the training portion of each fold.
