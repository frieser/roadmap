---
tags: ['ai', 'prompt-engineering', 'few-shot']
---

## Summary
**Few-Shot Prompting** is a technique where a small number of examples (demonstrations) are provided within the prompt to show the model how to perform a task. This "in-context learning" significantly improves performance on complex tasks, niche domains, or when a specific output format is required.

## Detailed Explanation

### The Power of Examples
While zero-shot prompting relies on the model's training, few-shot prompting leverages the model's ability to recognize patterns in the input. By providing 2-5 examples of an input followed by the desired output, you "prime" the model to follow the same logic for the final, unseen input.

### Structure of a Few-Shot Prompt
A typical few-shot prompt consists of:
1.  **Task Description (Optional):** A brief instruction.
2.  **Examples:** Pairs of `Input: [Example Input]` and `Output: [Example Output]`.
3.  **Target Input:** The actual data you want the model to process.

### Example: Custom Sentiment Analysis
> "Classify the sentiment of tweets about tech products.
>
> Tweet: 'The new battery life is insane!'
> Sentiment: High Praise
>
> Tweet: 'Software update is a bit buggy.'
> Sentiment: Minor Grievance
>
> Tweet: 'I hate the new keyboard layout.'
> Sentiment: Strong Dislike
>
> Tweet: 'The screen is brighter than expected.'
> Sentiment:"

### When to Use Few-Shot
*   **Specific Formats:** When you need the output in a very specific JSON, YAML, or custom format.
*   **Complex Reasoning:** When the logic required to solve the task is non-standard.
*   **Niche Domains:** For medical, legal, or specialized technical jargon that the model might not have seen frequently during training.
*   **Consistency:** To ensure the model uses the same terminology across multiple calls.

### Python Implementation with LangChain
LangChain provides a dedicated `FewShotPromptTemplate` for managing these prompts.

```python
from langchain.prompts import PromptTemplate, FewShotPromptTemplate

# 1. Define examples
examples = [
    {"input": "happy", "output": "sad"},
    {"input": "tall", "output": "short"},
]

# 2. Define example formatter
example_formatter_template = """
Word: {input}
Antonym: {output}
"""
example_prompt = PromptTemplate(
    input_variables=["input", "output"],
    template=example_formatter_template,
)

# 3. Create FewShotPromptTemplate
few_shot_prompt = FewShotPromptTemplate(
    examples=examples,
    example_prompt=example_prompt,
    prefix="Give the antonym of every input",
    suffix="Word: {input}\nAntonym:",
    input_variables=["input"],
    example_separator="\n",
)

# print(few_shot_prompt.format(input="big"))
```

## Interview Questions

**Q: What is Few-Shot Prompting?**
**A:** Few-shot prompting is a technique where you provide a few examples of a task (input-output pairs) within the prompt itself. This helps the LLM understand the context, pattern, and desired output format better than a simple instruction.

**Q: What is 'In-Context Learning'?**
**A:** In-context learning refers to the model's ability to learn from the information provided in the prompt (the context) without any changes to its underlying weights. It "learns" the pattern temporarily for that specific inference call.

**Q: How many examples are typically used in few-shot prompting?**
**A:** Usually 1 to 5 examples are sufficient. Using too many examples (e.g., 20+) can consume a lot of tokens and might actually confuse the model or reach the context window limit without providing proportional benefits.

**Q: What should you do if the examples you provide are biased?**
**A:** The model is likely to pick up on that bias. For example, if all your sentiment examples are positive, the model might become biased toward predicting positive sentiment. It is crucial to provide a diverse and balanced set of examples.
