# Client-Side vs. Server-Side Hooks

## Summary
Git hooks are scripts that Git executes before or after events such as commit, push, and receive. They are divided into client-side (local) and server-side (remote) hooks, each serving different purposes in the development and deployment lifecycle.

## Detailed Explanation

### Client-Side Hooks
These reside on your local machine in the `.git/hooks` directory.
- **Triggered by:** `git commit`, `git merge`, `git push`, `git checkout`, etc.
- **Purpose:** Local enforcement of coding standards, running unit tests, or formatting code.
- **Sharing:** Since `.git/hooks` is not committed, sharing hooks requires external tools like the `pre-commit` framework.

### Server-Side Hooks
These reside on the Git server (e.g., GitHub, GitLab, Bitbucket).
- **Triggered by:** `git receive` (when someone pushes to the server).
- **Purpose:** Final validation before changes are accepted into the shared repository.
- **Types:** `pre-receive` (runs before any refs are updated), `update` (runs for each branch being updated), `post-receive` (runs after updates, used for notifications or CI/CD triggers).

### AI Engineering Comparison
| Feature | Client-Side Hook | Server-Side Hook |
| :--- | :--- | :--- |
| **Typical AI Use** | Formatting code (`black`), checking config syntax. | Rejecting pushes containing large datasets or secrets. |
| **Feedback Loop** | Immediate (runs on developer's machine). | Delayed (runs after `git push`). |
| **Bypassability** | Easy (`--no-verify`). | Hard (cannot be bypassed by the client). |

## Interview Questions
1.  **Why are client-side hooks not committed to the repository by default?**
    Security and portability. Hooks are executable scripts; automatically running a script from a cloned repo could be dangerous. Different OSs might also need different script versions.
2.  **How can a team ensure everyone uses the same hooks?**
    Using a tool like `pre-commit` (Python-based) which manages hook installation via a committed `.pre-commit-config.yaml` file.
3.  **When would you use a `pre-receive` hook instead of a `pre-commit` hook?**
    When you want to enforce a rule that NO ONE can bypass (e.g., "no large binary files over 50MB in this repo").
