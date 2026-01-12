## Summary
Tool use (or Function Calling) is the mechanism that allows LLMs to interact with external systems. Instead of just generating text, the model generates a structured request (usually JSON) that identifies a specific function to call and the arguments to pass to it.

## Detailed Explanation
### **The Function Calling Workflow**
1.  **Definition**: The developer provides the model with a list of available tools, described using a JSON Schema.
2.  **Selection**: The LLM decides which tool is appropriate for the user's query and generates the JSON arguments.
3.  **Execution**: The application (not the LLM) executes the function and gets a result.
4.  **Integration**: The result is passed back to the LLM as a "Tool" or "Function" role message.
5.  **Final Response**: The LLM uses the tool's output to generate the final answer for the user.

### **JSON Schema Example**
Tools are typically defined using Pydantic in Python for type safety:
```python
from pydantic import BaseModel, Field

class GetWeather(BaseModel):
    """Get the current weather in a given location."""
    location: str = Field(description="The city and state, e.g. San Francisco, CA")
    unit: str = Field(default="celsius", enum=["celsius", "fahrenheit"])
```

### **Handling Tool Failures**
-   **Retry Logic**: If a tool returns an error, the agent can be prompted to try again with different parameters.
-   **Fallback**: If a specific API is down, the agent might switch to a different tool (e.g., from Google Search to Bing).
-   **Human-in-the-Loop**: For sensitive actions (e.g., sending an email), the agent can be configured to pause and wait for user approval.

## Interview Questions
*   **Q: How does an LLM "know" how to use a tool?**
    *   **A:** It doesn't "know" in the traditional sense; it is provided with a system prompt and a JSON description (schema) of the tool. Modern models (like GPT-4o or Claude 3.5) are specifically fine-tuned to recognize when a query matches a tool description and to output valid JSON.
*   **Q: What is the purpose of "Tool Choice" (or `function_call`) in the OpenAI API?**
    *   **A:** It allows the developer to force the model to use a specific tool (`required`), prevent it from using any tools (`none`), or let the model decide (`auto`).
*   **Q: What is "Type Narrowing" in the context of tool outputs?**
    *   **A:** It refers to ensuring that the data returned by a tool is correctly parsed and validated (often using Pydantic) before being sent back to the LLM, preventing the model from hallucinating based on malformed data.
