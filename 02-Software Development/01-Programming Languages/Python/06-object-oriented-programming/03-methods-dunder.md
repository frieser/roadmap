#Python
---
---

## Summary
Python offers three types of methods to define behavior within classes: **Instance**, **Class**, and **Static** methods, each serving different purposes based on their access to instance or class state. Additionally, **Dunder (Double Underscore) methods**, also known as **Magic Methods**, allow developers to tap into Python's internal protocols. These methods enable features like object representation, comparison, and operator overloading, making custom objects behave like built-in Python types.

## Detailed Explanation

### 1. Instance vs Class vs Static Methods

| Method Type | Decorator | First Argument | Purpose |
| :--- | :--- | :--- | :--- |
| **Instance** | None | `self` | Access/Modify instance state and class state. |
| **Class** | `@classmethod` | `cls` | Access/Modify class state. Often used for factory methods. |
| **Static** | `@staticmethod` | None | Utility functions that don't need access to state but belong to the class namespace. |

#### Example Implementation
```python
class Connection:
    port = 80  # Class variable

    def __init__(self, host):
        self.host = host  # Instance variable

    # Instance Method
    def get_info(self):
        return f"Connecting to {self.host} on port {self.port}"

    # Class Method
    @classmethod
    def set_port(cls, new_port):
        cls.port = new_port

    # Static Method
    @staticmethod
    def is_valid_ip(ip):
        return ip.count('.') == 3

# Usage
conn = Connection("localhost")
print(conn.get_info())         # Connecting to localhost on port 80
Connection.set_port(443)       # Changes port for all instances
print(Connection.is_valid_ip("127.0.0.1")) # True
```

---

### 2. Magic Methods (Dunder Methods)

Magic methods are special methods with double underscores at the beginning and end. They are invoked automatically by Python in specific contexts.

#### Object Representation: `__str__` vs `__repr__`
- `__str__`: Informal, user-friendly string representation. Used by `print()` and `str()`.
- `__repr__`: Formal, unambiguous representation for debugging. Used by `repr()`. Ideally, it should look like the command used to create the object.

```python
class Product:
    def __init__(self, name, price):
        self.name = name
        self.price = price

    def __str__(self):
        return f"{self.name} (${self.price})"

    def __repr__(self):
        return f"Product(name='{self.name}', price={self.price})"

p = Product("Laptop", 1200)
print(str(p))   # Laptop ($1200)
print(repr(p))  # Product(name='Laptop', price=1200)
```

#### Other Common Magic Methods
- `__len__`: Returns the "length" of an object when `len()` is called.
- `__eq__`: Defines equality logic for `==`.
- `__call__`: Makes an instance callable like a function.

```python
class Team:
    def __init__(self, members):
        self.members = members

    def __len__(self):
        return len(self.members)

    def __eq__(self, other):
        return self.members == other.members

    def __call__(self, *args):
        print(f"Team is active! Args: {args}")

t1 = Team(["Alice", "Bob"])
print(len(t1))   # 2
t1("Go!")        # Team is active! Args: ('Go!',)
```

---

### 3. Operator Overloading

Operator overloading allows custom objects to use standard operators (`+`, `-`, `*`, `<`, etc.).

| Operator | Magic Method |
| :--- | :--- |
| `+` | `__add__(self, other)` |
| `-` | `__sub__(self, other)` |
| `*` | `__mul__(self, other)` |
| `<` | `__lt__(self, other)` |
| `==` | `__eq__(self, other)` |

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)

    def __repr__(self):
        return f"Vector({self.x}, {self.y})"

v1 = Vector(1, 2)
v2 = Vector(3, 4)
print(v1 + v2)  # Vector(4, 6)
```

## Interview Questions

1. **What is the main difference between `@classmethod` and `@staticmethod`?**
   - `@classmethod` receives the class (`cls`) as an implicit first argument, allowing it to access class variables and factory methods. `@staticmethod` receives no implicit arguments and behaves like a regular function scoped to the class.

2. **When should you implement `__repr__` instead of `__str__`?**
   - You should always implement `__repr__` first, as it is the fallback for `__str__`. `__repr__` is for developers (debugging), while `__str__` is for end-users.

3. **What does the `__call__` method do?**
   - It allows an instance of a class to be called as if it were a function. This is useful for creating stateful functions or decorators.

4. **How do you implement "less than" comparison for a custom object?**
   - By defining the `__lt__(self, other)` magic method. Using the `@functools.total_ordering` decorator can then automatically fill in other comparison methods like `__le__`, `__gt__`, etc.

5. **Why is `self` used as the first argument in instance methods?**
   - `self` is a reference to the current instance of the class. It allows the method to access and modify the specific object's data. Note that `self` is a convention, not a keyword (though strongly recommended).
