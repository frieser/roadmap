## Summary
Bias and Fairness in AI refer to the systematic errors or prejudices that an AI model may exhibit, often reflecting historical or social biases present in its training data. Responsible AI engineering involves identifying, measuring, and mitigating these biases to ensure equitable outcomes for all users.

## Detailed Explanation
### **Types of Bias**
1.  **Selection Bias**: Training data is not representative of the real-world population (e.g., a medical model trained only on data from one hospital).
2.  **Label Bias**: The labels in the training data reflect human prejudice (e.g., historical hiring data that favors men).
3.  **Algorithmic Bias**: The model's architecture or loss function inadvertently amplifies small differences in the data.

### **Fairness Metrics**
-   **Demographic Parity**: The probability of a positive outcome (e.g., being hired) is the same for all groups (e.g., men and women).
-   **Equal Opportunity**: The true positive rate is the same for all groups.
-   **Disparate Impact**: A mathematical comparison of outcome rates between different groups (e.g., the "4/5ths rule" in US employment law).

### **Mitigation Strategies**
-   **Pre-processing**: Cleaning and balancing the training data.
-   **In-processing**: Adding fairness constraints directly to the model's loss function.
-   **Post-processing**: Adjusting the model's outputs (e.g., changing the classification threshold for specific groups) to achieve fairer results.

## Interview Questions
*   **Q: How can an LLM show bias even if it wasn't explicitly programmed to be biased?**
    *   **A:** Because it is trained on massive datasets from the internet that contain historical biases, stereotypes, and unequal representations. The model learns these patterns as statistical regularities and reproduces them in its outputs.
*   **Q: What is the difference between Demographic Parity and Equal Opportunity?**
    *   **A:** Demographic Parity requires that the same percentage of people from each group get the positive outcome, regardless of their qualifications. Equal Opportunity requires that of those who *should* get the positive outcome (the qualified ones), the same percentage actually gets it across all groups.
*   **Q: What is "Red Teaming" in the context of fairness?**
    *   **A:** It is the process of intentionally trying to provoke a model into producing biased or harmful content to identify its weaknesses before it is released to the public.
