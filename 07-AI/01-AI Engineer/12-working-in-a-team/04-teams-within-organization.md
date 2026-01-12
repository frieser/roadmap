## Summary
**Teams** within an organization allow you to group members and manage their access to repositories as a unit. For complex AI organizations, nested teams help mirror the actual company structure.

## Detailed Explanation
### **Team Structure**
- **Parent Team**: e.g., "AI Department".
- **Child Teams**: e.g., "Deep Learning", "Data Engineering", "MLOps".
- **Automatic Membership**: Members of a child team are automatically included in the parent team's discussions and permissions.

### **AI Team Roles**
- **@org/data-scientists**: Write access to research repos.
- **@org/mlops-engineers**: Maintainer access to deployment and infrastructure repos.
- **@org/reviewers**: A team specifically for high-level architectural oversight.

### **Benefits of Mentions**
Mentioning `@org/nlp-team` in an issue notifies everyone on that team, ensuring that the right group of experts is alerted to a problem without needing to remember every individual's name.

## Interview Questions
**Q: How do nested teams simplify permission management in a large organization?**
**A:** Nested teams allow permissions to flow downward. If you give the "AI Department" parent team read access to a repo, all child teams (like "Vision" and "NLP") automatically get that same access, reducing the need for manual setup.

**Q: What is the "Team Maintainer" role?**
**A:** A Team Maintainer is a member of a team who can manage that team's membership and settings (like adding/removing people or changing the team's description) without needing full Organization Admin permissions.
