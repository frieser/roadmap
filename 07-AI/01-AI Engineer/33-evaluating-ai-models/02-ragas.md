## Summary
Ragas (Retrieval Augmented Generation Assessment) is a specialized framework for evaluating RAG pipelines. It provides metrics that measure the performance of both the **Retrieval** component (finding the right data) and the **Generation** component (using that data correctly).

## Detailed Explanation
### **The RAG Triad**
Ragas focuses on the relationship between three elements: the **Query**, the **Context** (retrieved documents), and the **Answer**.

### **Core Metrics**
1.  **Faithfulness (Generation)**:
    *   Measures how much of the answer is derived directly from the retrieved context. It prevents hallucinations.
2.  **Answer Relevancy (Generation)**:
    *   Measures how well the answer addresses the user's original query.
3.  **Context Recall (Retrieval)**:
    *   Measures if all the necessary information to answer the question was actually present in the retrieved context. (Requires a ground truth answer).
4.  **Context Precision (Retrieval)**:
    *   Measures the quality of the ranking. Are the most relevant documents at the top of the retrieved list?

### **Why use Ragas?**
Instead of just saying "the chatbot is bad," Ragas tells you *where* it's bad.
-   If **Faithfulness** is low: Your model is hallucinating; try a better prompt or model.
-   If **Context Recall** is low: Your retrieval/embedding system is failing; try better chunking or a different vector DB.

## Interview Questions
*   **Q: Explain the metric "Faithfulness" in Ragas.**
    *   **A:** Faithfulness checks if the claims made in the generated answer can be found in the retrieved context. It is calculated by breaking the answer into individual statements and using an LLM to verify each statement against the context.
*   **Q: What is the difference between Context Precision and Context Recall?**
    *   **A:** Context Recall measures if the retrieved documents contain the answer (the "what"). Context Precision measures if the relevant documents are ranked higher than irrelevant ones (the "where").
*   **Q: How does Ragas calculate these metrics without human labels?**
    *   **A:** It uses an "LLM-as-a-Judge" internally. It prompts a model (like GPT-4) to perform specific tasks like "Extract all claims from this answer" and "Verify these claims against this context."
