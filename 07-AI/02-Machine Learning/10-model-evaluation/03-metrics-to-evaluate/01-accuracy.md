---
tags: ['ai', 'roadmap']
---

## Summary
**Accuracy** is the most intuitive performance metric for classification models. It represents the proportion of total predictions that the model got correct (both true positives and true negatives). While easy to understand, it can be highly misleading when dealing with imbalanced datasets.

## Detailed Explanation

### Definition and Formula
Accuracy measures how often the classifier makes the correct prediction. It is the ratio of the number of correct predictions to the total number of input samples.

The mathematical formula is:
$$\text{Accuracy} = \frac{TP + TN}{TP + TN + FP + FN}$$

Where:
- **TP (True Positives)**: Correctly predicted positive instances.
- **TN (True Negatives)**: Correctly predicted negative instances.
- **FP (False Positives)**: Incorrectly predicted positive instances (Type I error).
- **FN (False Negatives)**: Incorrectly predicted negative instances (Type II error).

### When to Use
- When the target classes in the dataset are **balanced** (e.g., 50% Class A, 50% Class B).
- When the cost of a False Positive is roughly equal to the cost of a False Negative.

### The Imbalanced Data Pitfall (Accuracy Paradox)
Accuracy is a poor metric for imbalanced datasets. Consider a dataset where 99% of samples are "Normal" and 1% are "Fraudulent". 
- A "dumb" model that predicts "Normal" for **every** case will achieve **99% accuracy**.
- However, this model is useless because it fails to detect a single instance of fraud.
- In such cases, metrics like **Precision**, **Recall**, or **F1-Score** are much more informative.

### Python Implementation
Using `scikit-learn`, calculating accuracy is straightforward:

```python
from sklearn.metrics import accuracy_score

# True labels
y_true = [0, 1, 2, 0, 1, 2]
# Predicted labels
y_pred = [0, 2, 1, 0, 0, 1]

# Calculate accuracy
accuracy = accuracy_score(y_true, y_pred)
print(f"Accuracy: {accuracy:.2f}")

# Example with imbalanced data
y_true_imb = [0, 0, 0, 0, 0, 0, 0, 0, 0, 1]
y_pred_imb = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0] # Constant 0 prediction
print(f"Imbalanced Accuracy: {accuracy_score(y_true_imb, y_pred_imb):.2f}")
```

## Interview Questions
1. **Q: Why is accuracy not a good metric for imbalanced datasets?**
   - **A:** Because a model can achieve very high accuracy by simply predicting the majority class every time, while completely failing to identify the minority class (which is often the one we care about, like fraud or disease).

2. **Q: What is the "Accuracy Paradox"?**
   - **A:** It is the phenomenon where a model with higher accuracy may actually have lower predictive power or less utility than a model with lower accuracy, especially in imbalanced classification tasks.

3. **Q: In which scenario would you prefer Accuracy over F1-Score?**
   - **A:** When the dataset is perfectly balanced and the costs of False Positives and False Negatives are equal. In such cases, accuracy provides a simple and clear picture of overall performance.

4. **Q: If a model has 98% accuracy on a dataset where the minority class is 2%, what does this tell you about the model?**
   - **A:** It tells you very little. The model could be a "majority-class classifier" that never identifies the minority class. You need to check the Confusion Matrix, Precision, and Recall to understand if it's actually learning anything useful.
