#Python
---
---

# FastAPI

## Summary
**FastAPI** is a modern, fast (high-performance), web framework for building APIs with Python 3.8+ based on standard Python type hints. It is designed to be easy to use, fast to code, and production-ready. Its core is built on two powerful libraries: **Starlette** (for web handling) and **Pydantic** (for data validation).

## Detailed Explanation

### Pydantic Integration
FastAPI uses **Pydantic** to handle data validation, serialization, and JSON Schema generation. When you define a request body using a Pydantic model, FastAPI automatically:
- Validates that the input matches the model's schema.
- Parses and converts types (e.g., string to datetime).
- Generates clear error messages in JSON format if validation fails.
- Includes the model in the auto-generated OpenAPI documentation.

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field
from typing import Optional

app = FastAPI()

class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float = Field(..., gt=0)
    tax: Optional[float] = None

@app.post("/items/")
async def create_item(item: Item):
    return item
```

### Starlette Base
FastAPI is a sub-class of **Starlette**, which means it inherits all of Starlette's features and performance.
- **ASGI Support**: Native support for the Asynchronous Server Gateway Interface, allowing for high-concurrency connections.
- **Middlewares**: Easy integration of CORS, GZip, Static Files, and Session middlewares.
- **WebSocket**: Built-in support for real-time, bidirectional communication.
- **Background Tasks**: Ability to run tasks after returning a response (e.g., sending an email).

### Auto-generated OpenAPI/Swagger
FastAPI automatically generates an **OpenAPI** (formerly Swagger) schema for your entire API.
- **Swagger UI**: Accessible at `/docs`, providing an interactive interface to test your endpoints.
- **ReDoc**: Accessible at `/redoc`, providing a clean, searchable documentation layout.
- **Standardized**: Because it follows the OpenAPI standard, you can use tools like `openapi-generator` to create client SDKs automatically.

### Dependency Injection System
The Dependency Injection (DI) system in FastAPI is one of its most powerful features. Using `Depends()`, you can inject logic into your path operations.
- **Code Reuse**: Share logic like database sessions, authentication, or common query parameters.
- **Hierarchical**: Dependencies can depend on other dependencies, creating a tree of logic.
- **Testability**: Dependencies can be easily overridden during testing.

```python
from fastapi import Depends, HTTPException, status

async def common_parameters(q: str | None = None, skip: int = 0, limit: int = 100):
    return {"q": q, "skip": skip, "limit": limit}

@app.get("/items/")
async def read_items(commons: dict = Depends(common_parameters)):
    return commons
```

### Async Support
FastAPI is designed from the ground up to support `async` and `await`.
- **Async Def**: Path operations defined with `async def` run directly on the event loop, making them perfect for non-blocking I/O operations (like database queries or external API calls).
- **Synchronous Def**: If you define an endpoint with regular `def`, FastAPI runs it in a separate thread pool to ensure it doesn't block the main event loop, maintaining high performance even with synchronous code.

```python
import asyncio

@app.get("/async-data")
async def get_async_data():
    await asyncio.sleep(1)  # Non-blocking sleep
    return {"message": "Data fetched asynchronously"}
```

## Interview Questions

### 1. What is the relationship between Starlette, Pydantic, and FastAPI?
FastAPI is built on top of **Starlette** for web-related functionality (routing, middlewares) and **Pydantic** for data-related functionality (validation, serialization). FastAPI adds the "glue" that connects them, along with features like Dependency Injection and OpenAPI generation.

### 2. When should you use `async def` vs regular `def` in FastAPI?
Use `async def` when your code performs I/O bound operations that support `await` (e.g., `httpx` for API calls or `motor` for MongoDB). Use regular `def` if you are performing CPU-bound tasks or using synchronous libraries (like `requests` or standard `psycopg2`). FastAPI handles synchronous `def` by running them in a thread pool.

### 3. How does Dependency Injection work in FastAPI?
It uses the `Depends()` function. When a path operation is called, FastAPI resolves all dependencies before executing the function. It can also cache the result of a dependency if it's used multiple times within the same request.

### 4. How can you handle Path Parameters and Query Parameters?
Path parameters are part of the URL path (e.g., `/items/{item_id}`), while query parameters are appended to the URL (e.g., `?q=search`). FastAPI identifies them based on the function signature and the path template.

### 5. What are Background Tasks and how do you use them?
`BackgroundTasks` is a class that allows you to schedule a function to run after the response has been sent to the client. This is useful for tasks like sending emails or processing logs that don't need to block the user's response.
