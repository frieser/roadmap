#Python
---
---

## Summary

The `threading` module in Python provides a way to run multiple operations concurrently within a single process. It allows for the creation and management of threads, which are small units of execution that share the same memory space. Due to the **Global Interpreter Lock (GIL)**, Python threads are best suited for **I/O-bound tasks** (like network requests or file operations) rather than CPU-bound tasks, as only one thread can execute Python bytecode at a time.

## Detailed Explanation

### The Thread Class
The `threading.Thread` class is the primary way to create a new thread. You can either pass a callable to the `target` argument or subclass `Thread` and override the `run()` method.

```python
import threading
import time

def task(name):
    print(f"Task {name} starting...")
    time.sleep(2)
    print(f"Task {name} finished.")

# Create and start a thread
t = threading.Thread(target=task, args=("A",), daemon=True)
t.start()

# Wait for thread to finish
t.join()
```

- **`start()`**: Begins thread execution.
- **`join()`**: Blocks the calling thread until the thread whose `join()` method is called terminates.
- **`daemon`**: If `True`, the thread will exit automatically when the main program exits.

### I/O Bound Benefits
In I/O-bound programs, the CPU spends most of its time waiting for external events (network, disk, user input). When a thread hits an I/O operation, it releases the GIL, allowing other threads to run. This makes `threading` highly effective for improving the performance of network scrapers, API clients, and database-heavy applications.

### Race Conditions
A race condition occurs when multiple threads attempt to access and modify a shared resource simultaneously, leading to unpredictable results.

```python
import threading

counter = 0

def increment():
    global counter
    for _ in range(100000):
        counter += 1  # Not atomic!

threads = [threading.Thread(target=increment) for _ in range(10)]
for t in threads: t.start()
for t in threads: t.join()

print(f"Final counter: {counter}")  # Likely less than 1,000,000
```

### Synchronization Primitives

#### Lock
A basic synchronization primitive that can be in either "locked" or "unlocked" state. Only one thread can hold the lock at a time.

```python
lock = threading.Lock()

with lock:
    # Critical section
    counter += 1
```

#### RLock (Reentrant Lock)
A lock that can be acquired multiple times by the same thread. It is useful for recursive functions or nested calls within the same thread.

```python
rlock = threading.RLock()

def recursive_func(n):
    with rlock:
        if n > 0:
            recursive_func(n - 1)
```

#### Semaphore
A semaphore manages an internal counter which is decremented by each `acquire()` call and incremented by each `release()` call. It is used to limit access to a resource with limited capacity (e.g., a connection pool).

```python
# Allow only 3 threads at a time
semaphore = threading.Semaphore(3)

with semaphore:
    # Access limited resource
    pass
```

#### Event
One of the simplest mechanisms for communication between threads: one thread signals an event and other threads wait for it.

```python
event = threading.Event()

def waiter():
    print("Waiting for event...")
    event.wait()
    print("Event received!")

threading.Thread(target=waiter).start()
time.sleep(3)
event.set()  # Signal the event
```

## Interview Questions

### Q: What is the Global Interpreter Lock (GIL) and how does it impact threading?
**A:** The GIL is a mutex that protects access to Python objects, preventing multiple threads from executing Python bytecodes at once. This means that in CPython, multi-threading cannot achieve true parallelism for CPU-bound tasks. However, it works well for I/O-bound tasks because the GIL is released during blocking I/O operations.

### Q: Difference between `threading` and `multiprocessing`?
**A:** 
- **Threading**: Shares the same memory space, lightweight, affected by the GIL (best for I/O-bound).
- **Multiprocessing**: Each process has its own memory space and Python interpreter, bypasses the GIL (best for CPU-bound), but has higher overhead and requires IPC (Inter-Process Communication) to share data.

### Q: When would you use a `Semaphore` instead of a `Lock`?
**A:** A `Lock` allows only one thread to access a resource at a time. A `Semaphore` is used when you want to allow a specific number of concurrent accesses (e.g., limiting the number of simultaneous connections to a database to 5).

### Q: What is a daemon thread?
**A:** A daemon thread is a "background" thread that does not prevent the Python program from exiting. If only daemon threads are left running, the program will terminate. Non-daemon threads will keep the program alive until they complete.

### Q: Why is `counter += 1` not thread-safe in Python?
**A:** Even though it's a single line of Python code, it is compiled into multiple bytecode instructions: `LOAD_GLOBAL`, `LOAD_CONST`, `BINARY_ADD`, `STORE_GLOBAL`. A thread switch can occur between any of these steps, leading to lost updates if another thread modifies the global value in the meantime.