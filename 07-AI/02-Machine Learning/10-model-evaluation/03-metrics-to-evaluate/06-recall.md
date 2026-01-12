---
tags: ['ai', 'roadmap']
---

## Summary
**Recall**, also known as **Sensitivity** or **True Positive Rate (TPR)**, is a performance metric for classification models that measures the ability of a model to find all the relevant cases within a dataset. It is defined as the number of true positives divided by the total number of actual positives. Recall is particularly critical in domains where the cost of a "False Negative" is high, such as in medical diagnosis or threat detection.

## Detailed Explanation
Recall quantifies how many of the actual positive instances the model successfully "recalled."

### Formula
The mathematical representation of Recall is:

$$Recall = \frac{TP}{TP + FN}$$

Where:
- **TP (True Positives)**: Correctly predicted positive instances.
- **FN (False Negatives)**: Actual positive instances that were incorrectly predicted as negative.

### Intuition: The "Safety Net"
Think of Recall as a safety net. If you are screening for a rare disease, you want a net that catches every single person who actually has the disease. You might catch some healthy people by mistake (False Positives), but your priority is to ensure no sick person is missed (False Negatives).

### Comparison: Precision vs. Recall
- **Precision**: "Of all the cases I predicted as positive, how many were actually positive?" (Focuses on the quality of positive predictions).
- **Recall**: "Of all the actual positive cases, how many did I find?" (Focuses on the quantity of positive identification).

### Use Case: Disease Diagnosis
In cancer screening, a **False Negative** (telling a sick person they are healthy) can be fatal because the patient won't receive treatment. Therefore, we optimize for **High Recall**, even if it results in lower Precision (more healthy people being called back for further tests).

### Python Implementation
Using `scikit-learn`, we can easily calculate recall:

```python
from sklearn.metrics import recall_score

# Actual labels (1 = Positive, 0 = Negative)
y_true = [1, 0, 1, 1, 0, 1]
# Model predictions
y_pred = [1, 0, 0, 1, 0, 1]

# Calculate Recall
# pos_label=1 indicates that '1' is our positive class
recall = recall_score(y_true, y_pred)

print(f"Recall: {recall:.2f}")
# Output: Recall: 0.75 
# (The model caught 3 out of 4 actual positives)
```

For multi-class problems, you can specify an averaging strategy:
```python
# 'macro': calculate recall for each label, then find their unweighted mean
# 'weighted': calculate recall for each label, then find the mean weighted by support
recall_multi = recall_score(y_true_multi, y_pred_multi, average='macro')
```

## Interview Questions

**Q: What is the main difference between Recall and Precision?**
**A:** Precision focuses on the accuracy of the positive predictions (avoiding False Positives), while Recall focuses on the completeness of the positive identification (avoiding False Negatives).

**Q: In which scenario is Recall more important than Precision?**
**A:** Recall is prioritized when the cost of a False Negative is much higher than a False Positive. Examples include medical diagnosis (cancer screening), fraud detection, and disaster early-warning systems.

**Q: How does the classification threshold affect Recall?**
**A:** Lowering the classification threshold (e.g., from 0.5 to 0.3) typically increases Recall because the model becomes more likely to predict the positive class, catching more actual positives but likely increasing False Positives.

**Q: What is the F1-Score and why is it used?**
**A:** The F1-Score is the harmonic mean of Precision and Recall. It provides a single metric that balances both, which is especially useful when you need to find an optimal balance between the two or when dealing with imbalanced datasets.

**Q: Can a model have a Recall of 1.0 but still be completely useless?**
**A:** Yes. If a model predicts "Positive" for every single instance, its Recall will be 1.0 because it caught every actual positive. However, its Precision would be very low, and it would have no real discriminative power.
