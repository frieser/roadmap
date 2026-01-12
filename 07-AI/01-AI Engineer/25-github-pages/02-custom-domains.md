## Summary
GitHub Pages allows you to point a custom domain (like `www.example.com`) to your hosted site instead of using the default `github.io` subdomain. This is essential for professional portfolios and branded open-source projects.

## Detailed Explanation

### Configuration Steps
1. **On GitHub**: Go to Repository Settings > Pages. Enter your custom domain in the "Custom domain" field. This creates a `CNAME` file in the root of your repository.
2. **On your DNS Provider (e.g., Namecheap, Cloudflare)**:
   - For **Apex domains** (example.com): Create `A` records pointing to GitHub's IP addresses.
   - For **Subdomains** (www.example.com): Create a `CNAME` record pointing to `username.github.io`.

### Important Considerations
- **HTTPS/SSL**: GitHub automatically generates an SSL certificate for your custom domain using Let's Encrypt. It may take up to 24 hours to provision.
- **Enforce HTTPS**: You should always check the "Enforce HTTPS" box in settings to ensure all traffic is encrypted.
- **WWW vs Apex**: It is generally recommended to use a `www` subdomain and have the apex domain redirect to it, as this is more robust for DNS management.

## Interview Questions

**Q: What is the purpose of the `CNAME` file in a GitHub Pages repository?**
**A:** The `CNAME` file tells GitHub which custom domain is associated with that specific repository. When GitHub's servers receive a request for that domain, they use this file to route the request to the correct repo.

**Q: Can you use a custom domain with a private repository's GitHub Pages site?**
**A:** Yes, but GitHub Pages for private repositories is a GitHub Pro/Team/Enterprise feature. Once enabled, the custom domain works the same way as it does for public repositories.

**Q: Why might your custom domain show a "404 Not Found" error even after setting it up on GitHub?**
**A:** This usually happens if the DNS records haven't propagated yet, or if the DNS records are not correctly pointing to GitHub's servers. You can use tools like `dig` or `nslookup` to verify your DNS configuration.
