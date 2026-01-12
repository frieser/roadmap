#Python
---
---

## Summary
**YAPF (Yet Another Python Formatter)** is an open-source code formatter for Python, originally developed by Google. Unlike traditional formatters that only fix linting violations (like `autopep8`), YAPF takes a different approach based on the `clang-format` algorithm. It reformats the entire code by building a solution space of possible formatting decisions and selecting the one with the lowest "penalty" or cost, ensuring the code adheres strictly to a configured style guide while maintaining a "human-written" feel.

## Detailed Explanation

### Philosophy: "Code that looks like a programmer wrote it"
The primary goal of YAPF is not just to make code conform to a style guide, but to produce **"code that looks like a programmer would write"** if they were following that guide perfectly. 

Instead of simple search-and-replace rules, YAPF uses a **cost-based algorithm**:
1. It analyzes the "Logical Line" (a statement that could fit on one line).
2. It explores various ways to split the line (before/after operators, brackets, etc.).
3. Each decision (to split or not to split) incurs a "penalty" based on configured rules (knobs).
4. It finds the optimal path through this decision tree that minimizes the total penalty.

### Configuration via `.style.yapf`
YAPF is highly configurable, allowing developers to tune its behavior via hundreds of "knobs."

#### Configuration Files
YAPF searches for configuration in the following order:
1. Command line argument: `--style`
2. `.style.yapf` (under `[style]` section)
3. `setup.cfg` (under `[yapf]` section)
4. `pyproject.toml` (under `[tool.yapf]` section)
5. `~/.config/yapf/style` (Global user config)

#### Example `.style.yapf`
```ini
[style]
based_on_style = google
indent_width = 4
column_limit = 80
spaces_before_comment = 4
split_before_logical_operator = true
```

#### Predefined Styles
You can base your configuration on one of the standard styles:
- `pep8` (Default)
- `google`
- `yapf` (Used for Google open-source projects)
- `facebook`

### Usage

#### Command Line Interface (CLI)
```bash
# Reformat a file in-place
yapf -i example.py

# Print a diff of the changes without applying them
yapf -d example.py

# Reformat an entire directory recursively
yapf -ri my_project/

# Check if a file is formatted (useful for CI/CD)
yapf -d example.py || exit 1
```

#### Disabling Formatting
You can instruct YAPF to ignore specific sections of code using comments:

```python
# yapf: disable
def long_function_name(
    var_one, var_two, var_three,
    var_four):
    print(var_one)
# yapf: enable

# Or for a single literal
BAZ = {
    (1, 2, 3),
    (4, 5, 6),
}  # yapf: disable
```

#### Python API
YAPF can be used as a library within other Python tools:
```python
from yapf.yapflib.yapf_api import FormatCode

code = "def f(a,b): return a+b"
formatted_code, changed = FormatCode(code, style_config='pep8')
print(formatted_code)
# Output:
# def f(a, b):
#     return a + b
```

## Interview Questions

### 1. How does YAPF differ from Black?
While both are Python formatters, **YAPF is highly configurable** through "knobs," whereas **Black is "uncompromising"** and opinionated with almost no configuration. YAPF aims to make code look "human-written" using a cost-based optimization algorithm, while Black focuses on determinism (producing the exact same output regardless of the previous state).

### 2. How do you prevent YAPF from reformatting a specific block of code?
You use the `# yapf: disable` and `# yapf: enable` comments to wrap the code block you want YAPF to ignore.

### 3. What is the significance of the `based_on_style` setting?
It allows for "style inheritance." You can start with a standard style (like `pep8` or `google`) and only specify the individual settings (knobs) you want to override, ensuring consistency without defining every rule from scratch.

### 4. Is YAPF safe to run on large codebases?
Yes, because YAPF is designed to **never alter the semantics** of the code. It works on the token stream and only changes whitespace and line breaks. It will not add parentheses or change the order of imports (unlike `isort`).

### 5. How can you use YAPF to ensure code quality in a CI/CD pipeline?
By running `yapf --diff --recursive .`. If the command returns a non-zero exit code, it means some files are not correctly formatted, and the build should fail.
