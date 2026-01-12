## Summary
`git bisect` is a powerful debugging tool that uses a binary search algorithm to find the specific commit that introduced a bug or regression. For AI Engineers, this is invaluable for identifying exactly when model performance dropped or when a data processing error was introduced.

## Detailed Explanation
### **The Bisect Workflow**
1. **Start**: `git bisect start`
2. **Mark Bad**: `git bisect bad` (Mark the current commit as having the bug).
3. **Mark Good**: `git bisect good <commit-hash>` (Mark a past commit where the bug did not exist).
4. **Testing**: Git will automatically check out a commit in the middle. You run your tests (or model evaluation) and tell Git if it's `good` or `bad`.
5. **Finish**: Once found, Git displays the offending commit. Run `git bisect reset` to return to your original branch.

### **Automation with `git bisect run`**
You can automate the process if you have a script that returns 0 for success (good) and non-zero for failure (bad):
```bash
git bisect run python evaluate_model.py --threshold 0.85
```

### **AI Engineering Use Case**
Finding a regression in training loss. If you notice today's model is significantly worse than last week's, `git bisect` can pinpoint the exact change in code or config that caused the degradation.

## Interview Questions
- **Q: What algorithm does `git bisect` use to find a bug?**
- **A:** A binary search algorithm.

- **Q: How do you automate a bisect process?**
- **A:** By using `git bisect run <script_name>`. The script should return 0 for a good commit and 1-127 (except 125) for a bad one.

- **Q: What should you do if a commit chosen by bisect cannot be tested (e.g., it doesn't build)?**
- **A:** Use `git bisect skip` to tell Git to find a nearby commit to test instead.
