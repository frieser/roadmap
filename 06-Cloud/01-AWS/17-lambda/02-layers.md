#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'lambda']
---

## Summary
**AWS Lambda Layers** is a mechanism to centrally manage and share dependencies, custom runtimes, and other shared resources across multiple Lambda functions. Instead of bundling all libraries within each function's deployment package, you move them into a layer, keeping the function code small, focused, and easier to manage in the AWS console.

## Detailed Explanation

### Core Concepts
When a Lambda function is invoked, AWS extracts the layer contents into the `/opt` directory of the execution environment. This directory is included in the runtime's search path (e.g., `PYTHONPATH`, `NODE_PATH`), allowing your code to import libraries as if they were local.

### `/opt` Directory Structure
To ensure the runtime finds your dependencies, you must follow a specific directory structure within the layer's `.zip` file:

| Runtime | Path inside ZIP | Location in `/opt` |
| --- | --- | --- |
| **Python** | `python/` | `/opt/python/` |
| **Node.js** | `nodejs/node_modules/` | `/opt/nodejs/node_modules/` |
| **Java** | `java/lib/` | `/opt/java/lib/` |
| **Ruby** | `ruby/gems/x.x.x/` | `/opt/ruby/gems/x.x.x/` |
| **Binaries** | `bin/` | `/opt/bin/` |

### Benefits of Using Layers
1. **Reduced Package Size**: Keeping deployment packages under 50MB allows for direct code editing in the AWS Console.
2. **Code Reusability**: Share common utilities (e.g., logging, database wrappers) across hundreds of functions.
3. **Faster Deployments**: You don't need to upload heavy libraries (like `pandas` or `scikit-learn`) every time you change a line of business logic.
4. **Separation of Concerns**: Operations can manage shared dependencies/runtimes while developers focus on function logic.

### Constraints and Limits
* **Maximum Layers**: You can attach up to **5 layers** per Lambda function.
* **Unzipped Size Limit**: The total unzipped size of the function code + all layers must not exceed **250 MB**.
* **Order Matters**: Layers are merged in the order they are defined. If two layers contain the same file, the one later in the list overrides the earlier one.
* **Immutability**: Layer versions are immutable. To update a layer, you must publish a new version.

### Sharing and Permissions
Layers can be:
* **Private**: Only accessible within the owner's AWS account.
* **Shared**: Accessible by specific AWS accounts or organizations via resource-based policies.
* **Public**: Available to any AWS user (e.g., AWS SDK layers or community-provided runtimes).

## Interview Questions

### 1. What happens if two layers attached to a Lambda function contain the same file at the same path?
AWS Lambda merges the layers into the `/opt` directory in the order specified in the function configuration. If a file collision occurs, the version from the **last layer** (the one with the highest index in the list) will overwrite previous ones.

### 2. Can you use a Lambda Layer across different AWS Regions?
No. Lambda Layers are **regional** resources. To use a layer in multiple regions, you must upload and publish the layer to each specific region separately.

### 3. How do you reference a dependency located in a Layer within your Python code?
Since the Python runtime automatically includes `/opt/python` in its `sys.path`, you can simply use the standard import statement: `import my_shared_library`. You do not need to reference the `/opt` path explicitly in your code.

### 4. What is the impact of Layers on Cold Starts?
Layers generally have **negligible impact** on cold start times compared to bundling the same dependencies in the deployment package. However, the total unzipped size (250MB limit) remains the primary factor; larger total code sizes increase the time AWS takes to initialize the execution environment.

### 5. Why might you use a Custom Runtime as a Layer?
If you want to run a language not natively supported by AWS (like Rust, PHP, or a specific version of C++), you can package the runtime's executable and a bootstrap file into a layer. This allows the Lambda service to invoke your custom code via the Runtime API.
