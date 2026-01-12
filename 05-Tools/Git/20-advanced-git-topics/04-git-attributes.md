# Git Attributes

## Summary
The `.gitattributes` file allows you to define per-path settings. It controls things like line ending normalization, file diffing strategies, and export behavior.

## Detailed Explanation

### Common Settings
*   **Line Endings**: Ensure consistency across Windows (CRLF) and Linux (LF).
    ```
    * text=auto
    *.go text eol=lf
    ```
*   **Linguist**: Tell GitHub to ignore vendor files for language stats.
    ```
    vendor/** linguist-vendored
    ```
*   **Export**: Exclude files from `git archive` (zip downloads).
    ```
    /test export-ignore
    ```

### Go-specific Context
It is highly recommended to enforce `eol=lf` for `*.go` files to prevent `gofmt` issues on Windows machines.

## Interview Questions
**Q: How do you treat a file as binary so Git doesn't try to merge it?**
**A:** `*.dat binary` in `.gitattributes`.
