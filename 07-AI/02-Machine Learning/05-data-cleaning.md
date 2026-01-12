---
tags: ['ai', 'roadmap']
---

## Summary
Data cleaning is the critical process of identifying and correcting errors, inconsistencies, and inaccuracies in a dataset before it is used for machine learning models. Since the performance of any AI system is fundamentally limited by the quality of its training data—a principle known as "Garbage In, Garbage Out" (GIGO)—this stage often consumes the majority of a data scientist's time. A successful cleaning phase ensures data integrity, improves model accuracy, and reduces the risk of biased or nonsensical outputs.

## Detailed Explanation
### **1. Handling Missing Values**
Missing data is a common challenge in real-world datasets. Strategies include:
- **Deletion**: Removing rows or columns with missing values. Useful if the missingness is extensive and the data isn't missing at random (MNAR).
- **Imputation**: Filling in missing values using statistical measures:
    - **Univariate**: Filling with Mean, Median (robust to outliers), or Mode (for categorical data).
    - **Multivariate**: Using algorithms like **K-Nearest Neighbors (KNN)** or **Iterative Imputer** to predict missing values based on other features.
- **Flagging**: Creating a new binary feature to indicate if a value was missing, which can be a signal itself.

### **2. Outlier Detection and Treatment**
Outliers are data points that differ significantly from other observations.
- **Detection**:
    - **Z-Score**: Measures how many standard deviations a point is from the mean.
    - **Interquartile Range (IQR)**: Identifying points outside $[Q1 - 1.5 \times IQR, Q3 + 1.5 \times IQR]$.
- **Treatment**:
    - **Trimming/Removal**: Deleting outliers if they are likely errors.
    - **Winsorization/Capping**: Limiting the extreme values to a specific percentile.
    - **Transformation**: Applying Log or Box-Cox transformations to reduce the impact of skewness.

### **3. Data Consistency and Structural Errors**
- **Typos and Formatting**: Standardizing string values (e.g., converting "U.S.A" and "usa" to "USA").
- **Duplicate Records**: Identifying and removing identical rows that can lead to overfitting or biased evaluations.
- **Unit Consistency**: Ensuring all measurements use the same units (e.g., meters vs. feet).

### **4. Exploratory Data Analysis (EDA) Integration**
Data cleaning is iterative and tightly coupled with EDA. Visualizations like histograms, box plots, and scatter plots help identify:
- Distribution skews and scale issues.
- Correlation patterns.
- Hidden outliers.
- Feature relationships that reveal illogical data (e.g., negative age or impossible dates).

### **5. Feature Scaling (Brief Overview)**
While often considered "preprocessing," scaling is the final step in ensuring clean, consistent input:
- **Normalization (Min-Max Scaling)**: Rescaling features to a range like $[0, 1]$.
- **Standardization (Z-score Normalization)**: Centering features around a mean of 0 with a standard deviation of 1.

## Interview Questions
- **Q: How do you decide whether to impute or delete missing values?**
  - **A:** Deletion is preferred if the percentage of missing values is very high (e.g., >60%) or if the missing data is not representative of the population. Imputation is better when the dataset is small or when the missingness follows a pattern (MAR/MCAR) that can be reliably estimated using other features.
- **Q: What is the risk of using the Mean for imputation?**
  - **A:** The mean is highly sensitive to outliers. If a feature is skewed, using the mean can introduce bias and reduce the variance of the dataset artificially, potentially misleading the model. The median is often a safer choice for skewed distributions.
- **Q: Can outliers ever be useful?**
  - **A:** Yes. In tasks like **Anomaly Detection** (e.g., fraud detection or equipment failure prediction), outliers are the primary focus. Deleting them in these contexts would remove the very signal the model is trying to learn.
- **Q: Why is removing duplicates important for model evaluation?**
  - **A:** If duplicates exist across the training and test sets (Data Leakage), the model may "memorize" specific rows rather than generalizing, leading to artificially high performance metrics that won't hold up in production.
