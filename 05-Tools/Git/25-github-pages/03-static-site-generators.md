# Static Site Generators

## Summary
SSGs build your raw content (Markdown) into HTML files. GitHub Pages has built-in support for **Jekyll**, but you can use any SSG with GitHub Actions.

## Detailed Explanation

### Jekyll
*   **Native**: GitHub runs Jekyll build automatically if it sees `_config.yml`.
*   **Pros**: Zero config.
*   **Cons**: Ruby-based, can be slow.

### Other SSGs (Hugo, Next.js, Gatsby)
*   **Workflow**:
    1.  Commit source code to `main`.
    2.  GitHub Action runs `hugo build`.
    3.  Action uploads the `public/` folder to Pages.

### Go-specific Context
**Hugo** is the world's fastest framework for building websites, written in Go. It is extremely popular in the Go community for blogs and documentation.

## Interview Questions
**Q: What is the `gh-pages` branch?**
**A:** Historically, this was the specific branch name required for Pages. Now you can deploy from any branch, but the name stuck as a convention for "deployment artifacts".
