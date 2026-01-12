#Python
---
---

## Summary
Inheritance is a fundamental pillar of Object-Oriented Programming (OOP) that allows a class (known as a **subclass** or **child class**) to acquire the attributes and methods of another class (known as a **superclass** or **parent class**). This promotes code reusability and establishes a hierarchical relationship between classes. Python is unique among many popular languages for its robust support for **Multiple Inheritance** and its sophisticated **Method Resolution Order (MRO)** algorithm.

## Detailed Explanation

### Single vs Multiple Inheritance
*   **Single Inheritance**: A subclass inherits from a single parent class. This is the simplest form of inheritance and forms a linear hierarchy.
    ```python
    class Animal:
        def speak(self):
            print("Animal speaks")

    class Dog(Animal):
        def speak(self):
            print("Bark!")
    ```
*   **Multiple Inheritance**: A subclass can inherit from multiple parent classes. This allows a class to combine functionalities from various sources.
    ```python
    class Flyer:
        def fly(self):
            print("Flying...")

    class Swimmer:
        def swim(self):
            print("Swimming...")

    class Duck(Flyer, Swimmer):
        pass

    d = Duck()
    d.fly()
    d.swim()
    ```

### super() function
The `super()` function returns a proxy object that delegates method calls to a parent or sibling class of type. It is essential for:
1.  **Avoiding explicit parent names**: Makes the code more maintainable if the inheritance hierarchy changes.
2.  **Multiple Inheritance**: Ensuring that every class in a complex hierarchy (like a diamond) is called exactly once and in the correct order.

```python
class Base:
    def __init__(self):
        print("Base init")

class Child(Base):
    def __init__(self):
        super().__init__()
        print("Child init")
```

### Method Resolution Order (MRO)
MRO is the order in which Python looks for a method or attribute in a class hierarchy. Python uses the **C3 Linearization** algorithm.
*   It ensures a monotonic property (parents always come after children).
*   It handles the **Diamond Problem** (where two classes inherit from the same base, and a fourth class inherits from both).
*   You can inspect the MRO using the `mro()` method or `__mro__` attribute.

```python
class A: pass
class B(A): pass
class C(A): pass
class D(B, C): pass

print(D.mro())
# Output: [<class '__main__.D'>, <class '__main__.B'>, <class '__main__.C'>, <class '__main__.A'>, <class 'object'>]
```

### Mixin pattern
A **Mixin** is a special kind of multiple inheritance where a class provides a specific set of methods to other classes but is not intended to stand alone as a base class.
*   Mixins are used to add optional features to classes.
*   They help keep hierarchies flat and modular.

```python
import json

class JsonMixin:
    def to_json(self):
        return json.dumps(self.__dict__)

class User(JsonMixin):
    def __init__(self, name, age):
        self.name = name
        self.age = age

u = User("Alice", 30)
print(u.to_json()) # {"name": "Alice", "age": 30}
```

### Abstract Base Classes (ABC)
The `abc` module allows you to define **Abstract Base Classes**, which cannot be instantiated and require subclasses to implement specific methods. This acts as a formal "contract" or interface.

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self):
        pass

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius
        
    def area(self):
        return 3.14 * self.radius ** 2

# s = Shape() # Raises TypeError: Can't instantiate abstract class Shape
c = Circle(5)
print(c.area())
```

## Interview Questions
1.  **What is the "Diamond Problem" and how does Python solve it?**
    *   It occurs in multiple inheritance when a subclass inherits from two classes that both inherit from a common base class. Python solves this using the C3 Linearization (MRO) algorithm, ensuring the common base is only visited once after its descendants.
2.  **What is the difference between `super().__init__()` and `BaseClass.__init__(self)`?**
    *   `super()` follows the MRO, which is crucial in multiple inheritance to avoid multiple calls to the same base class. `BaseClass.__init__(self)` calls that specific class explicitly and can lead to bugs in complex hierarchies.
3.  **Why would you use a Mixin instead of standard inheritance?**
    *   Mixins promote "composition via inheritance." They allow you to add discrete functionalities (like logging or serialization) to many unrelated classes without creating a deep or confusing inheritance tree.
4.  **How do you check the Method Resolution Order of a class?**
    *   By calling `ClassName.mro()` or accessing `ClassName.__mro__`.
5.  **What happens if a subclass does not implement an `@abstractmethod`?**
    *   The subclass itself becomes abstract and cannot be instantiated. Attempting to create an instance will raise a `TypeError`.
