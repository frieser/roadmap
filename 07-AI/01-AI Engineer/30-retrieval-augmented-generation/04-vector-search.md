---
tags: ['ai', 'rag', 'search', 'retrieval']
---

## Summary
**Vector Search** is the retrieval component of a RAG pipeline. It involves taking a user's query, converting it into an embedding vector, and searching a vector database for the most similar document chunks. Modern vector search uses **Approximate Nearest Neighbor (ANN)** algorithms to provide near-instant results even across millions or billions of records.

## Detailed Explanation

### How Vector Search Works
1.  **Query Embedding:** The user's input string is sent to the same embedding model used to index the documents.
2.  **Distance Calculation:** The system calculates the distance (e.g., Cosine Similarity) between the query vector and all vectors in the database.
3.  **ANN Search:** To avoid checking every single vector (which is slow), vector DBs use algorithms like **HNSW** (Hierarchical Navigable Small World) or **IVF** to quickly navigate to the "neighborhood" of similar vectors.
4.  **Top-k Retrieval:** The system returns the `k` most similar chunks (e.g., Top 5).

### Improving Retrieval Quality
A common problem in RAG is that the "most similar" vector mathematically might not be the "most relevant" one for the LLM. AI Engineers use several techniques to fix this:

*   **Hybrid Search:** Combining vector search with keyword search (BM25).
*   **Re-ranking:** Using a more powerful but slower model (a **Cross-Encoder**) to re-evaluate the top 10-20 results from the initial vector search.
*   **Query Expansion:** Using an LLM to rewrite the user's query into multiple versions or more descriptive sentences before searching.
*   **Max Marginal Relevance (MMR):** A technique to select results that are both similar to the query and diverse from each other, preventing redundant information in the prompt.

### Python Implementation (Conceptual with Chroma)

```python
# Assuming 'collection' is already populated
query_text = "How do I deploy a Flask app to AWS?"

# Perform Top-k retrieval
results = collection.query(
    query_texts=[query_text],
    n_results=5,
    include=['documents', 'distances', 'metadatas']
)

# results['documents'][0] will contain the top 5 most similar chunks
```

### Re-ranking Example (with FlashRank or SentenceTransformers)
```python
# from flashrank import Ranker, RerankRequest
# ranker = Ranker()
# rerank_request = RerankRequest(query=query_text, passages=results['documents'][0])
# results = ranker.rerank(rerank_request)
```

## Interview Questions

**Q: What is 'Top-k' retrieval in RAG?**
**A:** `k` represents the number of chunks the system retrieves from the vector database. For example, if `k=5`, the system will find the 5 most similar pieces of information to the user's query.

**Q: What is the difference between Vector Search and Keyword Search?**
**A:** Keyword search looks for exact word matches (e.g., searching for "automobile" won't find "car"). Vector search looks for **semantic similarity**, meaning it understands that "automobile" and "car" are related concepts, even if the exact words don't match.

**Q: Why do we use 'Approximate Nearest Neighbor' (ANN) instead of a regular 'Nearest Neighbor' search?**
**A:** A standard Nearest Neighbor search (Brute Force) would require comparing the query to every single vector in the database, which is too slow for large datasets. ANN algorithms create index structures that allow the database to find "pretty good" matches extremely quickly, with a tiny trade-off in accuracy.

**Q: What is a 'Re-ranker' and why is it useful?**
**A:** A re-ranker is a second, more accurate model (typically a Cross-Encoder) that takes the top results from a fast vector search and re-orders them. It's useful because vector search (Bi-Encoders) is fast but can be less precise; re-ranking ensures the most contextually relevant information is placed at the top of the list for the LLM.
