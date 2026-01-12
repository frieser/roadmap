## Summary
AI Safety is the discipline of ensuring that AI systems act in accordance with human values and do not cause harm, even when they encounter unexpected situations or are given ambiguous goals. This includes both technical guardrails and ethical frameworks.

## Detailed Explanation
### **The Alignment Problem**
The challenge of ensuring that an AI's goals perfectly match the user's intent. A common example is the "Paperclip Maximizer," where an AI given the goal to "make as many paperclips as possible" might eventually see humans as a source of atoms for paperclips.

### **Safety Guardrails**
1.  **Llama Guard / ShieldGemma**: Specialized models fine-tuned to classify whether a prompt or response is harmful (e.g., violence, hate speech, self-harm).
2.  **NeMo Guardrails**: A programmable framework from NVIDIA that allows developers to define "Rails" (rules) for their AI, such as "Do not talk about politics" or "Only answer based on the provided documents."
3.  **Constitution AI (RLAIF)**: A technique used by Anthropic where the model is given a "Constitution" (a set of ethical rules) and uses it to self-correct and align its own outputs.

### **Global Frameworks**
-   **EU AI Act**: The world's first comprehensive AI law, which categorizes AI systems by risk level (Unacceptable, High, Limited, Minimal).
-   **NIST AI Risk Management Framework (RMF)**: A voluntary framework in the US to help organizations manage AI risks.

## Interview Questions
*   **Q: What are "Guardrails" in the context of an AI application?**
    *   **A:** Guardrails are an independent layer of software that monitors the inputs and outputs of an AI model to ensure they comply with safety, security, and topicality rules. They can block harmful content, prevent off-topic discussions, and ensure data privacy.
*   **Q: What is the "EU AI Act" and how does it affect AI Engineers?**
    *   **A:** It is a regulation that imposes strict requirements on AI systems used in the EU, particularly "High-Risk" systems (like those used in hiring or law enforcement). AI Engineers must ensure their models are transparent, documented, and undergo rigorous risk assessments to comply.
*   **Q: Explain the concept of "Constitutional AI."**
    *   **A:** It is a method of training AI to be safe by providing it with a written set of principles (a constitution). The model then uses these principles to evaluate and refine its own responses during training, reducing the need for human labeling of every harmful prompt.
