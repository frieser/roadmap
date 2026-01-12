#Python
---
---
## Summary
Asynchronous programming in Python, primarily driven by the `asyncio` library, is a concurrency model that allows multiple tasks to run cooperatively on a single thread. Unlike multi-threading, which relies on the OS to switch contexts, `asyncio` uses an **Event Loop** to manage tasks. It is particularly effective for **I/O-bound** operations (network requests, database queries, file I/O) where the CPU would otherwise sit idle waiting for external resources.

## Detailed Explanation

### 1. The Event Loop
The **Event Loop** is the central execution mechanism in `asyncio`. It runs in a loop, monitors events (like network I/O completion), and executes the corresponding tasks. Only one task runs at any given time on the loop, achieving concurrency through cooperative multitasking—tasks voluntarily yield control back to the loop using the `await` keyword.

### 2. async/await Syntax
- `async def`: Defines a function as a **coroutine**.
- `await`: Yields control back to the event loop, pausing the coroutine until the awaited object (another coroutine, Task, or Future) completes.

### 3. Coroutines vs Tasks vs Futures
- **Coroutine**: A function defined with `async def`. Calling it doesn't execute the code; it returns a coroutine object.
- **Task**: A wrapper for a coroutine that schedules it to run on the event loop. Created via `asyncio.create_task(coro)`. Tasks allow for concurrent execution.
- **Future**: A low-level object representing a result that hasn't been computed yet. Tasks are a subclass of Futures.

### 4. aiohttp Example
The `requests` library is synchronous and blocks the event loop. For asynchronous HTTP requests, `aiohttp` is the standard choice.

```python
import aiohttp
import asyncio

async def fetch_url(session, url):
    async with session.get(url) as response:
        status = response.status
        # await is necessary to read the body without blocking
        text = await response.text()
        print(f"URL: {url}, Status: {status}, Length: {len(text)}")
        return text

async def main():
    urls = [
        "https://www.python.org",
        "https://www.google.com",
        "https://github.com"
    ]
    
    async with aiohttp.ClientSession() as session:
        # Schedule all requests concurrently
        tasks = [fetch_url(session, url) for url in urls]
        # Wait for all tasks to complete
        results = await asyncio.gather(*tasks)
        print(f"Fetched {len(results)} pages")

if __name__ == "__main__":
    asyncio.run(main())
```

### 5. Blocking Code Traps
The "Golden Rule" of `asyncio` is: **Never block the event loop.**
- **`time.sleep()`**: Blocks the entire thread. Use `await asyncio.sleep()` instead.
- **CPU-bound operations**: Long-running calculations block the loop. Use `loop.run_in_executor()` with a `ProcessPoolExecutor` to offload them.
- **Synchronous I/O**: Libraries like `requests` or standard `open()` are blocking. Use `aiohttp` or `aiofiles`.

## Interview Questions

1. **What is the difference between Concurrency and Parallelism in Python?**
   - Concurrency (asyncio) is about *dealing* with many things at once (task switching on one thread). Parallelism (multiprocessing) is about *doing* many things at once (multiple CPUs).

2. **What happens if you use `time.sleep(5)` inside an `async` function?**
   - It blocks the entire event loop for 5 seconds. No other coroutines or tasks will progress during this time.

3. **How do you run a synchronous, CPU-bound function without blocking the event loop?**
   - Use `asyncio.to_thread()` (Python 3.9+) or `loop.run_in_executor()`.

4. **What is the purpose of `asyncio.gather()`?**
   - It allows you to run multiple awaitables concurrently and wait for all of them to finish, returning a list of their results.

5. **Why should you use `asyncio.create_task()` instead of just awaiting coroutines?**
   - Awaiting a coroutine directly (`await coro()`) runs it sequentially. `create_task(coro())` schedules it to start immediately, allowing other code to run concurrently until the task result is actually needed.
