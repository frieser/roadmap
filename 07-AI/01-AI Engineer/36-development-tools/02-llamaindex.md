## Summary
LlamaIndex (formerly GPT Index) is a specialized framework designed to connect private or domain-specific data to Large Language Models. It focuses heavily on data ingestion, indexing, and advanced retrieval strategies.

## Detailed Explanation
### **The LlamaIndex Workflow**
1.  **Loading**: Using "LlamaHub" data connectors to ingest data from PDFs, Notion, Slack, SQL, etc.
2.  **Indexing**: Organizing the data into a searchable structure.
    *   **VectorStoreIndex**: The most common, used for semantic search.
    *   **SummaryIndex**: Good for summarizing a collection of documents.
    *   **KnowledgeGraphIndex**: For structured, relationship-based retrieval.
3.  **Querying**: Using a "Query Engine" or "Chat Engine" to retrieve data and generate a response.

### **Advanced Retrieval Techniques**
-   **Sub-Question Querying**: Breaking a complex query into multiple sub-questions across different documents.
-   **Router Query Engine**: Automatically deciding which index to use based on the user's question.
-   **Post-processing / Reranking**: Using a second model (like Cohere Rerank) to improve the order of retrieved results.

### **LlamaIndex vs. LangChain**
-   **LangChain**: Better for complex "agentic" workflows, logic, and general-purpose LLM orchestration.
-   **LlamaIndex**: Better for data-heavy applications where efficient retrieval and indexing of complex documents are the primary focus.

## Interview Questions
*   **Q: What is LlamaHub?**
    *   **A:** LlamaHub is an open-source library of data connectors (loaders) for LlamaIndex, allowing you to easily ingest data from hundreds of sources like GitHub, Google Drive, and various databases.
*   **Q: Explain "Reranking" in LlamaIndex.**
    *   **A:** Reranking is a post-processing step where a more precise (but slower) model evaluates the top $N$ results from an initial vector search and re-orders them to ensure the most relevant content is at the very top.
*   **Q: When would you use a "SummaryIndex" instead of a "VectorStoreIndex"?**
    *   **A:** You use a SummaryIndex when the user's question requires information from *all* parts of a document (e.g., "Summarize this entire report") rather than finding a specific piece of information (e.g., "What was the revenue in Q3?").
