## Summary
LangChain is the most widely used framework for building LLM-powered applications. It provides a modular set of components and a standardized interface for connecting LLMs with external data sources, tools, and long-term memory.

## Detailed Explanation
### **Core Components**
1.  **Model I/O**: Standardized interfaces for Prompts, LLMs (text completion), and Chat Models (messages).
2.  **Data Connection**: Components for loading, transforming, and storing data (Document Loaders, Text Splitters, Embeddings, Vector Stores).
3.  **Chains**: The core logic that "chains" different components together.
    *   **LCEL (LangChain Expression Language)**: A declarative way to compose chains using the pipe operator (`|`), making it easy to stream, batch, and run tasks in parallel.
4.  **Memory**: Allows the model to "remember" previous interactions in a conversation.
5.  **Agents**: Systems that use an LLM to decide which actions to take and in what order.

### **LCEL Example**
```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI

prompt = ChatPromptTemplate.from_template("Tell me a joke about {topic}")
model = ChatOpenAI()
chain = prompt | model

# Invoking the chain
response = chain.invoke({"topic": "bears"})
```

### **LangGraph**
A newer addition to the ecosystem specifically designed for building cyclic, stateful multi-agent systems, overcoming the limitations of linear LangChain chains.

## Interview Questions
*   **Q: What is LCEL in LangChain and why is it useful?**
    *   **A:** LCEL (LangChain Expression Language) is a declarative syntax for building chains. It is useful because it provides first-class support for streaming, async operations, batch processing, and parallel execution out of the box.
*   **Q: What is the difference between an "LLM" and a "ChatModel" in LangChain?**
    *   **A:** An "LLM" (like `OpenAI`) takes a string as input and returns a string. A "ChatModel" (like `ChatOpenAI`) takes a list of structured messages (System, Human, AI) as input and returns a message object.
*   **Q: Why would you use a Vector Store in a LangChain application?**
    *   **A:** To perform efficient similarity searches on large datasets. This is the foundation of RAG, allowing the application to retrieve only the most relevant "chunks" of data to include in the LLM's prompt.
