---
---

## Summary
Here Documents (Heredocs) allow you to pass multi-line blocks of text to a command's standard input. This is extremely useful for generating configuration files, feeding scripts to interpreters, or displaying long help messages without multiple `echo` statements.

## Detailed Explanation

### Syntax
```bash
command <<DELIMITER
Line 1
Line 2
DELIMITER
```
The `DELIMITER` (often `EOF`) marks the start and end. It must appear at the beginning of the line with no trailing spaces.

### Variations
1.  **Standard (`<<EOF`)**: Variables inside are expanded (`$VAR` is replaced).
2.  **Quoted (`<<'EOF'`)**: Disables variable expansion. Everything is treated as literal. Useful for writing scripts inside scripts.
3.  **Tab Suppression (`<<-EOF`)**: Ignores leading *tabs* (not spaces), allowing you to indent the heredoc for code readability.

## Go-Specific Context/Examples

Go supports raw string literals using backticks `` `...` ``, which serve a similar purpose for multi-line strings.

### Example: Multi-line String in Go
```go
query := `
SELECT *
FROM users
WHERE id = 1
`
```

### Example: Go generating a file
In Go, you'd usually use `os.WriteFile` or `fmt.Fprint` rather than a shell heredoc logic.

## Interview Questions

**Q: How do you write a Heredoc to a file instead of passing it to a command?**
**A:** Use `cat` with redirection.
```bash
cat <<EOF > output.txt
Hello World
EOF
```

**Q: Why use `<<-EOF` instead of `<<EOF`?**
**A:** It allows you to indent the heredoc content with **tabs** inside an `if` statement or function, keeping the script code clean. Standard `<<EOF` requires the content and the closing delimiter to be flush left (no indentation).

**Q: Can you pipe a Heredoc?**
**A:** Yes.
```bash
cat <<EOF | grep "search"
line 1
search me
line 3
EOF
```
