#Python
---
---

## Summary
Python classes are the fundamental building blocks of Object-Oriented Programming (OOP) in the language. They serve as blueprints for creating objects, which are instances of those classes. Classes encapsulate data (attributes) and behavior (methods) into a single entity, promoting code reuse, modularity, and better organization.

## Detailed Explanation

### Class Definition
A class is defined using the `class` keyword followed by the class name (by convention using PascalCase) and a colon. The body of the class contains attribute definitions and methods.

```python
class Person:
    """A simple class to represent a person."""
    pass

# Creating an instance (object)
p = Person()
```

### `__init__` constructor
The `__init__` method is a special "dunder" (double underscore) method that Python automatically calls when you create a new instance of a class. Its primary purpose is to initialize the object's attributes with initial values.

```python
class User:
    def __init__(self, username, email):
        self.username = username
        self.email = email
```

### 'self' keyword
The `self` parameter is a reference to the current instance of the class. It allows methods to access the attributes and other methods of the specific object being worked with. 

- It must be the **first parameter** of any instance method.
- While you could name it anything, `self` is a strong community convention that should always be followed.

```python
class Counter:
    def __init__(self):
        self.count = 0
    
    def increment(self):
        self.count += 1  # Accessing attribute via self
```

### Class vs Instance variables
Understanding the scope of variables is crucial in Python OOP:

1.  **Instance Variables**: Variables that are unique to each instance. They are defined inside methods (usually `__init__`) using `self`.
2.  **Class Variables**: Variables that are shared by all instances of a class. They are defined directly within the class body, outside any methods.

```python
class Circle:
    pi = 3.14159  # Class variable (shared by all circles)

    def __init__(self, radius):
        self.radius = radius  # Instance variable (unique to this circle)

c1 = Circle(5)
c2 = Circle(10)

print(c1.pi)      # 3.14159
print(Circle.pi)  # 3.14159
```

### Data Classes (Briefly)
Introduced in Python 3.7, the `@dataclass` decorator automates the creation of common methods like `__init__`, `__repr__`, and `__eq__` for classes that primarily exist to store data. This significantly reduces boilerplate code.

```python
from dataclasses import dataclass

@dataclass
class Product:
    name: str
    price: float
    quantity: int = 0

p = Product("Laptop", 999.99, 10)
print(p)  # Output: Product(name='Laptop', price=999.99, quantity=10)
```

## Interview Questions

1. **Q: What is the difference between a class and an object?**
   **A:** A class is a blueprint or template that defines the structure and behavior for a type of object. An object is a specific instance of a class, created using the class blueprint, with its own set of data.

2. **Q: What is the purpose of the `self` keyword in Python?**
   **A:** `self` represents the instance of the object itself. It allows methods to access and modify the attributes of the specific object that called the method, distinguishing it from other instances of the same class.

3. **Q: How do class variables differ from instance variables?**
   **A:** Class variables are shared by all instances of a class, while instance variables are unique to each instance. Changes to a class variable affect all instances (if accessed via the class), whereas changes to an instance variable only affect that specific object.

4. **Q: What are "dunder" methods in Python?**
   **A:** Dunder methods (short for "double underscore") are special methods like `__init__`, `__str__`, and `__repr__` that Python uses to implement specific behaviors, such as object initialization, string representation, and operator overloading. They are also called "magic methods."

5. **Q: When would you use a Data Class?**
   **A:** Data classes are ideal when you need a class primarily to store data. They reduce boilerplate code by automatically generating methods like `__init__`, `__repr__`, and `__eq__` based on type hints, making the code cleaner and easier to maintain.
