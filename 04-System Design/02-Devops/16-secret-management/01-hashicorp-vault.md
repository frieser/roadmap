---
---

# HashiCorp Vault

HashiCorp Vault is the industry standard for identity-based secrets and encryption management. It goes beyond simple key storage to provide "Encryption as a Service" and dynamic, just-in-time credential generation.

## Summary

Vault centralizes secret management. Instead of hardcoding passwords, applications authenticate with Vault (using AppRole, Kubernetes, or IAM) and lease a secret. Vault's killer feature is **Dynamic Secrets**: it can create a temporary PostgreSQL user for your app and automatically revoke it after 1 hour, ensuring that even if the app is compromised, the credential is short-lived.

## Detailed Explanation

### 1. Core Concepts
*   **Secrets Engines**: Pluggable components that store or generate data.
    *   **KV (Key-Value)**: Static secrets (like AWS Secrets Manager).
    *   **Database**: Dynamic DB credentials.
    *   **PKI**: Generates X.509 certificates on the fly (Internal CA).
    *   **Transit**: Encryption as a Service (encrypt/decrypt data without storing it).
*   **Auth Methods**: How clients prove who they are.
    *   **Token**: The core method (like a session cookie).
    *   **AppRole**: Like a username/password for machines.
    *   **Kubernetes**: Validates Service Account Tokens (JWT).

### 2. The Lease Model
Almost everything in Vault has a **Lease**. When you read a secret, you get it for a limited time (TTL). You must renew the lease to keep using it. If the lease expires, Vault revokes the credential.

---

## Go Implementation Example

Using the official `hashicorp/vault/api` client to read a static secret from the KV engine.

```go
package main

import (
	"fmt"
	"log"
	"os"

	"github.com/hashicorp/vault/api"
)

func main() {
	// 1. Initialize Configuration (Reads VAULT_ADDR from env)
	config := api.DefaultConfig()
	client, err := api.NewClient(config)
	if err != nil {
		log.Fatalf("unable to initialize Vault client: %v", err)
	}

	// 2. Authenticate (Simplest: Token)
	// In prod, use client.Auth().Login(ctx, authMethod)
	token := os.Getenv("VAULT_TOKEN")
	client.SetToken(token)

	// 3. Read a Secret (KV Version 2)
	// Path: secret/data/myapp/config
	secret, err := client.Logical().Read("secret/data/myapp/config")
	if err != nil {
		log.Fatalf("unable to read secret: %v", err)
	}

	if secret == nil {
		log.Fatal("Secret not found")
	}

	// 4. Parse Data
	// KV v2 data structure is: secret.Data["data"] -> map[string]interface{}
	data, ok := secret.Data["data"].(map[string]interface{})
	if !ok {
		log.Fatalf("data type assertion failed: %T %#v", secret.Data["data"], secret.Data["data"])
	}

	apiKey, ok := data["api_key"].(string)
	if !ok {
		log.Fatal("api_key not found in secret")
	}

	fmt.Printf("Successfully retrieved API Key: %s\n", apiKey)
}
```

## Interview Questions

**Q: What is the difference between Vault and AWS Secrets Manager?**
**A:**
*   **Vault**: Cloud-agnostic. Supports on-prem, Azure, GCP, and AWS equally. Offers dynamic secrets (creating users on the fly) and encryption-as-a-service. Requires hosting (or HCP Vault).
*   **AWS Secrets Manager**: AWS-native. Deeply integrated with IAM. Primarily for static secrets with rotation (via Lambda). Fully managed (SaaS). Easier if you are 100% on AWS; Vault is better for hybrid/multi-cloud.

**Q: Explain "Dynamic Secrets" in Vault.**
**A:** Dynamic secrets are generated on-demand. When an app asks for a database password, Vault doesn't return a stored static password. Instead, it connects to the DB, creates a new user (e.g., `v-root-myrole-3h24k`), grants it specific permissions, and returns that credentials to the app. It then associates a TTL lease with that user and automatically deletes the user when the lease expires.

**Q: How do you authenticate a Kubernetes Pod with Vault?**
**A:** Using the **Kubernetes Auth Method**. The Pod sends its Service Account Token (JWT) to Vault. Vault verifies the token's validity and signature with the Kubernetes API Server (TokenReview API). If valid, Vault maps the Service Account to a Vault Role and returns a Vault Token to the Pod.
