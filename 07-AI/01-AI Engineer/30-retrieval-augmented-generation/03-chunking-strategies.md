---
tags: ['ai', 'rag', 'chunking', 'preprocessing']
---

## Summary
**Chunking** is the process of breaking large documents into smaller, manageable pieces (chunks) before they are embedded and stored in a vector database. Choosing the right chunking strategy is critical for RAG performance because it determines the "granularity" of the information the system can retrieve. Chunks that are too large may contain irrelevant info, while chunks that are too small may lose the necessary context.

## Detailed Explanation

### Why Chunk?
1.  **Token Limits:** Embedding models and LLMs have finite context windows.
2.  **Semantic Focus:** Smaller chunks help the vector search find exact answers rather than broad documents.
3.  **Context Window Efficiency:** You want to fill the LLM's prompt with as much *relevant* information as possible.

### Common Chunking Strategies

1.  **Fixed-Size Chunking:**
    *   **Description:** Splitting text into chunks of a fixed number of characters or tokens (e.g., 500 characters).
    *   **Pros:** Simple and fast.
    *   **Cons:** Often cuts off sentences in the middle, losing meaning.

2.  **Recursive Character Chunking:**
    *   **Description:** Attempts to split on a hierarchy of characters (e.g., `\n\n`, `\n`, ` `, `""`) to keep paragraphs and sentences together.
    *   **Pros:** Much better at preserving semantic context than fixed-size.

3.  **Semantic Chunking:**
    *   **Description:** Uses an embedding model to determine where the meaning of the text changes. If the "semantic distance" between two sentences is too high, a new chunk is started.
    *   **Pros:** Highly accurate and context-aware.
    *   **Cons:** Computationally expensive.

### The Role of 'Overlap'
When chunking, it is common practice to include an **overlap** (e.g., 10-20% of the chunk size). This ensures that information located at the boundary of a split isn't lost and provides the LLM with some surrounding context for each chunk.

### Implementation with Python (LangChain)

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

text = "Your very long document text goes here..."

# Initialize the splitter
splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200,
    separators=["\n\n", "\n", " ", ""]
)

# Split the text
chunks = splitter.split_text(text)

# print(f"Created {len(chunks)} chunks.")
# print(f"First chunk: {chunks[0][:100]}")
```

## Interview Questions

**Q: What is the trade-off between large and small chunk sizes?**
**A:** **Large chunks** provide more context to the LLM but may include irrelevant information ("noise") that confuses the model or wastes tokens. **Small chunks** are more precise for retrieval but may lack the surrounding context needed to understand the information correctly.

**Q: Why is 'Recursive Character Chunking' often the default choice?**
**A:** Because it tries to respect the natural structure of human writing. By splitting on double newlines (paragraphs) first, then single newlines (sentences), and finally spaces (words), it maximizes the chances of keeping a complete idea within a single chunk.

**Q: What is 'Chunk Overlap' and why is it used?**
**A:** Chunk overlap is the practice of having the end of one chunk share some text with the beginning of the next. It is used to prevent "context loss" at the split points, ensuring that the semantic relationship between sentences isn't broken just because they happened to fall on either side of a character limit.

**Q: When would you use 'Semantic Chunking'?**
**A:** You would use semantic chunking when the document structure is highly inconsistent or when the highest possible retrieval accuracy is required. It is especially useful for long, complex documents where thematic shifts don't always align with paragraph breaks.
