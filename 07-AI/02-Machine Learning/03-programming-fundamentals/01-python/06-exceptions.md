---
tags: ['ai', 'roadmap', 'python']
---

# Python Exception Handling

## Summary
Exception handling in Python is a structured way to manage runtime errors, preventing programs from crashing and allowing for graceful recovery or cleanup. It utilizes a dedicated syntax (`try`, `except`, `else`, `finally`) to isolate error-prone code and define specific responses to different types of failures. For AI Engineers, this is particularly critical when dealing with unpredictable external factors like data quality issues, API timeouts (e.g., OpenAI/Anthropic), and hardware constraints (e.g., CUDA errors).

## Detailed Explanation

### 1. The Core Syntax: `try` and `except`
The fundamental way to handle exceptions is by wrapping code that might fail in a `try` block. If an error occurs, the execution jumps to the `except` block.

```python
try:
    # Code that might raise an exception
    result = 10 / 0
except ZeroDivisionError:
    # Code that runs if the specific exception occurs
    print("Error: Cannot divide by zero.")
```

### 2. Handling Multiple Exceptions
You can catch different types of exceptions separately to provide specific handling for each.

```python
try:
    with open('data.csv', 'r') as f:
        data = f.read()
    value = int(data)
except FileNotFoundError:
    print("The file was not found.")
except ValueError:
    print("The file content is not a valid integer.")
except Exception as e:
    # Catching any other unexpected exceptions
    print(f"An unexpected error occurred: {e}")
```

### 3. The `else` and `finally` Blocks
*   **`else`**: Runs ONLY if the code in the `try` block executed without any exceptions. It is useful for logic that depends on the successful completion of the `try` block but shouldn't be "protected" by the `try`.
*   **`finally`**: ALWAYS runs, regardless of whether an exception was raised or caught. This is ideal for cleaning up resources like closing files, network connections, or freeing GPU memory.

```python
try:
    f = open('model_weights.bin', 'rb')
except IOError:
    print("Could not open file")
else:
    print("File opened successfully")
    # Process file...
finally:
    if 'f' in locals():
        f.close()
    print("Cleanup complete.")
```

### 4. Raising Exceptions
You can manually trigger an exception using the `raise` keyword. This is useful for enforcing business logic or data validation.

```python
def validate_input(value):
    if value < 0:
        raise ValueError("Input must be a non-negative number.")
    return value
```

### 5. Custom Exceptions
In large AI projects, it's often helpful to define your own exception classes to represent specific domain errors.

```python
class ModelInferenceError(Exception):
    """Exception raised for errors during model inference."""
    def __init__(self, model_name, message):
        self.model_name = model_name
        self.message = message
        super().__init__(self.message)

# Usage
raise ModelInferenceError("ResNet-50", "Failed to load weights into GPU memory.")
```

### 6. AI-Specific Use Cases
*   **API Resilience**: Handling `RateLimitError` or `Timeout` when calling LLM APIs.
*   **Data Validation**: Catching `KeyError` or `TypeError` during data preprocessing in Pandas.
*   **Hardware Management**: Catching `RuntimeError` from PyTorch when a GPU is out of memory (OOM).

## Interview Questions

**Q: What is the difference between `except Exception as e` and `except:`?**
**A:** `except:` catches every single exception, including `SystemExit`, `KeyboardInterrupt`, and `GeneratorExit`, which can make it hard to stop a program (e.g., via Ctrl+C). `except Exception as e` catches all exceptions that inherit from the `Exception` class, which covers almost all runtime errors while still allowing system-level signals to pass through.

**Q: Why is it considered a best practice to use specific exceptions instead of a broad `except`?**
**A:** Specific exceptions ensure that you are only handling errors you expect and know how to fix. A broad `except` can hide bugs or unexpected behavior (like a typo in a variable name) that should instead be allowed to crash the program so they can be identified and fixed.

**Q: Explain the execution flow when an exception occurs in the `try` block and there is a `finally` block.**
**A:** When an exception occurs, the remaining code in the `try` block is skipped. Python looks for a matching `except` block. If found, it executes the handler. Then, the `finally` block executes. If no matching `except` is found, the exception is temporarily "paused," the `finally` block executes, and then the exception is re-raised to the caller.

**Q: How do you access the error message of a caught exception?**
**A:** By using the `as` keyword in the `except` statement: `except ValueError as e:`. The variable `e` (or any name you choose) will hold the exception instance, and you can print it or access its attributes.

**Q: What happens if an exception is raised inside an `except` block?**
**A:** If a new exception is raised inside an `except` block, the original exception is "chained" to the new one (visible via the `__context__` or `__cause__` attributes). This is useful for translating low-level errors into high-level domain errors while preserving the original traceback.
