## Summary
LLM-based evaluation, also known as "LLM-as-a-Judge," uses powerful models (like GPT-4o) to grade the performance of other models or systems. This approach addresses the limitations of traditional string-matching metrics like BLEU or ROUGE, which fail to capture semantic meaning and nuances.

## Detailed Explanation
### **The Problem with Traditional Metrics**
-   **BLEU (Bilingual Evaluation Understudy)**: Measures n-gram overlap. Good for translation but penalizes valid paraphrases.
-   **ROUGE (Recall-Oriented Understudy for Gisting Evaluation)**: Measures recall of n-grams. Primarily used for summarization.
-   **Limitation**: Neither "understands" if the model's answer is factually correct or helpful if the words don't match the reference exactly.

### **LLM-as-a-Judge (G-Eval)**
In this pattern, a "Judge" model is given a prompt, the generated response, and optionally a reference answer. It then provides:
1.  **A Score**: (e.g., 1 to 5).
2.  **Reasoning**: An explanation of why it gave that score.

### **Best Practices for LLM Judges**
-   **Use a stronger model**: The judge must be significantly more capable than the model being evaluated.
-   **Provide a Rubric**: Clearly define what a "5" vs. a "1" looks like (e.g., "Factuality: Give 5 if all claims are supported by the text").
-   **Swap Position**: To avoid "positional bias" (where judges prefer the first answer they see), evaluate pairs twice with the order swapped.
-   **Chain of Thought**: Ask the judge to "think step-by-step" before providing the final score.

## Interview Questions
*   **Q: Why is BLEU score often considered a poor metric for evaluating creative writing or complex reasoning?**
    *   **A:** BLEU only looks at exact word overlaps (n-grams). In creative writing or reasoning, the same idea can be expressed in many different ways without sharing exact words. A model could provide a perfect answer that gets a BLEU score of 0.
*   **Q: What is "LLM-as-a-Judge" and what are its main biases?**
    *   **A:** It's using an LLM to evaluate the quality of another LLM's output. Common biases include **Positional Bias** (favoring the first answer), **Verbosity Bias** (favoring longer answers), and **Self-Preference Bias** (favoring its own style of writing).
*   **Q: How do you validate that your LLM Judge is actually reliable?**
    *   **A:** By calculating the **Human-AI Agreement**. You have humans grade a subset of the data and compare those grades to the LLM's grades using correlation metrics like Cohen's Kappa or Pearson correlation.
