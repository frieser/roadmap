# Citation Files

## Summary
A `CITATION.cff` file allows researchers and users to correctly cite your software in academic papers. GitHub detects this file and provides a "Cite this repository" button.

## Detailed Explanation

### Format
It uses YAML format (Citation File Format).
```yaml
cff-version: 1.2.0
message: "If you use this software, please cite it as below."
authors:
- family-names: "Lisa"
  given-names: "Mona"
title: "My Research Software"
version: 2.0.4
date-released: 2021-08-11
```

### Use Case
Crucial for scientific software or open-source libraries that want academic credit.

### Go-specific Context
If you are writing a Go library for scientific computing (e.g., Gonum), adding a `CITATION.cff` is highly recommended to encourage academic attribution.

## Interview Questions
**Q: Where do you find the citation info on GitHub?**
**A:** In the "About" sidebar, there will be a "Cite this repository" link if the file exists.
