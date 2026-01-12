## Summary
Planning and Reasoning are the higher-order cognitive capabilities that allow an agent to solve multi-step, complex problems. It involves breaking a high-level goal into a sequence of sub-tasks and critically evaluating progress at each step.

## Detailed Explanation
### **Key Reasoning Techniques**
1.  **Chain of Thought (CoT)**:
    *   The model is prompted to "Think step-by-step." This increases the compute spent on the problem and usually leads to more logical conclusions.
2.  **Tree of Thoughts (ToT)**:
    *   The model explores multiple "branches" of reasoning simultaneously. It evaluates each branch and "backtracks" if a path looks unproductive.
3.  **Self-Reflection / Reflexion**:
    *   The agent critiques its own work. For example: "I just wrote this code, does it handle edge cases? No, let me rewrite the second function."
4.  **Plan-and-Execute**:
    *   The agent first generates a full plan (Step 1, Step 2, Step 3) and then executes it. This is more stable than "one-step-at-a-time" reasoning for long tasks.

### **The "LLM-as-a-Judge" for Planning**
In many systems, a more powerful model (e.g., GPT-4o) acts as the "Planner" and "Reviewer," while smaller, faster models (e.g., GPT-4o-mini) execute the individual tasks.

### **Reasoning Models (e.g., OpenAI o1)**
Newer "Reasoning" models use **Reinforcement Learning (RL)** and **Chain of Thought** during training to internalize these patterns, allowing them to solve extremely hard logic and math problems without explicit "think step-by-step" prompting.

## Interview Questions
*   **Q: Explain the difference between Chain of Thought (CoT) and Tree of Thoughts (ToT).**
    *   **A:** CoT is a linear sequence of reasoning steps. ToT is a non-linear exploration where the model considers multiple possible next steps, evaluates them, and can backtrack or pursue the most promising paths.
*   **Q: What is "Prompt Injection" in the context of agentic planning?**
    *   **A:** It's a security risk where a user (or a malicious website the agent is browsing) provides input that "highjacks" the agent's internal plan, forcing it to ignore its original goals and perform unintended actions.
*   **Q: How does "Self-Reflection" improve agent performance?**
    *   **A:** It allows the agent to identify and correct its own hallucinations or logic errors before providing a final answer, effectively acting as its own quality assurance layer.
