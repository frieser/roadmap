---
---

# Vercel and Netlify Functions

Vercel and Netlify revolutionized frontend deployment (JAMstack) but have evolved into powerful full-stack serverless platforms. They abstract away the complexity of AWS Lambda, offering a "zero-config" experience where API routes are deployed simply by adding files to a specific directory (`/api` or `/netlify/functions`).

## Summary

*   **Serverless Functions**: Standard Node.js/Go/Python functions (usually running on AWS Lambda under the hood). Good for database calls, authentication, etc.
*   **Edge Functions**: Code that runs on the CDN edge (Cloudflare Workers or Vercel Edge). Extremely low latency, but limited runtime capabilities (often subset of JS/WASM, no standard TCP sockets).

## Detailed Explanation

### 1. Developer Experience (DX)
The key differentiator is **Git Integration**. You push to GitHub, and Vercel/Netlify builds the frontend *and* deploys the backend functions automatically. No Terraform or CloudFormation required.

### 2. Edge vs. Serverless
*   **Serverless**: Region-bound (e.g., us-east-1). Access to full Node.js/Go runtime. Higher latency.
*   **Edge**: Global. Runs in the datacenter closest to the user. Near-zero cold start. Restricted environment (V8 isolate). Ideal for AB testing, personalized headers, or redirects.

---

## Go Implementation Example

Both platforms support Go natively. You simply place a `.go` file in the functions directory, and the build system compiles and deploys it as a Lambda.

### Vercel (`api/user.go`)
Vercel uses the standard `net/http` signature.

```go
package handler

import (
	"fmt"
	"net/http"
)

// Handler is the entry point exposed to Vercel
func Handler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello from Vercel Go!")
}
```

### Netlify (`netlify/functions/hello/main.go`)
Netlify traditionally uses the AWS Lambda signature (context + event), though newer versions support standard HTTP.

```go
package main

import (
	"github.com/aws/aws-lambda-go/events"
	"github.com/aws/aws-lambda-go/lambda"
)

func handler(request events.APIGatewayProxyRequest) (*events.APIGatewayProxyResponse, error) {
	return &events.APIGatewayProxyResponse{
		StatusCode: 200,
		Body:       "Hello from Netlify Go!",
	}, nil
}

func main() {
	lambda.Start(handler)
}
```

## Interview Questions

**Q: When would you use Vercel/Netlify over raw AWS Lambda?**
**A:** Use them when you are building a web application (React, Next.js, Vue) and want tight coupling between frontend and backend without managing infrastructure. It allows a single developer to ship a full-stack app in minutes. Use raw AWS Lambda when you need complex event triggers (S3, DynamoDB streams), VPC access, or fine-grained IAM permissions that the abstracted platforms don't expose.

**Q: What are the limitations of Edge Functions?**
**A:** Edge functions run in a restricted environment (often V8 Isolates). They typically:
1.  Have a size limit (e.g., 1MB code size).
2.  Cannot use standard Node.js APIs (like `fs` for filesystem).
3.  May have trouble connecting to traditional databases (Postgres/MySQL) because creating TCP connections at the edge is slow and resource-heavy (though solutions like connection pooling proxies are mitigating this).

**Q: How does "Atomic Deployment" work on these platforms?**
**A:** Every time you push code, Vercel/Netlify creates a completely new, immutable deployment with a unique URL. Traffic is only switched to this new deployment after the build passes. If anything fails, the live site is untouched. This allows for instant rollbacks by simply pointing the DNS to the previous deployment's hash.
