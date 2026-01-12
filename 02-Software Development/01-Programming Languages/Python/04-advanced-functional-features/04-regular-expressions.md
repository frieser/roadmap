# Regular Expressions

## Summary
Python's `re` module provides support for regular expressions (Perl-style). Regex is a sequence of characters that forms a search pattern, used for string searching and manipulation.

## Detailed Explanation

### Basic Usage
```python
import re

text = "The rain in Spain"
# Search: Returns Match object or None
x = re.search("^The.*Spain$", text) 
```

### Common Functions
*   `re.search(pattern, string)`: Finds the first location where pattern matches.
*   `re.match(pattern, string)`: Checks for a match only at the **beginning** of the string.
*   `re.findall(pattern, string)`: Returns a list of all non-overlapping matches.
*   `re.sub(pattern, repl, string)`: Replaces matches with a string.
*   `re.compile(pattern)`: Compiles a regex for better performance if reused.

### Groups
Parentheses `()` define capture groups.
```python
email = "user@example.com"
match = re.search(r"(\w+)@(\w+\.\w+)", email)
if match:
    print(match.group(0)) # user@example.com (Full match)
    print(match.group(1)) # user (Group 1)
    print(match.group(2)) # example.com (Group 2)
```

### Modern Features (3.11+)
*   **Atomic Grouping**: `(?>...)` (supported in 3.11+).
*   **`re.NOFLAG`**: Explicitly pass 0 as a flag.

### Flags
*   `re.IGNORECASE` (`re.I`): Case-insensitive match.
*   `re.MULTILINE` (`re.M`): `^` and `$` match start/end of each line, not just string.
*   `re.VERBOSE` (`re.X`): Allows comments and whitespace in regex pattern for readability.

## Interview Questions

**Q: What is the difference between `re.match()` and `re.search()`?**
**A:** `re.match()` checks for a match only at the **start** of the string. `re.search()` scans through the entire string to find the first location where the pattern produces a match.

**Q: Why use raw strings (`r"..."`) for regex patterns?**
**A:** Because regex uses backslashes `\` for special characters (like `\d` for digit). Python strings also use backslashes for escapes. Using raw strings tells Python not to interpret backslashes, passing them directly to the regex engine. Otherwise, you'd need `\\d`.

**Q: How do you perform a case-insensitive search?**
**A:** Pass the `re.IGNORECASE` (or `re.I`) flag to the regex function. Example: `re.search(r"pattern", text, re.IGNORECASE)`.
