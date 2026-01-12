#Python
---
---

## Summary
Sanic is a high-performance Python web framework and server built on top of `asyncio`. It is designed for speed and scale, leveraging the `async/await` syntax to provide non-blocking request handling. With a syntax heavily inspired by Flask, Sanic offers a familiar developer experience while delivering performance comparable to Go and Node.js by utilizing `uvloop` as its event loop.

## Detailed Explanation

### Async-first Design
Sanic was built from the ground up to be asynchronous. Unlike traditional frameworks that added async support later, Sanic requires handlers to be `async def` (though it can support sync ones). This design allows the server to handle thousands of concurrent connections without blocking on I/O operations, making it ideal for microservices, WebSockets, and real-time applications.

### Flask-like Syntax
One of Sanic's biggest advantages is its low barrier to entry for Python developers. It uses a decorator-based routing system similar to Flask:

```python
from sanic import Sanic, response

app = Sanic("MyHelloWorldApp")

@app.get("/")
async def hello_world(request):
    return response.json({"hello": "world"})

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8000)
```

### Speed and uvloop
Sanic is consistently ranked among the fastest Python web frameworks. Its performance comes from:
1.  **uvloop**: An extremely fast implementation of the `asyncio` event loop built in Cython and based on `libuv`. Sanic uses `uvloop` by default if it is installed.
2.  **Minimal Overhead**: The core framework is lean, focusing on high-speed request/response cycles.
3.  **Fast HTTP Parser**: It uses a high-performance C-based HTTP parser.

### Request and Response Objects
Every route handler receives a `Request` object and must return a `Response` object.

*   **Request**: Contains information like headers, query parameters (`request.args`), JSON body (`request.json`), and files.
*   **Response**: Sanic provides a `response` module to easily create various types of responses:
    ```python
    from sanic import response

    # JSON response
    return response.json({"status": "ok"})

    # Text response
    return response.text("Hello!")

    # Redirect
    return response.redirect("/login")
    ```

### Listeners (Server Hooks)
Listeners allow you to run code at specific stages of the server's lifecycle. This is useful for setting up database connections or cleaning up resources.

```python
@app.listener("before_server_start")
async def setup_db(app, loop):
    app.ctx.db = await create_db_pool()

@app.listener("after_server_stop")
async def close_db(app, loop):
    await app.ctx.db.close()
```
Available hooks include: `before_server_start`, `after_server_start`, `before_server_stop`, and `after_server_stop`.

## Interview Questions

**Q: What is Sanic and why would you choose it over Flask?**
**A:** Sanic is an asynchronous Python web framework designed for high performance. You would choose Sanic over Flask when you need to handle a high volume of concurrent connections or I/O-bound tasks efficiently, as Sanic's non-blocking nature allows it to handle many more requests per second than the synchronous Flask.

**Q: What is uvloop and how does it relate to Sanic?**
**A:** `uvloop` is a high-performance replacement for the standard `asyncio` event loop, built on `libuv`. Sanic uses `uvloop` by default to achieve significantly higher throughput and lower latency, making it one of the fastest web frameworks in the Python ecosystem.

**Q: How do you handle background tasks in Sanic?**
**A:** Sanic provides `app.add_task()` to schedule background tasks that run independently of the request/response cycle. These tasks are managed by the Sanic worker and can be used for long-running processes that shouldn't block the user's request.

**Q: Explain Sanic's "listeners".**
**A:** Listeners are hooks that trigger functions at specific points in the server lifecycle, such as before the server starts or after it stops. They are essential for resource management, like initializing database pools or closing network connections.
