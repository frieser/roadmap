---
---

## Summary
In Bash, proper quoting is essential to handle whitespace, special characters, and variable expansion correctly. The three main quoting mechanisms are Single Quotes (`'`), Double Quotes (`"`), and ANSI-C Quoting (`$'`).

## Detailed Explanation

### 1. Single Quotes (`'...'`)
*   **Strongest Quoting**: Preserves the literal value of **every** character within the quotes.
*   **No Expansion**: Variables (`$VAR`) are not expanded. Backslashes (`\`) are literals.
*   **Limitation**: You cannot include a single quote inside single quotes (even with escaping).

### 2. Double Quotes (`"..."`)
*   **Weak Quoting**: Preserves the literal value of most characters, BUT allows:
    *   Variable expansion: `$VAR`.
    *   Command substitution: `$(cmd)`.
    *   Backslash escaping: `\"`, `\$`, `\\`.
*   **Use Case**: When you need to interpolate variables but keep the result as a single argument (prevent word splitting).

### 3. ANSI-C Quoting (`$'...'`)
*   Allows C-style backslash escapes.
*   `$'\n'` (Newline), `$'\t'` (Tab), `$'\x41'` (Hex 'A').
*   Useful for inserting non-printable characters.

## Go-Specific Context/Examples

Go handles literals differently:
*   **Double Quotes (`"..."`)**: Interpreted string literals (supports `\n`, `\t`).
*   **Backticks (`` `...` ``)**: Raw string literals. Can span multiple lines. No escape sequences processed.

### Analogy
*   Bash `'...'` ≈ Go `` `...` `` (Raw).
*   Bash `$'...'` ≈ Go `"..."` (Interpreted).

## Interview Questions

**Q: How do you include a single quote inside a single-quoted string?**
**A:** You technically can't. You have to close the quote, insert an escaped quote, and re-open.
`'It'\''s me'` -> `It's me`. Or use double quotes: `"It's me"`.

**Q: Why should you always quote variables like `"$VAR"`?**
**A:** To prevent **Word Splitting** and **Globbing**. If `$VAR` contains spaces (e.g., `file name.txt`) and you run `rm $VAR`, Bash sees `rm file` and `rm name.txt`. Quoting ensures it's treated as one argument.

**Q: What is the difference between `echo '$HOME'` and `echo "$HOME"`?**
**A:**
*   `'$HOME'` prints the literal string `$HOME`.
*   `"$HOME"` prints the path `/home/user`.
