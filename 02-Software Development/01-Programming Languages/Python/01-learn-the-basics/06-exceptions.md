# Exceptions

## Summary
Exceptions handle errors gracefully at runtime. Python uses `try`, `except`, `else`, and `finally` blocks. Python 3.11 introduced `ExceptionGroup` for handling multiple exceptions simultaneously.

## Detailed Explanation

### Basic Structure
*   **`try`**: Block of code to test for errors.
*   **`except`**: Block of code to handle the error.
*   **`else`**: Runs if no errors were raised.
*   **`finally`**: Runs regardless of the result (cleanup).

```python
try:
    print(x)
except NameError:
    print("Variable x is not defined")
except Exception as e:
    print(f"Something else went wrong: {e}")
else:
    print("Nothing went wrong")
finally:
    print("Cleanup complete")
```

### Raising Exceptions
Use the `raise` keyword.

```python
x = -1
if x < 0:
    raise ValueError("Number must be positive")
```

### Exception Groups (Python 3.11+)
Allows raising and handling multiple exceptions at once, useful in concurrent programming or complex validation.

```python
try:
    raise ExceptionGroup("Errors", [
        ValueError("Invalid value"),
        TypeError("Invalid type")
    ])
except* ValueError as e:
    print(f"Caught ValueErrors: {e.exceptions}")
except* TypeError as e:
    print(f"Caught TypeErrors: {e.exceptions}")
```

Note the `except*` syntax, which allows handling parts of the exception group.

## Interview Questions

**Q: What is the difference between `except Exception` and `except BaseException`?**
**A:** `Exception` is the base class for all non-system-exiting exceptions. `BaseException` includes system events like `KeyboardInterrupt` and `SystemExit`. Catching `BaseException` is generally bad practice as it prevents the user from exiting the program with Ctrl+C.

**Q: When does the `else` block run in error handling?**
**A:** The `else` block runs only if the `try` block completes *without* raising any exceptions.

**Q: What is the purpose of `finally`?**
**A:** To execute cleanup code (like closing files or database connections) that must run whether or not an exception occurred.
