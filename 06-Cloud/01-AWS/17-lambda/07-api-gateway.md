#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'lambda']
---

## Summary
AWS Lambda is frequently paired with **Amazon API Gateway** to create serverless RESTful architectures. API Gateway acts as the entry point, handling HTTP requests, routing, and security, while Lambda provides the backend compute. Understanding the differences between **REST APIs** (feature-rich, legacy) and **HTTP APIs** (faster, cheaper, modern) is crucial for architectural decisions.

## Detailed Explanation

### REST vs. HTTP APIs
API Gateway offers two main flavors for RESTful services:

| Feature | REST API | HTTP API |
| :--- | :--- | :--- |
| **Cost** | Higher (standard pricing) | Lower (~71% cheaper) |
| **Latency** | Milliseconds overhead | Lower overhead |
| **Auth** | IAM, Cognito, Lambda Authorizers | JWT (Native), IAM, Lambda Authorizers |
| **Features** | API Keys, Usage Plans, WAF, Caching | Limited (mostly just CORS and OIDC) |
| **Integration** | Standard Proxy/Custom | Lambda Proxy 2.0 (Simpler) |

- **REST APIs**: Best for enterprise applications requiring WAF protection, API keys, or complex request/response transformations (VTL).
- **HTTP APIs**: Best for serverless backends where cost and performance are prioritized, and JWT-based authentication is sufficient.

### Lambda Proxy Integration
This is the recommended way to integrate API Gateway with Lambda.
- **Mechanism**: The entire HTTP request (headers, query params, body) is passed to the Lambda function as a single `event` object.
- **Response**: The Lambda function must return a specific JSON object:
  ```json
  {
    "isBase64Encoded": false,
    "statusCode": 200,
    "headers": { "content-type": "application/json" },
    "body": "{\"message\": \"Hello from Lambda!\"}"
  }
  ```
- **Differences**: 
  - **REST (Payload 1.0)**: Verbose event object.
  - **HTTP (Payload 2.0)**: Simplified event structure (e.g., cookies are a separate list, simplified header access).

### Lambda Authorizers
A Lambda function that controls access to your API using bearer token authentication or request parameters.
1. **Token-based**: Receives a header (like `Authorization`) and returns an IAM policy.
2. **Request-based**: Receives the entire request (headers, query string, stage variables) to make an auth decision.
3. **Caching**: Authorizers can cache the resulting IAM policy for a specified TTL (default 300s) to reduce latency and cost.

## Interview Questions

1. **When should you choose HTTP APIs over REST APIs?**
   - Choose HTTP APIs when you need low latency and low cost, and don't require advanced features like API keys, WAF integration, or request validation. It is ideal for modern serverless apps using OIDC/JWT.

2. **What is the difference between Lambda Proxy and Lambda non-proxy (Custom) integration?**
   - In **Proxy Integration**, API Gateway passes the raw request to Lambda and expects a specific response format. In **Non-Proxy Integration**, API Gateway uses VTL (Velocity Template Language) to transform the request before it reaches Lambda and transform the response before it reaches the client.

3. **How do Lambda Authorizers work?**
   - When a request hits API Gateway, it calls the Authorizer Lambda. The Lambda validates the credentials (e.g., JWT) and returns an IAM policy that allows or denies access to the specific API resource. It can also return context data passed to the backend Lambda.

4. **How does Payload Format 2.0 in HTTP APIs simplify development compared to 1.0?**
   - It provides a flatter event structure, separates cookies from headers, and simplifies the way path and query parameters are accessed, making the Lambda code cleaner and less prone to errors when parsing inputs.

5. **Can you protect an API Gateway endpoint with AWS WAF?**
   - Only **REST APIs** support direct AWS WAF integration. To protect an HTTP API with WAF, you would typically need to place it behind an Amazon CloudFront distribution and apply the WAF to CloudFront.
