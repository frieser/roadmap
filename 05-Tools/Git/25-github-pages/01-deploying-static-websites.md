# Deploying Static Websites

## Summary
GitHub Pages is a static site hosting service that takes HTML, CSS, and JavaScript files directly from a repository and publishes a website.

## Detailed Explanation

### Sources
You can deploy from:
1.  **Branch**: `main` or `gh-pages`.
2.  **Folder**: `/docs` folder on `main` branch.
3.  **Action**: Custom GitHub Actions workflow (most flexible).

### Usage
Go to Settings > Pages > Select Source > Save.
Your site will be live at `https://username.github.io/repo`.

### Go-specific Context
If you write a Go library, you can use **Hugo** (which is written in Go) to build a documentation site and deploy it to Pages via Actions.

## Interview Questions
**Q: Can you host a backend API on GitHub Pages?**
**A:** No, it only serves static files (client-side only). You need a real server (Heroku, AWS, Render) for backend logic.
