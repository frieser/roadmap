# Commenting

## Summary
Commenting is the primary way to discuss code and issues. GitHub supports Markdown in comments, allowing for rich text, code snippets, and images.

## Detailed Explanation

### Types of Comments
1.  **Issue/PR Comment**: General discussion at the bottom of the thread.
2.  **Review Comment**: Attached to a specific line of code in a Pull Request diff.
3.  **Commit Comment**: Attached to a specific commit (less common).

### Suggested Changes
In a PR review, you can suggest a code change directly:
```markdown
```suggestion
func main() {
    fmt.Println("Hello, World!")
}
```
The author can then click "Commit suggestion" to apply it immediately without leaving the browser.

### Go-specific Context
When reviewing Go code, you can use syntax highlighting:
```go
// ```go
if err != nil {
    return err
}
// ```
```

## Interview Questions
**Q: How do you quote someone's reply?**
**A:** Select the text and press `r`, or copy it and prefix with `>`.

**Q: Can you edit a comment history?**
**A:** You can edit the comment text, and GitHub keeps a "Edited" dropdown history so others can see what was changed (transparency).
