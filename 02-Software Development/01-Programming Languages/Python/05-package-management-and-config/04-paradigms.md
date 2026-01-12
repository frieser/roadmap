# Paradigms: OOP vs Functional

## Summary
Python is a multi-paradigm language. It supports Object-Oriented Programming (OOP) (everything is an object) and Functional Programming (FP) (first-class functions, pure functions). Most Python code is a hybrid of both.

## Detailed Explanation

### Object-Oriented Programming (OOP)
This is Python's "native" state.
*   **Classes & Objects**: Encapsulate data (attributes) and behavior (methods).
*   **Inheritance**: Sharing logic between classes.
*   **Polymorphism**: Different classes responding to the same method call (Duck Typing).

```python
class Dog:
    def speak(self):
        return "Woof"

class Cat:
    def speak(self):
        return "Meow"

# Polymorphism
def animal_sound(animal):
    print(animal.speak())
```

### Functional Programming (FP)
Python supports FP features but is not a pure FP language (like Haskell).
*   **First-Class Functions**: Functions can be passed as arguments, returned, and assigned to variables.
*   **Immutability**: Python has immutable types (tuple, frozenset), but most are mutable.
*   **Higher-Order Functions**: `map`, `filter`, `reduce`.
*   **Lambdas**: Anonymous functions.

```python
# FP Style: Map/Filter
nums = [1, 2, 3, 4]
squared_evens = list(map(lambda x: x**2, filter(lambda x: x % 2 == 0, nums)))
```

### Pythonic Style (The Hybrid)
Python often prefers List Comprehensions (FP concept) over `map`/`filter`, and simple functions over complex class hierarchies.

```python
# Pythonic replacement for map/filter above
squared_evens = [x**2 for x in nums if x % 2 == 0]
```

## Interview Questions

**Q: Is Python a pure object-oriented language?**
**A:** In Python, "everything is an object" (functions, modules, basic types), so it is deeply object-oriented. However, it does not force you to use classes for everything (unlike Java). You can write procedural scripts or functional code.

**Q: What is "Duck Typing"?**
**A:** "If it walks like a duck and quacks like a duck, it's a duck." Python doesn't check types; it checks behavior. If an object has a `speak()` method, you can call it, regardless of its class inheritance.

**Q: Which paradigm is better in Python?**
**A:** Neither. The best Python code usually mixes them: using Classes to organize state and configuration, and pure Functions for data transformation logic.
