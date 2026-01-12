#Python
---
---

## Summary
Pydantic is the most widely used data validation library for Python, leveraging Python type hints to enforce data schemas. Unlike standard type checkers that only perform static analysis, Pydantic performs **runtime validation** and **data coercion**. It ensures that data conforms to specified types while providing a powerful API for serialization, complex validation logic, and configuration management.

## Detailed Explanation

### 1. BaseModel and Type Hints
The core of Pydantic is the `BaseModel`. By defining a class that inherits from `BaseModel`, you specify the expected structure and types of your data using Python's standard type hints.

Pydantic's most powerful feature is **data coercion**. If a field is typed as an `int` but receive a string `"123"`, Pydantic will automatically convert it to an integer.

```python
from pydantic import BaseModel

class User(BaseModel):
    id: int
    name: str
    is_active: bool = True

# Data is automatically coerced
user = User(id="123", name="Alice")
print(user.id)        # 123 (int, not "123")
print(user.is_active) # True (default value)
```

### 2. The Field Function
The `Field` function allows you to add metadata, validation constraints, and serialization instructions to model fields.

*   **Constraints**: `gt`, `lt`, `min_length`, `max_length`, `pattern`.
*   **Metadata**: `title`, `description`, `examples`.
*   **Serialization**: `alias`, `exclude`, `include`.

```python
from pydantic import BaseModel, Field

class Product(BaseModel):
    name: str = Field(..., min_length=2, max_length=50)
    price: float = Field(..., gt=0, description="The price must be positive")
    sku: str = Field(alias="stock_keeping_unit")

product = Product(name="Coffee", price=15.5, stock_keeping_unit="COF-001")
print(product.model_dump(by_alias=True))
# {'name': 'Coffee', 'price': 15.5, 'stock_keeping_unit': 'COF-001'}
```

### 3. Validators
Pydantic v2 provides two main decorators for custom validation:

*   **@field_validator**: Validates a specific field. Can run `before` (on raw input) or `after` (on coerced value) validation.
*   **@model_validator**: Validates the entire model. Useful for cross-field validation.

```python
from typing_extensions import Self
from pydantic import BaseModel, field_validator, model_validator

class Registration(BaseModel):
    username: str
    password: str
    confirm_password: str

    @field_validator('username')
    @classmethod
    def username_must_be_lowercase(cls, v: str) -> str:
        if v.lower() != v:
            raise ValueError('Username must be lowercase')
        return v

    @model_validator(mode='after')
    def check_passwords_match(self) -> Self:
        if self.password != self.confirm_password:
            raise ValueError('Passwords do not match')
        return self
```

### 4. Serialization
Pydantic models can be easily converted to dictionaries or JSON strings.

*   `model_dump()`: Serializes the model to a Python `dict`.
*   `model_dump_json()`: Serializes the model to a JSON string.

Options like `exclude`, `include`, and `exclude_unset` provide granular control over the output.

```python
from pydantic import BaseModel, Field

class Profile(BaseModel):
    username: str
    bio: str | None = None
    secret_token: str = Field(exclude=True)

profile = Profile(username="dev_ninja", secret_token="top-secret")
print(profile.model_dump()) 
# {'username': 'dev_ninja', 'bio': None} (secret_token excluded)
```

### 5. Settings Management
Using the `pydantic-settings` library, you can manage application configuration by loading environment variables into Pydantic models.

```python
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
    api_key: str
    db_url: str = "postgresql://localhost/db"
    
    model_config = SettingsConfigDict(env_prefix='APP_', env_file='.env')

# Pydantic will look for APP_API_KEY and APP_DB_URL in environment variables
# settings = Settings()
```

## Interview Questions

**Q: What is the difference between Pydantic and Python's built-in `dataclasses`?**
**A:** `dataclasses` are primarily for storing data with minimal boilerplate and do not provide runtime validation by default. Pydantic is designed for data validation, providing runtime type checking, data coercion, and advanced serialization features.

**Q: How does Pydantic v2 improve performance compared to v1?**
**A:** Pydantic v2's core logic (the validation engine) is rewritten in **Rust** (pydantic-core). This results in significantly faster validation and serialization, often 5-50x faster than v1.

**Q: Explain the difference between `model_validate()` and `model_validate_json()`.**
**A:** `model_validate()` takes a Python dictionary or object and validates it against the schema. `model_validate_json()` takes a raw JSON string and parses/validates it in a single step, which is more efficient than calling `json.loads()` and then `model_validate()`.

**Q: What is "Data Coercion" in Pydantic?**
**A:** It is the process of converting input data to the required type if possible. For example, if a field is defined as `bool`, Pydantic will coerce values like `"true"`, `"1"`, `1`, or `True` into the boolean `True`.

**Q: When should you use a `before` validator vs an `after` validator?**
**A:** Use `before` validators when you need to transform or validate the raw input data (like a string that needs parsing) before Pydantic attempts to coerce it. Use `after` validators when you want to validate the data after it has been coerced and verified against the field's type and constraints.
