---
---

## Summary
Rendering strategies (CSR, SSR, SSG, ISR) define how web content is delivered to users. **SPA/CSR** focuses on client-side interactivity, **SSR** prioritizes SEO and initial load speed, **SSG** offers maximum performance via static distribution, and **ISR** provides a scalable middle ground. Modern architecture leverages **Hybrid Rendering** and **Edge Computing** to balance Core Web Vitals (LCP, FID, CLS) with infrastructure costs.

## Detailed Explanation

### 1. Client-Side Rendering (CSR) / SPA
In CSR, the server sends a minimal HTML file (often just a `<div>`) and a JavaScript bundle. The browser executes the JS to build the DOM and fetch data.
*   **Pros**: Fluid transitions (no full-page reloads), reduced server load after initial fetch.
*   **Cons**: Slow "Time to Interactive" (TTI), high bundle sizes, and SEO challenges for bots that don't execute JS efficiently.

### 2. Server-Side Rendering (SSR)
The server generates the full HTML for each request. The browser receives a ready-to-render page.
*   **Pros**: Excellent SEO, fast "First Contentful Paint" (FCP), reliable on slow devices.
*   **Cons**: High "Time to First Byte" (TTFB), increased server resources, and complexity in state management (hydration).

### 3. Static Site Generation (SSG)
Pages are generated during the build process and served as flat files from a CDN.
*   **Pros**: Blazing fast, ultra-secure (no runtime server), and zero scaling costs.
*   **Cons**: Data can be stale; long build times for sites with thousands of pages.

### 4. Incremental Static Regeneration (ISR)
A Next.js pattern that allows you to update static content *after* the build. It regenerates pages in the background as traffic comes in.
*   **Pros**: Best of both worlds (SSG speed + dynamic data), scales to millions of pages.
*   **Cons**: "Stale-while-revalidate" means the first user might see old content.

### 5. Edge Rendering & Middleware
Logic is executed at the CDN level (the "Edge").
*   **Edge SSR**: Rendering HTML closer to the user to minimize latency.
*   **Middleware**: Intercepting requests at the edge to handle authentication, A/B testing, or localized redirects before reaching the origin.

### SEO and Performance (Core Web Vitals)
*   **LCP (Largest Contentful Paint)**: SSR and SSG are superior as they provide content immediately.
*   **INP (Interaction to Next Paint)**: CSR can negatively impact INP if heavy hydration blocks the main thread.
*   **CLS (Cumulative Layout Shift)**: Hybrid approaches must ensure that server-rendered HTML matches the client-rendered state to avoid jumps.

### Go Application: High-Performance SSR
While JavaScript frameworks are common, Go is often used by architects for high-performance SSR backends due to its concurrency model and speed. Below is a Go implementation of a basic SSR server using `html/template`.

```go
package main

import (
	"html/template"
	"net/http"
	"time"
)

// PageData represents the data injected into the template
type PageData struct {
	Title   string
	Content string
	Time    string
}

func main() {
	// Define a simple HTML template (SSR)
	tmpl := `
<!DOCTYPE html>
<html>
<head>
    <title>{{.Title}}</title>
</head>
<body>
    <h1>{{.Title}}</h1>
    <p>{{.Content}}</p>
    <footer>Generated at: {{.Time}}</footer>
</body>
</html>`

	t := template.Must(template.New("webpage").Parse(tmpl))

	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		data := PageData{
			Title:   "Software Architecture: SSR in Go",
			Content: "This page was rendered on the server for optimal SEO and performance.",
			Time:    time.Now().Format(time.RFC1123),
		}

		// Execute template and write to response (SSR)
		err := t.Execute(w, data)
		if err != nil {
			http.Error(w, err.Error(), http.StatusInternalServerError)
		}
	})

	http.ListenAndServe(":8080", nil)
}
```

## Interview Questions

**Q: When should you choose SSR over SSG?**
**A:** Choose SSR when content changes frequently (e.g., a personalized dashboard or real-time feed) and SEO is critical. Choose SSG when content is mostly static (e.g., documentation, blogs) to maximize performance and minimize costs.

**Q: What is "Hydration" in the context of SSR/SSG?**
**A:** Hydration is the process where client-side JavaScript takes over the static HTML sent by the server, attaching event listeners and making the page interactive without a full re-render.

**Q: How does ISR solve the "long build time" problem of SSG?**
**A:** ISR allows you to build only a subset of critical pages at build time. Remaining pages are generated on-demand and then cached statically, preventing build times from scaling linearly with the number of pages.

**Q: Explain the impact of Edge Functions on TTFB.**
**A:** Edge Functions reduce Time to First Byte (TTFB) by moving the computing logic geographically closer to the user. Instead of a request traveling to a central origin server (e.g., US-East-1), it is processed at the nearest CDN node, significantly reducing network latency.
