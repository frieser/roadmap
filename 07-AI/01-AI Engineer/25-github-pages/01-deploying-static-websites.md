## Summary
GitHub Pages is a static site hosting service that takes HTML, CSS, and JavaScript files directly from a repository on GitHub and publishes a website. For AI Engineers, it is a perfect way to host project portfolios, documentation for AI libraries, or interactive model demos built with client-side frameworks.

## Detailed Explanation

### How it Works
GitHub Pages sites are hosted on the `github.io` domain (e.g., `username.github.io/repo-name`).

### Deployment Methods
1. **Deploy from a Branch**: You specify a branch (usually `main` or `gh-pages`) and a folder (root or `/docs`). GitHub automatically builds and deploys the site whenever you push to that branch.
2. **Deploy via GitHub Actions**: The modern, more flexible way. You define a workflow that builds your site (e.g., running `npm run build` for a React app) and then uses the `actions/deploy-pages` action to publish the results.

### Types of Sites
- **User/Organization Site**: `username.github.io`. Created from a repository named exactly `username.github.io`.
- **Project Site**: `username.github.io/project-name`. Created from any other repository.

### Use Case for AI Engineering: Portfolio & Demos
AI Engineers can use GitHub Pages to host:
- **Project Portfolios**: Showcasing links to repositories and papers.
- **Interactive Demos**: Using libraries like **TensorFlow.js** or **ONNX Runtime Web**, you can run AI models directly in the user's browser without needing a backend.

## Interview Questions

**Q: Can GitHub Pages host dynamic content like a PHP or Python backend?**
**A:** **No.** GitHub Pages is strictly for **static** content (HTML, CSS, JS). If you need a backend, you must host it elsewhere (e.g., Heroku, AWS, or Azure) and call its APIs from your frontend JavaScript.

**Q: What is the difference between a User site and a Project site?**
**A:** A User site is tied to the account, has the URL `username.github.io`, and must be built from a repo named `username.github.io`. A Project site is tied to a specific project, has the URL `username.github.io/repo-name`, and can be built from any repository.

**Q: Is HTTPS supported on GitHub Pages?**
**A:** Yes, GitHub provides automatic HTTPS for all GitHub Pages sites via Let's Encrypt, including those with custom domains.
