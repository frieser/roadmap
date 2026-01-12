# GitHub Discussions

## Summary
GitHub Discussions is a collaborative communication forum for the community around an open source project. It is separate from Issues, which are meant for actionable work (bugs/features). Discussions are for Q&A, ideas, and open-ended conversation.

## Detailed Explanation

### Structure
*   **Categories**: General, Q&A, Ideas, Show and Tell.
*   **Threaded**: Conversations are threaded (unlike the linear issue stream).
*   **Answers**: In Q&A, a comment can be marked as the "Answer" (similar to StackOverflow).

### When to use
*   **Issue**: "The login button is broken." (Actionable bug)
*   **Discussion**: "How do I configure the login provider?" (Question) or "We should rethink the login flow." (Open idea).

### Go-specific Context
Many Go libraries use Discussions to handle support requests, keeping their Issue tracker clean for actual bug reports. The "Answered" feature helps build a knowledge base for the library.

## Interview Questions
**Q: Can you convert an Issue to a Discussion?**
**A:** Yes, maintainers can move an issue to a discussion if it turns out to be a question rather than a bug.

**Q: Do Discussions support Markdown?**
**A:** Yes, fully.
