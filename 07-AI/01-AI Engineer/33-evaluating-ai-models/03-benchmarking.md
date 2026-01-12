## Summary
Benchmarking involves evaluating an LLM on standardized datasets to compare its performance against other models. These benchmarks cover various capabilities, from general knowledge to specific tasks like coding or math.

## Detailed Explanation
### **Popular Benchmarks**
1.  **MMLU (Massive Multitask Language Understanding)**:
    *   Covers 57 subjects across STEM, the humanities, social sciences, and more. It is the gold standard for general intelligence.
2.  **GSM8K (Grade School Math 8K)**:
    *   High-quality grade school math word problems. Tests multi-step reasoning.
3.  **HumanEval**:
    *   A set of programming problems in Python. The model must write code that passes unit tests.
4.  **LMSYS Chatbot Arena**:
    *   A crowd-sourced, blind ELO rating system where humans vote on which model's response is better. This is considered the most "realistic" benchmark for human preference.

### **Benchmark Contamination**
A major issue where the benchmark's questions and answers are accidentally included in the model's training data. This leads to "artificially" high scores that don't reflect true reasoning capability.

### **Evaluating Your Own Model**
For a specific business use case, public benchmarks are less useful than **Custom Evaluation Sets**. You should create a "Golden Dataset" of 50-100 real queries and perfect answers specific to your domain.

## Interview Questions
*   **Q: What is the MMLU benchmark and what does it measure?**
    *   **A:** MMLU is a comprehensive benchmark consisting of multiple-choice questions across 57 subjects. It measures a model's broad world knowledge and problem-solving abilities across various academic and professional domains.
*   **Q: Why might a model have a high MMLU score but perform poorly in a real-world coding assistant role?**
    *   **A:** Because MMLU tests knowledge via multiple-choice questions, whereas coding requires generation, syntax correctness, and complex logic. A model might be good at "knowing facts" but poor at "applying them" to generate functional code (which is better tested by HumanEval).
*   **Q: What is "Data Contamination" in benchmarks?**
    *   **A:** It occurs when the evaluation data (questions and answers) is present in the model's training set. This allows the model to "memorize" the answers rather than "reasoning" through them, resulting in scores that overestimate the model's true intelligence.
