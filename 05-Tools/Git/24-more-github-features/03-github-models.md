# GitHub Models

## Summary
GitHub Models (and LFS) supports the storage and versioning of Machine Learning models. This includes tracking large binary blobs and providing diffs for notebooks.

## Detailed Explanation

### Features
*   **Notebook rendering**: GitHub natively renders `.ipynb` (Jupyter Notebooks).
*   **LFS Integration**: Necessary for storing model weights (`.h5`, `.pt`, `.onnx`) which can be gigabytes in size.

## Interview Questions
**Q: How do you diff a Jupyter Notebook?**
**A:** GitHub has a specialized viewer that hides the JSON structure and shows the cell changes visually.
