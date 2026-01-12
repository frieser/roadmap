## Summary
**git log** is the primary tool for inspecting project history. It offers numerous options to filter and format the output, which is essential for AI Engineers to track model evolution and identify the source of regressions.

## Detailed Explanation
### **Essential Options**
- **-p / --patch**: Shows the actual code changes in each commit.
- **--stat**: Shows a summary of which files were changed and how many lines were added/removed.
- **--oneline**: Condenses each commit to a single line (hash + message).
- **--graph**: Visualizes the branching and merging structure.
- **--author="Name"**: Filters commits by a specific person.
- **--grep="keyword"**: Searches for keywords in commit messages (e.g., "learning rate").
- **-S "string"**: (The "Pickaxe") Finds commits that added or removed a specific string in the code.

### **AI Use Case**
`git log -p -S "0.001" -- path/to/config.yaml`
This command would find every commit that changed the value "0.001" (likely a learning rate) in the configuration file, helping you trace hyperparameter changes.

## Interview Questions
**Q: How do you see the history of a specific file on GitHub?**
**A:** Using `git log -- <filename>`. Adding `-p` will show the exact changes made to that file in each commit.

**Q: What is the "pickaxe" option (`-S`) in git log and why is it useful?**
**A:** The `-S` option searches for commits that changed the number of occurrences of a specific string in the codebase. It is extremely useful for finding when a specific variable, function name, or hyperparameter value was introduced or removed.
