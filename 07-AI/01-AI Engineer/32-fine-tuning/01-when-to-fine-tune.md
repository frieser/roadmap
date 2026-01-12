## Summary
Fine-tuning is the process of taking a pre-trained model and further training it on a smaller, domain-specific dataset. It is a powerful tool but should only be used when alternative methods like Retrieval-Augmented Generation (RAG) or prompt engineering are insufficient.

## Detailed Explanation
### **Fine-Tuning vs. RAG**
| Feature | Fine-Tuning | RAG |
| --- | --- | --- |
| **New Knowledge** | Static (learned during training) | Dynamic (retrieved at runtime) |
| **Accuracy** | Prone to hallucinations if facts change | Higher factual accuracy (cites sources) |
| **Format/Style** | Excellent for learning specific styles | Limited to prompt instructions |
| **Latency** | Lower (no retrieval step) | Higher (retrieval + generation) |
| **Cost** | High (GPU resources needed) | Low to Medium (Vector DB + API calls) |

### **When to Fine-Tune**
1.  **Style and Tone**: When you need the model to consistently speak in a very specific brand voice or use technical jargon correctly.
2.  **Domain-Specific Vocabulary**: Teaching the model terms not found in common web crawls (e.g., proprietary medical codes).
3.  **Strict Output Formatting**: If the model must output highly complex JSON or specialized code structures that prompt engineering can't consistently achieve.
4.  **Reducing Latency**: Removing the need for long system prompts or few-shot examples by "baking" that knowledge into the model's weights.

### **The "RAG-First" Rule**
In most AI engineering scenarios, it is better to start with RAG for factual knowledge and only move to fine-tuning if you need to improve the "how" (style/format) rather than the "what" (facts).

## Interview Questions
*   **Q: If you need to build a bot that answers questions about a company's internal HR policies, would you use fine-tuning or RAG?**
    *   **A:** RAG. HR policies change frequently, and RAG allows the model to retrieve the most up-to-date document at runtime. Fine-tuning would require retraining the model every time a policy is updated and is more likely to hallucinate facts.
*   **Q: Name a scenario where fine-tuning is strictly better than RAG.**
    *   **A:** When the goal is to follow a extremely complex, non-standard output format (like a proprietary markup language) where even with 10-shot prompting the model fails. Fine-tuning on thousands of examples would make the model "native" to that format.
*   **Q: What is the main drawback of fine-tuning for knowledge updates?**
    *   **A:** Knowledge "staleness." The model's information is frozen at the moment of training. It also suffers from "catastrophic forgetting," where it might lose general reasoning abilities as it over-indexes on the new data.
