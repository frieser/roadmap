## Summary
The `.gitattributes` file allows you to define path-specific settings for Git. It is used to handle line endings, specify diff strategies for non-text files, and configure Git Large File Storage (LFS).

## Detailed Explanation
### **Common Uses**
- **Line Endings**: Force LF or CRLF for specific files.
  `*.sh text eol=lf`
- **LFS Integration**: Tell Git which files should be handled by LFS.
  `*.onnx filter=lfs diff=lfs merge=lfs -text`
- **Custom Diffs**: For AI Engineers, you might want to treat Jupyter Notebooks (.ipynb) differently during a diff.

### **Configuring .gitattributes**
You create a `.gitattributes` file in the root of your project. Each line follows the pattern `pattern attribute1 attribute2...`.

### **AI-Specific: Jupyter Notebook Diffs**
Notebooks are JSON files, and standard line-by-line diffs are often unreadable. You can use attributes to hook into tools like `nbdiff`:
```
*.ipynb diff=jupyternotebook
```
(Requires corresponding configuration in `.gitconfig`).

## Interview Questions
- **Q: What file is used to configure Git Large File Storage (LFS) for specific file types?**
- **A:** The `.gitattributes` file.

- **Q: How do you force Git to treat a file as binary, even if it looks like text?**
- **A:** By adding `filename -text` to the `.gitattributes` file.

- **Q: What is the purpose of the `eol` attribute?**
- **A:** It specifies the end-of-line character (LF or CRLF) to be used for the file in the working directory.
