---
---

# AWS Secrets Manager

AWS Secrets Manager is a fully managed service that helps you protect access to your applications, services, and IT resources. It enables you to easily rotate, manage, and retrieve database credentials, API keys, and other secrets throughout their lifecycle.

## Summary

Unlike Parameter Store (which is a general KV store), Secrets Manager is purpose-built for **Secrets**. Its defining feature is **Automatic Rotation**: it can natively rotate RDS passwords without breaking connections, and trigger Lambda functions to rotate custom secrets (like API keys). It provides replication to multiple regions for disaster recovery.

## Detailed Explanation

### 1. Key Features
*   **Rotation**: The ability to change a password automatically. For RDS, this is built-in. For other services, you provide a Lambda function. Rotation involves 4 steps: Create, Set, Test, Finish.
*   **Versioning**: Secrets can have multiple versions. `AWSCURRENT` is the active version. When rotation happens, the new secret becomes `AWSPENDING` until it is verified.
*   **Integration**: Applications retrieve secrets at runtime via the SDK, removing the need for hardcoded config files.

### 2. Cost
Secrets Manager costs per secret per month ($0.40) + per 10,000 API calls. Parameter Store (Standard) is free. This is a key design consideration.

---

## Go Implementation Example

Retrieving a secret using `aws-sdk-go-v2`.

```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/aws/aws-sdk-go-v2/aws"
	"github.com/aws/aws-sdk-go-v2/config"
	"github.com/aws/aws-sdk-go-v2/service/secretsmanager"
)

func main() {
	secretName := "prod/myapp/database"
	region := "us-east-1"

	// 1. Load Config
	cfg, err := config.LoadDefaultConfig(context.TODO(), config.WithRegion(region))
	if err != nil {
		log.Fatalf("unable to load SDK config, %v", err)
	}

	// 2. Create Client
	svc := secretsmanager.NewFromConfig(cfg)

	// 3. Get Secret Value
	input := &secretsmanager.GetSecretValueInput{
		SecretId:     aws.String(secretName),
		VersionStage: aws.String("AWSCURRENT"), // Default to current version
	}

	result, err := svc.GetSecretValue(context.TODO(), input)
	if err != nil {
		log.Fatalf("failed to retrieve secret: %v", err)
	}

	// 4. Output Secret
	// Secrets can be String or Binary
	if result.SecretString != nil {
		fmt.Printf("Secret String: %s\n", *result.SecretString)
	} else {
		// Handle binary secret (decoded from Base64 automatically by SDK)
		fmt.Printf("Secret Binary length: %d\n", len(result.SecretBinary))
	}
}
```

## Interview Questions

**Q: Why use Secrets Manager over SSM Parameter Store?**
**A:** Use Secrets Manager if you need **Automatic Rotation**, cross-region replication of secrets, or integration with RDS/Redshift for credential management. Use SSM Parameter Store if you want a free/cheap store for configuration data (endpoints, flags) or static secrets that rarely change.

**Q: Explain the Rotation Lifecycle in AWS Secrets Manager.**
**A:** When rotation is triggered, Secrets Manager calls a Lambda function which performs four steps:
1.  **createSecret**: Generates a new password.
2.  **setSecret**: Updates the database/service with the new password (as a pending user).
3.  **testSecret**: Tries to log in with the new password to verify it works.
4.  **finishSecret**: Promotes the new password to `AWSCURRENT` status.

**Q: How do you handle secrets during local development?**
**A:** Developers should typically *not* have access to production secrets. Best practice is to use a local `.env` file for dev secrets, and use the AWS SDK in the application code. The code detects the environment: if `GO_ENV=local`, read `.env`; if `GO_ENV=prod`, fetch from Secrets Manager. This abstracts the secret source.
