# Custom Domains

## Summary
You can serve your GitHub Pages site from a custom domain (e.g., `www.example.com`) instead of `github.io`.

## Detailed Explanation

### Steps
1.  **DNS**: Configure your domain provider (GoDaddy, Namecheap).
    *   `A` record points to GitHub IPs (`185.199.108.153`, etc).
    *   `CNAME` record points to `username.github.io`.
2.  **Repo Settings**: Go to Pages > Custom Domain > Enter `www.example.com`.
3.  **CNAME File**: GitHub automatically creates a file named `CNAME` in your repo root containing the domain name.

### HTTPS
GitHub automatically provisions a Let's Encrypt SSL certificate for your custom domain.

## Interview Questions
**Q: Why do you need a CNAME file in the repo?**
**A:** It acts as a source of truth so that if you redeploy the site (which might wipe the output folder), GitHub still knows which domain to serve.
