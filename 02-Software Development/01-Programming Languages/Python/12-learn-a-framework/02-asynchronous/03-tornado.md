#Python
---
---

## Summary
Tornado is a Python web framework and asynchronous networking library, originally developed at FriendFeed (acquired by Facebook). By using non-blocking network I/O, Tornado can scale to tens of thousands of open connections, making it ideal for applications requiring long-lived connections like WebSockets or long polling. Unlike most Python web frameworks that are based on WSGI, Tornado runs its own event loop and handles concurrency through a single-threaded, non-blocking approach.

## Detailed Explanation

### History and the C10k Problem
Tornado was born out of the need to solve the **C10k problem**—the challenge of handling 10,000 concurrent connections on a single server. Traditional web servers (like Apache) used a thread-per-connection or process-per-connection model, which consumes significant memory and CPU due to context switching when scaled. FriendFeed developed Tornado to handle their real-time updates efficiently using an event-driven architecture, similar to Node.js or Nginx.

### Non-blocking I/O Loop
At the heart of Tornado is the `IOLoop`. It is an event loop that manages all network activity.
- **Mechanism**: It uses efficient system calls like `epoll` (Linux) or `kqueue` (macOS/BSD) to monitor thousands of file descriptors.
- **Single-Threaded**: By running in a single thread, it avoids the overhead of locking and context switching between threads.
- **Non-blocking**: Every operation (disk I/O, network requests, DB queries) must be non-blocking. If a handler performs a blocking operation (like a synchronous `time.sleep()` or a heavy computation), it blocks the entire event loop, preventing other requests from being processed.

### WebSockets Strengths
Tornado has long been a leader in the Python ecosystem for WebSocket support. Because it is built on an asynchronous foundation, it can maintain stateful, bidirectional connections with very low overhead.
- **Native Support**: Unlike Django or Flask (which often require extensions like Channels or Gevent), Tornado's `WebSocketHandler` is a first-class citizen.
- **Scalability**: It is particularly strong for chat applications, real-time dashboards, and collaborative tools where many users stay connected for long periods.

### Evolution: Coroutines vs. Async/Await
Tornado's approach to asynchronous programming has evolved significantly over the years:

1.  **Callback Style**: The earliest versions relied on callbacks and the `@tornado.web.asynchronous` decorator. This often led to "callback hell."
2.  **Generators and `yield`**: The `tornado.gen` module introduced coroutines using Python's `yield` keyword and the `@gen.coroutine` decorator. This allowed asynchronous code to look sequential.
3.  **Native `async/await`**: Modern Tornado (version 5.0+) fully embraces Python 3's native `async` and `await` syntax. This is now the recommended way to write Tornado applications.

#### Modern Async Example (async/await)
```python
import tornado.ioloop
import tornado.web
from tornado.httpclient import AsyncHTTPClient

class MainHandler(tornado.web.RequestHandler):
    async def get(self):
        # Asynchronous non-blocking HTTP fetch
        http_client = AsyncHTTPClient()
        response = await http_client.fetch("https://api.github.com/events", 
                                          headers={'User-Agent': 'Tornado'})
        self.write(f"Fetched GitHub events. Length: {len(response.body)}")

def make_app():
    return tornado.web.Application([
        (r"/", MainHandler),
    ])

if __name__ == "__main__":
    app = make_app()
    app.listen(8888)
    print("Server running on http://localhost:8888")
    tornado.ioloop.IOLoop.current().start()
```

#### WebSocket Example
```python
from tornado.websocket import WebSocketHandler

class EchoWebSocket(WebSocketHandler):
    def open(self):
        print("WebSocket opened")

    def on_message(self, message):
        # Bidirectional communication
        self.write_message(f"You sent: {message}")

    def on_close(self):
        print("WebSocket closed")
```

## Interview Questions

**Q: What is the C10k problem and how does Tornado address it?**
**A:** The C10k problem refers to handling 10,000 concurrent connections. Tornado addresses this by using a non-blocking, event-driven I/O loop (IOLoop) instead of a thread-per-connection model. This allows a single process to manage thousands of open connections with minimal memory and CPU overhead.

**Q: Why should you avoid using `time.sleep()` in a Tornado RequestHandler?**
**A:** Since Tornado is single-threaded and relies on an event loop, `time.sleep()` is a blocking call that halts the entire thread. This prevents the `IOLoop` from processing any other events or requests, effectively freezing the entire server for the duration of the sleep. You should use `await tornado.gen.sleep()` or similar non-blocking timers instead.

**Q: How does Tornado differ from traditional WSGI frameworks like Flask or Django?**
**A:** WSGI is a synchronous standard where each request is handled by a worker thread/process that blocks until the response is ready. Tornado is built on an asynchronous networking library and manages its own event loop, allowing it to handle long-lived connections (like WebSockets) natively, which is difficult or inefficient in standard WSGI.

**Q: What happened to the `@gen.coroutine` decorator in modern Tornado?**
**A:** While still available for backward compatibility, it has been largely superseded by Python's native `async` and `await` syntax. Native coroutines are more efficient and provide better integration with the standard library's `asyncio`.
