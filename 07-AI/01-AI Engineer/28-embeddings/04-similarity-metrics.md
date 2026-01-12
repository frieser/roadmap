---
tags: ['ai', 'math', 'similarity', 'vectors']
---

## Summary
**Similarity Metrics** are mathematical formulas used to calculate the "distance" or "closeness" between two vectors in an embedding space. Choosing the correct metric is crucial because it determines how an AI system (like a search engine or RAG pipeline) perceives the relationship between data points. The most common metrics in AI Engineering are **Cosine Similarity**, **Dot Product**, and **Euclidean Distance**.

## Detailed Explanation

### 1. Cosine Similarity
*   **Definition:** Measures the cosine of the angle between two vectors. It focuses on the **orientation** (direction) of the vectors rather than their magnitude.
*   **Range:** -1 to 1 (usually 0 to 1 for normalized embeddings).
*   **Best for:** Most text embedding tasks where the "length" of the text shouldn't affect the similarity score.
*   **Formula:** `sim(A, B) = (A · B) / (||A|| * ||B||)`

### 2. Dot Product (Inner Product)
*   **Definition:** The sum of the products of the corresponding entries of the two sequences of numbers. It considers both the **angle** and the **magnitude** (length).
*   **Range:** -∞ to +∞.
*   **Best for:** Models trained with a dot-product objective. If vectors are normalized to a length of 1, Dot Product is mathematically identical to Cosine Similarity.
*   **Formula:** `A · B = ∑ (A_i * B_i)`

### 3. Euclidean Distance (L2 Distance)
*   **Definition:** The "straight-line" distance between two points in space.
*   **Range:** 0 to +∞. (A score of 0 means the vectors are identical).
*   **Best for:** Tasks where the magnitude of the features is important, such as image processing or certain physical sensor data.
*   **Formula:** `d(A, B) = √∑ (A_i - B_i)²`

### Choosing the Right Metric
The choice of metric is usually dictated by the **embedding model** you are using. You should use the same metric that was used during the model's training phase:
*   **OpenAI:** Recommends **Cosine Similarity**.
*   **Hugging Face Models:** Often trained with **Cosine Similarity** or **Dot Product**.
*   **Vector DBs:** Most databases (Pinecone, Chroma, etc.) allow you to specify the metric when creating a collection.

### Python Implementation (Numpy)

```python
import numpy as np

v1 = np.array([1, 2, 3])
v2 = np.array([2, 4, 6]) # Same direction, different magnitude

def cos_sim(a, b):
    return np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b))

def euclidean(a, b):
    return np.linalg.norm(a - b)

def dot_prod(a, b):
    return np.dot(a, b)

print(f"Cosine Similarity: {cos_sim(v1, v2)}") # Output: 1.0 (perfectly similar direction)
print(f"Euclidean Distance: {euclidean(v1, v2)}") # Output: 3.74 (different points in space)
print(f"Dot Product: {dot_prod(v1, v2)}") # Output: 28
```

## Interview Questions

**Q: Why is Cosine Similarity preferred for text embeddings over Euclidean Distance?**
**A:** In text, the length of the document often varies. Two documents about the same topic (e.g., "cats") might have very different vector lengths if one is a sentence and the other is a paragraph. Cosine similarity only looks at the *angle* (semantic direction), making it invariant to the magnitude, whereas Euclidean distance would show them as very different because the points are far apart in space.

**Q: When are Dot Product and Cosine Similarity the same?**
**A:** They are the same when the vectors are **normalized** (have a magnitude of 1). In this case, the denominator of the Cosine Similarity formula becomes 1, leaving only the Dot Product in the numerator.

**Q: What does a Cosine Similarity of 0 indicate?**
**A:** It indicates that the two vectors are **orthogonal** (perpendicular) to each other, meaning there is no linear correlation between them. In an embedding space, this suggests that the two items have no semantic relationship.

**Q: In a Vector Database, why is it important to select the correct distance metric at the time of index creation?**
**A:** The database uses the selected metric to build its internal search index (like HNSW). If you build an index using Euclidean distance but query it with Cosine similarity, the search results will be mathematically incorrect and irrelevant to your semantic needs.
