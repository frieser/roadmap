# Teams

## Summary
Teams are groups of organization members that reflect your company or project structure. You can mention teams (`@org/team`) and assign permissions to teams instead of individuals.

## Detailed Explanation

### Structure
*   **Parent/Child Teams**: You can nest teams (e.g., `Engineering` > `Backend` > `Platform`).
*   **Code Owners**: You can assign a Team as the code owner of a directory, so they are automatically requested for review.

### Go-specific Context
In the Kubernetes project, "SIGs" (Special Interest Groups) are mapped to GitHub Teams. For example, `@kubernetes/sig-cli-maintainers` will be pinged for PRs touching `kubectl` code.

## Interview Questions
**Q: What is the benefit of assigning permissions to a Team?**
**A:** It simplifies onboarding. When a new hire joins, you just add them to the "Developers" team, and they instantly get access to all 50 repositories that the team has access to.
