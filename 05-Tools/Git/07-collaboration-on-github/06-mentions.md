# Mentions

## Summary
Mentions allow you to notify specific people or teams in a conversation. Using the `@` symbol triggers a notification for the mentioned entity.

## Detailed Explanation

### Types
*   **`@username`**: Notifies a specific person.
*   **`@org/team`**: Notifies a group of people (e.g., `@google/go-team`). Requires team configuration in an Organization.
*   **`@here` / `@channel`**: Not supported in GitHub (this is a Slack/Discord concept), though people sometimes use them out of habit.

### Notification settings
Users can configure how they receive these notifications (email, web, mobile).

### Go-specific Context
In the Go repo, you might mention `@golang/proposal-review` to get the attention of the proposal review committee, though usually, bots handle the routing based on labels.

## Interview Questions
**Q: What happens if you mention a user who doesn't have access to the private repo?**
**A:** They will not be notified, and the mention will not be a link (it will just be plain text), preventing information leakage.

**Q: Can you mention everyone in a repo?**
**A:** Only via `@all` if enabled (deprecated/discouraged in many contexts) or by mentioning a team that contains everyone.
