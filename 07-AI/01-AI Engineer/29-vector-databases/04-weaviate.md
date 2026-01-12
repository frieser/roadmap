---
tags: ['ai', 'vectordb', 'weaviate', 'opensource', 'graphql']
---

## Summary
**Weaviate** is an open-source vector search engine that allows you to store data objects and vector embeddings from your favorite ML models. It is unique for its **GraphQL-based** interface and its "modular" architecture, which allows it to integrate directly with models like OpenAI, Hugging Face, and Cohere. Weaviate excels at **Hybrid Search**, combining vector-based semantic search with traditional keyword-based search (BM25).

## Detailed Explanation

### Core Architecture
Weaviate treats data as objects. It is more than just a vector store; it's a full-featured search engine.

1.  **Classes:** Similar to a schema or a table. You define classes (e.g., `Article`, `User`) and their properties.
2.  **Modules:** Weaviate has a plugin system. For example, the `text2vec-openai` module automatically handles sending your text to OpenAI to generate vectors.
3.  **HNSW Indexing:** Uses the Hierarchical Navigable Small World algorithm for fast, high-accuracy vector retrieval.
4.  **Hybrid Search:** Weaviate can perform a search that combines semantic results (vectors) and keyword results (BM25) with a weighted score, often yielding better results than either method alone.

### Key Features
*   **GraphQL API:** A powerful way to query your data and navigate relationships between objects.
*   **Auto-Schema:** Can automatically infer the data structure if you don't provide one.
*   **Scalability:** Supports sharding and replication for high availability.
*   **Multi-Tenancy:** Robust support for isolating data between different users or organizations.

### Implementation with Python (Weaviate v4 Client)

```python
import weaviate
import os

# Connect to a local Weaviate instance
client = weaviate.connect_to_local()

try:
    # 1. Create a Collection (Class)
    # collections = client.collections.create(
    #     name="Article",
    #     vectorizer_config=weaviate.classes.config.Configure.Vectorizer.text2vec_openai(),
    # )

    articles = client.collections.get("Article")

    # 2. Add an object
    articles.data.insert(
        properties={
            "title": "Introduction to Vector DBs",
            "content": "Weaviate is a powerful tool for AI..."
        }
    )

    # 3. Hybrid Search
    response = articles.query.hybrid(
        query="What is Weaviate?",
        limit=3
    )

    # for item in response.objects:
    #     print(item.properties)

finally:
    client.close()
```

## Interview Questions

**Q: What is 'Hybrid Search' in Weaviate?**
**A:** Hybrid search combines **vector search** (which captures semantic meaning) with **keyword search** (BM25, which captures exact matches). By combining both, you get the best of both worlds: finding documents that are conceptually relevant while still respecting specific terminology or rare keywords.

**Q: Why does Weaviate use GraphQL?**
**A:** GraphQL allows users to precisely define what data they want to retrieve and how to traverse relationships between different data objects. It is more flexible than traditional REST APIs for complex search and retrieval tasks.

**Q: What are 'Modules' in Weaviate?**
**A:** Modules are extensions that provide additional capabilities, such as vectorization (e.g., `text2vec-cohere`), image processing (`img2vec-neural`), or even generative tasks (e.g., `generative-openai` for performing RAG directly within the database).

**Q: Is Weaviate open-source?**
**A:** Yes, the core Weaviate engine is open-source. There is also a managed service called **Weaviate Cloud (WCD)** for users who don't want to manage their own infrastructure.
