# Markdown

## Summary
Markdown is a lightweight markup language used for formatting text in GitHub READMEs, Issues, and Pull Requests. GitHub Flavored Markdown (GFM) adds extra features like task lists, tables, and syntax highlighting.

## Detailed Explanation

### Basic Syntax
*   `# Heading 1`, `## Heading 2`
*   `**Bold**`, `*Italic*`
*   `[Link](url)`
*   `![Image](url)`
*   `- List item`

### GFM Features
*   **Task Lists**:
    ```markdown
    - [x] Done
    - [ ] Todo
    ```
*   **Tables**:
    ```markdown
    | Header | Header |
    | --- | --- |
    | Cell | Cell |
    ```
*   **Syntax Highlighting**:
    ```go
    // This is highlighted as Go code
    func main() {}
    ```

### Go-specific Context
Go documentation comments (godoc) use a simplified text format, but newer tools like `pkgsite` render them with markdown-like features. GitHub renders `.md` files in Go repositories automatically.

## Interview Questions
**Q: How do you create a code block in Markdown?**
**A:** Use triple backticks (\`\`\`). You can specify the language name for highlighting.

**Q: How do you mention a user or issue?**
**A:** `@username` and `#issue_number`.
