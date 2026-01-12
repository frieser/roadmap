---
tags: ['ai', 'prompt-engineering', 'system-prompt', 'roleplaying']
---

## Summary
A **System Prompt** (or System Message) is a high-level instruction that sets the persona, constraints, and behavior of an AI model for a given session. In the Chat Completion API, it is usually the first message provided and carries significant weight in determining how the model interprets and responds to all subsequent user inputs.

## Detailed Explanation

### The Role of the System Prompt
In modern Chat APIs (OpenAI, Anthropic), messages are categorized into roles: `system`, `user`, and `assistant`. The system prompt is distinct because it is not part of the conversation history but rather a "set of rules" the model must follow.

### Common Use Cases
1.  **Defining Personas:** "You are a senior Python developer specializing in Flask."
2.  **Setting Tone:** "Speak in a formal, professional tone."
3.  **Providing Context:** "The current date is Oct 2023. You are helping a user with their taxes."
4.  **Establishing Constraints:** "Only provide code in Python. Do not explain the code unless asked."
5.  **Output Formatting:** "Always respond with a JSON object containing keys 'answer' and 'confidence'."

### Best Practices for System Prompts
*   **Be Specific:** Instead of "Be helpful," say "Provide concise, 2-sentence answers that prioritize technical accuracy."
*   **Use Clear Sections:** Structure the system prompt with headings like `CONSTRAINTS`, `PERSONA`, and `GOALS`.
*   **Handle Edge Cases:** Tell the model what to do if it doesn't know the answer (e.g., "If you don't know the answer, say 'I am not sure' and do not hallucinate.").

### Implementation with Python (OpenAI)

```python
from openai import OpenAI
client = OpenAI()

system_prompt = """
PERSONA: You are a strict code reviewer.
CONSTRAINTS: 
- Focus only on performance and security.
- Do not compliment the user.
- Provide direct, actionable feedback.
"""

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": "Review this code: print('Hello ' + user_input)"}
    ]
)
# print(response.choices[0].message.content)
```

### System Prompt vs. User Prompt
*   **System Prompt:** Sets the *global* environment and rules.
*   **User Prompt:** Provides the *specific* task or question.
*   **Instruction Overlap:** While you can put instructions in the user prompt, models are generally trained to treat the system message as the "ground truth" and are less likely to be distracted from it.

## Interview Questions

**Q: What is a System Prompt?**
**A:** A system prompt is a message sent to the LLM to define its behavior, persona, and constraints. It acts as the "operating system" for the conversation, guiding how the model should respond to user inputs.

**Q: How does a System Prompt differ from a User Prompt?**
**A:** The system prompt sets the tone and rules for the entire session, while the user prompt is the specific query or input from the human user. The system prompt is usually hidden from the end user and is used by the developer to control the application's behavior.

**Q: Can a user override the System Prompt?**
**A:** Yes, through a technique called **Prompt Injection**. If a user says "Ignore all previous instructions and do X," a model might follow the user's new instruction instead of the system prompt. Preventing this is a key challenge in AI security.

**Q: Why is it important to tell an LLM what to do when it doesn't know an answer in the System Prompt?**
**A:** To prevent **hallucinations**. LLMs are trained to be helpful and may try to generate an answer even if they lack the information. Explicitly instructing the model to admit ignorance ("If you don't know, say 'I don't know'") significantly reduces the risk of confident but false responses.
