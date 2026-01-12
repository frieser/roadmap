# commit-msg Hook

## Summary
The `commit-msg` hook is used to validate the commit message before the commit is finalized. It can be used to enforce a specific format or ensure that the message contains required information (like an issue number).

## Detailed Explanation
- **When it runs:** After the user enters the commit message, but before the commit is created.
- **Input:** It receives one argument: the name of a temporary file containing the commit message.

### AI Engineering Use Case
- **Experiment Linking:** Enforce that every commit includes an experiment ID or a link to a Jira/GitHub issue.
- **Conventional Commits:** Ensure messages follow the `feat:`, `fix:`, `refactor:` pattern, which helps in generating automated changelogs for model releases.

## Interview Questions
1.  **How do you abort a commit from within a `commit-msg` hook?**
    By exiting the script with a non-zero status code.
2.  **Can you modify the commit message within this hook?**
    Yes, the script can edit the temporary file passed as an argument.
