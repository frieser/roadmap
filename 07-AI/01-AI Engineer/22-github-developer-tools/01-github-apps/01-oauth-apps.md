## Summary
OAuth Apps in GitHub allow external applications to request access to a user's GitHub account and data. They use GitHub as an identity provider (Social Login) and can perform actions on behalf of the user. While they are easy to set up, they generally offer broader permissions than GitHub Apps and are primarily used when you need to act specifically as a user.

## Detailed Explanation

### How OAuth Apps Work
The OAuth 2.0 flow is standard for these apps:
1. **Redirect**: Your app redirects the user to GitHub to request access.
2. **Authorize**: The user authorizes your app.
3. **Callback**: GitHub redirects the user back to your app with a temporary `code`.
4. **Token Exchange**: Your app exchanges the `code` for an `access_token`.
5. **API Access**: Use the `access_token` to call the GitHub API on the user's behalf.

### Key Characteristics
- **Scopes**: OAuth Apps use "scopes" to define access levels (e.g., `repo` for full access to private repos, `user` for profile data). Scopes are often too broad (all-or-nothing for all repositories).
- **Identity**: Actions performed by the app are attributed to the user who authorized it.
- **Rate Limits**: Usually share the rate limit of the authorizing user.

### Comparison for AI Engineers
If you are building a tool that helps a developer analyze *their own* code across all their projects, an OAuth App might be suitable. However, for most automated AI services (like a PR reviewer bot), GitHub Apps are preferred.

### Python Example: Fetching User Info with OAuth
```python
import requests

def get_github_user(access_token):
    url = "https://api.github.com/user"
    headers = {"Authorization": f"token {access_token}"}
    response = requests.get(url, headers=headers)
    return response.json()

# In a real app, you'd get the access_token after the OAuth callback flow.
```

## Interview Questions

**Q: What is the main security disadvantage of OAuth Apps compared to GitHub Apps?**
**A:** OAuth Apps use broad scopes. For example, the `repo` scope grants access to ALL of a user's repositories. GitHub Apps allow for "fine-grained permissions," granting access only to specific repositories and specific actions (e.g., "Read access to code, Read/Write access to PRs").

**Q: Can an OAuth App be installed on an organization?**
**A:** No. Users authorize an OAuth App for their entire account. Organization owners can restrict which OAuth Apps are allowed to access organization data, but there is no "installation" process per repository like with GitHub Apps.

**Q: What is the purpose of the 'state' parameter in the OAuth flow?**
**A:** The `state` parameter is an unguessable random string used to protect against Cross-Site Request Forgery (CSRF) attacks. Your app sends it to GitHub, and GitHub sends it back in the callback. You must verify it matches the original string.
