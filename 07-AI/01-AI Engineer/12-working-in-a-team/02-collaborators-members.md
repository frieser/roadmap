## Summary
Managing **Collaborators and Members** is about controlling who has access to your project's code and data. For AI teams, this involves distinguishing between internal researchers, external partners, and automated service accounts.

## Detailed Explanation
### **Individual vs Organization Access**
- **Collaborators**: Individuals invited to a specific repository. Best for small projects or external contributors.
- **Organization Members**: People with access to the entire GitHub Organization. Best for internal teams.

### **Roles in AI Teams**
- **Read-only**: Stakeholders or external auditors who need to see the code/results but not change them.
- **Triage**: Data managers who organize labels and issues but don't touch the model code.
- **Write**: The majority of AI Engineers and Researchers.
- **Maintain/Admin**: Team leads and MLOps engineers who manage secrets (like Hugging Face tokens) and repository settings.

## Interview Questions
**Q: What is the difference between a Repository Collaborator and an Organization Member?**
**A:** A **Collaborator** is invited to one specific repository. An **Organization Member** belongs to the whole organization and can be given access to multiple repositories and teams through organization-level settings.

**Q: Why should you avoid giving "Admin" access to everyone on an AI team?**
**A:** "Admin" access allows for destructive actions like deleting the repository, changing sensitive secrets, or removing branch protections. Following the **Principle of Least Privilege** ensures that team members only have the access they need to perform their jobs, reducing the risk of accidental errors.
