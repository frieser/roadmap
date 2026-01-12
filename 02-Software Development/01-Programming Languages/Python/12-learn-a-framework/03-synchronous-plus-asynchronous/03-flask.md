#Python
---
---

## Summary
Flask is a lightweight WSGI web application framework in Python, known for its **microframework** philosophy. It provides the essentials for web development—routing, request handling, and templating—while remaining agnostic about database choice, form validation, and authentication. This minimalist core makes it highly flexible and easy to learn, allowing developers to scale their applications by adding extensions from a vast ecosystem.

## Detailed Explanation

### Microframework Philosophy
Flask's core design principle is to keep the framework simple but extensible. It does not make decisions for the developer (e.g., no built-in ORM or admin panel), which avoids "opinionated" overhead. You only add what you need via **Extensions** (like Flask-SQLAlchemy, Flask-WTF, or Flask-Migrate).

### Application and Request Contexts
Flask uses "context locals" to make certain variables globally accessible within a specific context without having to pass them between functions explicitly.

*   **Application Context**: Keeps track of application-level data during a request, a CLI command, or other activity.
    *   `current_app`: The application instance for the active context.
    *   `g`: A temporary object for storing data during a single request (e.g., a database connection).
*   **Request Context**: Keeps track of request-level data during a request.
    *   `request`: The current request object (HTTP method, URL, headers, etc.).
    *   `session`: A dictionary-like object for storing data across requests for a specific user.

### Blueprints
**Blueprints** are a mechanism for organizing application components into reusable and modular structures. Instead of registering routes directly to the `app` instance, you register them to a Blueprint, which is then registered to the application in a factory function.

```python
from flask import Blueprint

# Define a blueprint
auth_bp = Blueprint('auth', __name__)

@auth_bp.route('/login')
def login():
    return "Login Page"

# In the main app file:
# app.register_blueprint(auth_bp, url_prefix='/auth')
```

### Async Routes in Flask 2.0+
Since version 2.0, Flask natively supports `async` and `await` for view functions, allowing for non-blocking I/O operations within a traditionally synchronous framework.

```python
from flask import Flask
import asyncio

app = Flask(__name__)

@app.route("/async-data")
async def get_async_data():
    # Simulate an external API call or long I/O operation
    await asyncio.sleep(1)
    return {"message": "Data retrieved asynchronously!"}
```

**Key Technical Notes for Async Flask:**
*   **Execution**: On a standard WSGI server, Flask runs the `async` view in a separate thread using `asgiref.sync.ensure_async`.
*   **Performance**: While it simplifies integration with async libraries (like `httpx`), true async performance gains (handling thousands of concurrent connections) usually require an ASGI server or a dedicated ASGI framework like FastAPI.

## Interview Questions

1.  **What does it mean that Flask is a "microframework"?**
    It means the core is kept small and extensible. It provides the basics but relies on extensions for features like database integration or form handling.
2.  **What is the difference between `g` and `session` in Flask?**
    `g` is for storing data for the duration of a **single request**, while `session` stores data across **multiple requests** for a specific user using signed cookies.
3.  **How do Blueprints help in organizing a Flask application?**
    They allow for modularity by grouping related routes and logic, making the code easier to maintain, test, and reuse.
4.  **Can Flask handle asynchronous requests?**
    Yes, since Flask 2.0. You can define routes with `async def`. It's useful for calling other async APIs or performing non-blocking I/O.
5.  **Explain the difference between the Application Context and the Request Context.**
    The Application Context (`current_app`, `g`) handles app-wide data during an operation, whereas the Request Context (`request`, `session`) handles data specific to the current HTTP request.
