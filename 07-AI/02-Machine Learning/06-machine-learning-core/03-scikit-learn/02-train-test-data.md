---
tags: ['ai', 'roadmap']
---

## Summary
The **train-test split** is a fundamental technique in supervised machine learning used to evaluate the performance of a model. It involves partitioning the available dataset into two separate subsets: a **training set** (used to build and tune the model) and a **testing set** (used to assess how well the model generalizes to new, unseen data). This separation is crucial for identifying and preventing **overfitting**.

## Detailed Explanation

In Scikit-Learn, the most common way to perform this split is using the `train_test_split` function from the `sklearn.model_selection` module.

### Core Parameters

1. **`test_size` / `train_size`**: 
   - Specifies the proportion of the dataset to include in the test or train split. 
   - Common ratios are **80/20** or **70/30**. 
   - If `test_size=0.2`, 20% of the data goes to testing and 80% to training.

2. **`random_state`**:
   - Ensures reproducibility. Machine learning splits are random by default. 
   - By setting `random_state=42` (or any fixed integer), you guarantee that every time you run the code, you get the exact same split. This is vital for debugging and comparing different models.

3. **`shuffle`**:
   - Boolean (default: `True`). Determines whether the data should be shuffled before splitting. Shuffling prevents the model from learning patterns based on the order of the data.

4. **`stratify`**:
   - Used to ensure that the train and test sets have the same proportion of class labels as the input dataset. 
   - **Critical for imbalanced datasets**. For example, if 90% of your data is "Class A" and 10% is "Class B", `stratify=y` ensures both splits maintain this 90/10 ratio.

### Python Implementation Example

```python
import numpy as np
from sklearn.model_selection import train_test_split

# Generate dummy data
X = np.arange(20).reshape((10, 2))  # 10 samples, 2 features
y = np.array([0, 0, 0, 0, 0, 1, 1, 1, 1, 1])  # Binary labels

# Perform the split
# We set test_size to 30%, random_state for reproducibility, 
# and stratify by y to keep label proportions.
X_train, X_test, y_train, y_test = train_test_split(
    X, y, 
    test_size=0.3, 
    random_state=42, 
    stratify=y
)

print(f"Total samples: {len(X)}")
print(f"Training samples: {len(X_train)} ({len(X_train)/len(X):.0%})")
print(f"Testing samples: {len(X_test)} ({len(X_test)/len(X):.0%})")
print(f"Labels in y_train: {y_train}")
print(f"Labels in y_test: {y_test}")
```

### Why Do We Split?

- **Generalization**: The primary goal is to see how the model performs on data it hasn't seen during training.
- **Overfitting Detection**: If a model performs exceptionally well on the training data but poorly on the test data, it has likely "memorized" the training noise instead of learning the underlying patterns.
- **Model Selection**: Comparing different algorithms or hyperparameters by their performance on the test set.

## Interview Questions

**Q: What is the risk of not using a test set?**
**A:** Without a test set, you cannot objectively measure the model's ability to generalize. You risk deploying an "overfitted" model that performs perfectly on your known data but fails completely in a real-world production environment with new data.

**Q: When should you NOT shuffle before splitting?**
**A:** Shuffling should be avoided in **Time Series** data. In time-dependent datasets, the order matters because you want to train on past data and test on future data. Shuffling would "leak" future information into the training set.

**Q: What is Stratified Sampling and why is it important?**
**A:** Stratified sampling ensures that the train and test sets are representative of the original dataset's class distribution. It is essential when dealing with imbalanced classes to prevent a situation where the test set contains none (or too few) of the minority class, making the evaluation metrics unreliable.

**Q: How do you choose the `test_size`?**
**A:** It depends on the total amount of data. For small to medium datasets, 20-30% is standard. For very large datasets (millions of rows), even 1% might be sufficient for a representative test set.

**Q: What is the difference between `train_test_split` and Cross-Validation?**
**A:** `train_test_split` is a single split (Hold-out method). Cross-Validation (like K-Fold) involves splitting the data into *k* subsets and rotating which one is the test set multiple times. Cross-validation provides a more robust estimate of performance but is computationally more expensive.
