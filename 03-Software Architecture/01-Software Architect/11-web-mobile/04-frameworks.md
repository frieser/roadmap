---
---

## Summary
Choosing the right Web and Mobile frameworks is one of the most visible decisions a Software Architect makes. The choice dictates the hiring pool, development velocity, and long-term maintenance burden. The debate often centers on "Single Page Apps (SPA) vs Multi-Page Apps (MPA)" and "Native Mobile vs Cross-Platform".

## Detailed Explanation

### Web Frameworks
1.  **React (Meta)**: Component-based, huge ecosystem, Virtual DOM. De facto standard.
2.  **Vue.js**: Easier learning curve, very popular in Asia and PHP communities.
3.  **Angular (Google)**: Full-featured, opinionated, TypeScript-native. Enterprise favorite.
4.  **HTMX / Hotwire**: The "Anti-SPA" movement. Sending HTML over the wire instead of JSON. Great for backend-heavy teams (Go/Django/Rails) who want interactivity without complex JS build steps.

### Mobile Frameworks
1.  **Native (Swift/Kotlin)**: Best performance, full API access. Expensive (need 2 teams).
2.  **React Native**: Write in JS/React, render to native views. Good compromise.
3.  **Flutter (Google)**: Write in Dart, render to Skia canvas (pixel perfect everywhere). Fast, but different ecosystem.

### Architect's Role
*   **Evaluate**: Does the team know JS? If yes, React. If they are Go devs, maybe HTMX.
*   **Standardize**: Don't let Team A use Vue and Team B use Angular unless necessary.
*   **Performance**: Consider Bundle Size, SEO (SSR vs CSR), and Battery life.

## Go-Specific Context/Examples

Go is primarily a backend language, but it pairs interestingly with frontend choices.

### Go + HTMX (The "Go Stack")
Instead of building a JSON API + React App, you serve HTML templates (`html/template` or `templ`) and use HTMX to swap parts of the page dynamically.
*   **Pros**: No `node_modules`, single binary deploy, very fast.
*   **Cons**: Less rich interactivity than full React for complex apps (like Google Maps).

### Go + React
The standard model. Go is a JSON REST/gRPC API. React is a static asset on CDN.

## Interview Questions

**Q: When would you choose HTMX over React?**
**A:** When the application is "Content-heavy" or "Dashboard-like" rather than highly interactive/stateful (like a game or graphic editor). Also, when the team consists of backend engineers who want to avoid the complexity of modern frontend build chains (Webpack/Vite).

**Q: What is the trade-off of Cross-Platform mobile (Flutter/RN)?**
**A:** Trade-off is **Access to Native Features** and **Performance**. While close to native, heavy animations or bleeding-edge OS features (ARKit updates) might lag behind or require writing native "Bridge" code, adding complexity.

**Q: Why is Server-Side Rendering (SSR) important?**
**A:** SEO and First Contentful Paint (FCP). Search engines index HTML better than empty pages waiting for JS to load. Users see content faster on slow networks.
