---
tags: ['ai', 'roadmap']
---

## Summary
The **F1 Score** is a fundamental metric in classification tasks that provides a balance between **Precision** and **Recall**. It is defined as the harmonic mean of these two metrics, making it particularly useful for evaluating models on imbalanced datasets where accuracy might be a misleading indicator of performance.

## Detailed Explanation
The F1 Score is the harmonic mean of Precision and Recall. Unlike the arithmetic mean, the harmonic mean penalizes extreme values, ensuring that a high F1 score requires both Precision and Recall to be high.

### The Formula
The standard F1 Score (also known as $F_1$ or $F$-measure) is calculated as:

$$F1 = 2 \cdot \frac{\text{Precision} \cdot \text{Recall}}{\text{Precision} + \text{Recall}}$$

Where:
*   **Precision (Positive Predictive Value)**: $\frac{TP}{TP + FP}$ - "Of all positive predictions, how many were correct?"
*   **Recall (Sensitivity/True Positive Rate)**: $\frac{TP}{TP + FN}$ - "Of all actual positive cases, how many did we find?"

### Why Use the Harmonic Mean?
If we used the arithmetic mean, a model with a Precision of 1.0 and a Recall of 0.0 would have a score of 0.5. However, such a model is useless because it fails to find any positive instances. The harmonic mean results in an F1 score of 0.0 in this case, correctly identifying the model's failure.

### Multi-class Averaging
When dealing with more than two classes, we must aggregate the F1 scores across classes using different averaging strategies:

#### 1. Macro-averaging
- **Process**: Calculate the F1 score for each class independently and then take the unweighted mean of those scores.
- **Goal**: Treats all classes as equally important, regardless of how many samples each class has. It is ideal for evaluating performance on minority classes.

#### 2. Micro-averaging
- **Process**: Sum up the individual True Positives (TP), False Positives (FP), and False Negatives (FN) for all classes and then calculate a single F1 score from these aggregates.
- **Goal**: Measures the overall performance of the model across all instances. It is dominated by the performance on the majority classes.

#### 3. Weighted-averaging
- **Process**: Calculate the F1 score for each class and take the average weighted by the number of true instances (support) for each class.
- **Goal**: Accounts for class imbalance while still providing a per-class perspective.

### Python Example
Using `scikit-learn` is the standard way to calculate these metrics in Python.

```python
import numpy as np
from sklearn.metrics import f1_score, classification_report

# Simulated labels for a binary classification problem
y_true = [0, 1, 1, 0, 1, 1, 0, 0, 1, 0]
y_pred = [0, 1, 0, 0, 1, 1, 0, 1, 1, 0]

# Binary F1 Score
binary_f1 = f1_score(y_true, y_pred)
print(f"Binary F1 Score: {binary_f1:.4f}")

# Simulated labels for a multi-class problem (Classes 0, 1, 2)
y_true_multi = [0, 1, 2, 0, 1, 2, 0, 1, 2]
y_pred_multi = [0, 2, 1, 0, 0, 1, 0, 1, 2]

# Calculating different averages
macro_f1 = f1_score(y_true_multi, y_pred_multi, average='macro')
micro_f1 = f1_score(y_true_multi, y_pred_multi, average='micro')
weighted_f1 = f1_score(y_true_multi, y_pred_multi, average='weighted')

print(f"Macro F1 Score:    {macro_f1:.4f}")
print(f"Micro F1 Score:    {micro_f1:.4f}")
print(f"Weighted F1 Score: {weighted_f1:.4f}")

# Detailed report
print("\nClassification Report:")
print(classification_report(y_true_multi, y_pred_multi))
```

## Interview Questions

- **Q: Why is F1 Score better than Accuracy for imbalanced datasets?**
  - **A:** Accuracy measures the percentage of correct predictions. In a dataset where 95% of samples belong to Class A, a model that always predicts Class A will have 95% accuracy but is useless for identifying Class B. F1 Score considers both Precision and Recall, ensuring that the model is penalized for failing to catch the minority class (low Recall) or for being too reckless in its predictions (low Precision).

- **Q: If a model has a Precision of 1.0 but an F1 score of 0.8, what is its Recall?**
  - **A:** Using the formula $F1 = \frac{2PR}{P+R}$:
    $0.8 = \frac{2(1.0)R}{1.0 + R} \implies 0.8(1+R) = 2R \implies 0.8 + 0.8R = 2R \implies 0.8 = 1.2R \implies R = \frac{0.8}{1.2} = \frac{2}{3} \approx 0.67$.
    The Recall is approximately 0.67.

- **Q: When would you prefer Macro-F1 over Micro-F1?**
  - **A:** You should prefer **Macro-F1** when you want to evaluate the model's ability to perform well across all classes equally, regardless of their size. This is crucial in scenarios like medical diagnosis for rare diseases, where correctly identifying the rare class is as important as identifying the common one.

- **Q: What is the F-beta score, and how does it relate to the F1 score?**
  - **A:** The $F_\beta$ score is a generalization of the F1 score that allows you to weigh Precision and Recall differently using a parameter $\beta$. The formula is $F_\beta = (1 + \beta^2) \cdot \frac{\text{Precision} \cdot \text{Recall}}{(\beta^2 \cdot \text{Precision}) + \text{Recall}}$. When $\beta=1$, it becomes the F1 score. A $\beta > 1$ gives more weight to Recall, while $\beta < 1$ gives more weight to Precision.
