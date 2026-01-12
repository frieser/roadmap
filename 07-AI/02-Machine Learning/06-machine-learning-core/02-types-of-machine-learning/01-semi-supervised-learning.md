---
tags: ['ai', 'roadmap']
---

## Summary
Semi-supervised learning (SSL) is a machine learning paradigm that sits between supervised and unsupervised learning. It leverages a small amount of labeled data in conjunction with a large volume of unlabeled data to improve model performance. This approach is highly effective when labeling data is expensive or slow, but raw data is abundant.

## Detailed Explanation

### Concept
In many practical applications, obtaining labeled data (e.g., human-annotated images or translated text) is the primary bottleneck. Semi-supervised learning addresses this by using the unlabeled data to understand the underlying structure, distribution, and manifold of the dataset. This allows the model to "fill in the gaps" and define more accurate decision boundaries than it could with labeled data alone.

### Core Assumptions
For SSL to work effectively, the data is generally expected to follow these properties:
- **Continuity Assumption**: Points that are close to each other in the feature space are likely to share the same label.
- **Cluster Assumption**: Data points tend to form discrete clusters. The decision boundary should ideally lie in low-density regions between these clusters.
- **Manifold Assumption**: High-dimensional data often lies on a lower-dimensional manifold. SSL helps the model learn this manifold structure using the large unlabeled set.

### Techniques

#### 1. Pseudo-labeling (Self-training)
The most common SSL technique. A model is trained on labeled data and then used to predict labels for unlabeled data. Predictions that meet a certain confidence threshold are converted into "pseudo-labels" and added to the training set for the next iteration.

#### 2. Consistency Regularization
This technique enforces the idea that the model's output should be invariant to small perturbations of the input. For instance, if you slightly rotate an unlabeled image, the model should still produce the same class prediction. Modern methods like **FixMatch** use this by ensuring a "strongly augmented" version of an image matches the "pseudo-label" generated from a "weakly augmented" version.

#### 3. Co-training
In co-training, the data features are split into two independent sets (views). Two models are trained, each on one view. Each model then labels the unlabeled data for the other model, essentially providing mutual supervision.

#### 4. Graph-Based Methods
Data points are represented as nodes in a graph, with edges reflecting similarity. Labels are then "propagated" from the few labeled nodes to the rest of the graph based on the strength of the connections.

## Interview Questions

### 1. When is Semi-Supervised Learning most useful?
SSL is most useful when you have a massive amount of data but only a fraction of it is labeled. It is commonly used in medical diagnosis, where expert annotation is expensive, or in Natural Language Processing (NLP), where raw text is infinite but labeled datasets are limited.

### 2. What is the main risk of Pseudo-labeling?
The main risk is **confirmation bias** (or error propagation). If the model makes a confident but incorrect prediction on an unlabeled sample, that error is "baked into" the next round of training as a pseudo-label, potentially leading the model to reinforce its own mistakes.

### 3. How does the "Low-Density Separation" principle relate to SSL?
The principle suggests that decision boundaries should not pass through dense areas of data. SSL uses unlabeled data to identify where these dense areas are, allowing the model to place the boundary in the "gaps" (low-density regions) between clusters.

### 4. What is the difference between Self-training and Co-training?
Self-training uses a single model to label its own data, while Co-training uses multiple models trained on different sets of features to label data for each other. Co-training is generally more robust as it reduces the chance of one model's bias dominating the process.

### 5. Explain the concept of "Consistency Regularization" in your own words.
It is a way of telling the model: "Regardless of small, insignificant changes to the input (like noise or shifting), your conclusion should remain the same." This helps the model find a stable and smooth decision boundary that respects the distribution of the unlabeled data.
