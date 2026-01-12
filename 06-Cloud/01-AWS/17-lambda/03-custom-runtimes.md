#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'lambda']
---

## Summary
AWS Lambda Custom Runtimes allow developers to run Lambda functions in any programming language or version not natively supported by AWS (e.g., Rust, C++, or specific PHP/Ruby versions). By using the `provided.al2` or `provided.al2023` runtimes, you provide an executable named `bootstrap` that interacts with the Lambda Runtime API to manage the function's lifecycle.

## Detailed Explanation

### The Runtime API
The Runtime API is an HTTP interface that the custom runtime uses to communicate with the Lambda service. When a Lambda function is initialized, the service starts the `bootstrap` file. This file must interact with the API using the endpoint defined in the `AWS_LAMBDA_RUNTIME_API` environment variable.

The interaction follows a specific workflow:
1. **Initialization**: The runtime performs one-time setup tasks.
2. **Event Loop**:
   - **Get Next Invocation**: The runtime calls `GET http://${AWS_LAMBDA_RUNTIME_API}/2018-06-01/runtime/invocation/next` to receive the next event and its headers (like `Lambda-Runtime-Aws-Request-Id`).
   - **Execute Handler**: The runtime passes the event data to the function handler.
   - **Send Response**: The runtime calls `POST http://${AWS_LAMBDA_RUNTIME_API}/2018-06-01/runtime/invocation/{AwsRequestId}/response` with the handler's result.
   - **Handle Errors**: If an error occurs, the runtime calls the error endpoint for that request ID.

### The `bootstrap` Executable
The `bootstrap` file is the entry point for your custom runtime. It can be a compiled binary (like Go or Rust) or a shell script that starts another interpreter.
- **Location**: It must be at the root of the deployment package or in a Lambda Layer.
- **Permissions**: It must have executable permissions (`chmod +x bootstrap`).
- **Responsibility**: It is responsible for the entire lifecycle: initializing the runtime environment, fetching events, and returning responses.

### Amazon Linux 2 and AL2023
Custom runtimes are typically built on:
- **`provided.al2`**: Based on Amazon Linux 2.
- **`provided.al2023`**: The latest OS-only runtime based on Amazon Linux 2023, offering a smaller footprint, updated libraries (like `glibc`), and improved security.

Using these "OS-only" runtimes gives you full control over the execution environment while AWS manages the underlying patching and scaling of the OS.

## Interview Questions

### 1. What is the purpose of the `bootstrap` file in an AWS Lambda custom runtime?
The `bootstrap` file is the executable entry point that Lambda starts when the function is initialized. Its job is to manage the interaction with the Lambda Runtime API, retrieve invocation events, pass them to the handler code, and return the results or errors back to the Lambda service.

### 2. How does a custom runtime receive event data from Lambda?
It receives event data by making a `GET` request to the Runtime API's "next invocation" endpoint: `http://${AWS_LAMBDA_RUNTIME_API}/2018-06-01/runtime/invocation/next`. This request blocks until an event is available, and returns the event payload in the body and metadata (like Request ID) in the HTTP headers.

### 3. What are the advantages of using `provided.al2023` over `provided.al2`?
`provided.al2023` is based on Amazon Linux 2023, which provides a more modern and secure environment. Key advantages include a smaller deployment footprint (minimal image), updated system libraries (newer `glibc`), a new package manager (`dnf`), and support for modern protocols and security standards.

### 4. Can you use Lambda Layers with custom runtimes?
Yes. In fact, Lambda Layers are a common way to distribute custom runtimes. You can place the `bootstrap` executable in a layer so that multiple functions can share the same runtime environment without bundling it in every deployment package.

### 5. What happens if the `bootstrap` file exits?
If the `bootstrap` process exits, the Lambda execution environment is considered failed. If the exit happens during initialization, Lambda will attempt to restart the environment. If it happens during an invocation, the request will return an error, and the environment will be recycled.
