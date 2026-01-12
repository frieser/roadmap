## Summary
**Mentions** (@username) are used to notify specific individuals or teams about a discussion. In a collaborative AI environment, they are used to request reviews, signal experts, or hand over tasks.

## Detailed Explanation
### **Types of Mentions**
- **Individual**: `@johndoe` - Notifies a specific person.
- **Team**: `@my-org/ml-experts` - Notifies everyone in a specific GitHub Team. This is great for broad expert advice.
- **Keywords**: `@octocat` will notify you if you are mentioned.

### **Best Practices for AI Teams**
- **Don't Overuse**: Only mention people who truly need to be involved.
- **Mention for Expertise**: "Hey @jane-data-sci, could you check the normalization logic here?"
- **Mention for Approval**: Mentions in PRs are often used to ping "Code Owners" for final approval.

## Interview Questions
**Q: What is the difference between mentioning a user and mentioning a team?**
**A:** Mentioning a user notifies one person. Mentioning a team notifies every member of that team. Teams are better for "area-of-expertise" requests where any member of the group can help.

**Q: How can @mentions be used in commit messages?**
**A:** While possible, it's generally discouraged in commit messages as it creates notifications every time the commit is viewed or pushed. It's better to mention people in the PR description or comments.
