## Summary
Supervised Fine-Tuning (SFT) is the stage where a pre-trained model is trained on labeled examples (input-output pairs) to follow specific instructions or perform specific tasks. It is usually the first step in the "Alignment" process, often followed by RLHF.

## Detailed Explanation
### **The Alignment Pipeline**
1.  **Pre-training**: Learning from the whole internet (predicting the next token). Result: Base Model.
2.  **SFT (Supervised Fine-Tuning)**: Learning from high-quality, human-labeled instructions. Result: Instruct Model.
3.  **RLHF / DPO**: Aligning with human preferences (e.g., "Which answer is safer/better?").

### **Key Metrics in SFT**
-   **Training Loss**: Should steadily decrease, but if it goes to zero too fast, the model is likely over-fitting.
-   **Validation Loss**: The most important metric; if it starts increasing while training loss decreases, the model is over-fitting.
-   **Perplexity**: A measure of how well the model predicts the test data. Lower is better.

### **Catastrophic Forgetting**
This occurs when a model loses the general knowledge it gained during pre-training because it was over-optimized for a specific task during SFT. To mitigate this, developers often use a low learning rate or mix in some general pre-training data.

## Interview Questions
*   **Q: What is the difference between a "Base" model and an "Instruct" model?**
    *   **A:** A base model is trained to predict the next word on a large corpus and might just continue a prompt (e.g., "What is the capital of France?" -> "What is the capital of Italy?"). An instruct model has undergone SFT to understand and follow the intent of the prompt (e.g., -> "The capital of France is Paris.").
*   **Q: How do you know when to stop training during SFT?**
    *   **A:** By monitoring the validation loss. You should stop training when the validation loss reaches its minimum and starts to trend upward (Early Stopping).
*   **Q: Why is SFT usually preferred over full pre-training for most business applications?**
    *   **A:** Pre-training requires trillions of tokens and millions of dollars in compute. SFT can be done with as few as a few hundred or thousand high-quality examples and significantly less compute, making it accessible for domain-specific alignment.
