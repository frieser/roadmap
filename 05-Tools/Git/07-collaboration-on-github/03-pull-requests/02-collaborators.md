# Collaborators

## Summary
Collaborators are GitHub users who have been granted direct write access (push permission) to a repository. This differs from outside contributors who must use forks and Pull Requests.

## Detailed Explanation

### Access Levels
*   **Read**: View only (Private repos).
*   **Triage**: Manage issues/PRs but no write access.
*   **Write**: Push to branches, merge PRs.
*   **Maintain**: Manage repo settings but not destructive actions.
*   **Admin**: Full control (delete repo).

### Inviting
Settings -> Collaborators -> Add people. They will receive an email invitation.

### Go-specific Context
In the Go project, "Approvers" and "Maintainers" are specific roles. Only a small set of people have commit rights to the `golang/go` repo to ensure language stability and security.

## Interview Questions
**Q: How do you add a collaborator to a personal repo?**
**A:** Go to Settings > Collaborators and search for their username.

**Q: Does a collaborator need to fork the repo?**
**A:** No, they can clone the repo directly and push to it (though using feature branches and PRs is still best practice).
