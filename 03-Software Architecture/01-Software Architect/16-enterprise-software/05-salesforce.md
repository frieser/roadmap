---
---

# Salesforce

## Summary
Salesforce is more than a CRM; it is a **PaaS (Platform as a Service)** built on a metadata-driven, multi-tenant architecture. For a Software Architect, understanding how Salesforce manages shared resources and integrates with external systems is critical.

## Detailed Explanation

### 1. Core Architecture Concepts

*   **Multi-tenant Architecture**: Salesforce uses a single pool of resources (CPU, Memory, DB) to serve multiple customers ("tenants"). Data is logically isolated via a `Tenant_ID` on every database row.
    *   **The Apartment Analogy**: Tenants share the building's infrastructure (pipes, electricity) but have their own locked apartments.
*   **Metadata-Driven Kernel**: The platform doesn't compile code into machine language for every tenant. Instead, it stores "metadata" (configs, object schemas, UI layouts) in a database and renders the application at runtime.
*   **Apex (Backend)**: A proprietary, strongly typed, object-oriented language. It is "save-on-server" and executes within the Salesforce runtime.
*   **Lightning Web Components (LWC)**: A modern UI framework built on standard Web Components. It leverages a lightweight core and performance-optimized shadow DOM.

### 2. Data Access: SOQL vs. SOSL

| Feature | **SOQL** (Object Query Language) | **SOSL** (Object Search Language) |
| :--- | :--- | :--- |
| **Purpose** | Precise data retrieval from specific objects. | Full-text search across multiple objects. |
| **Syntax** | `SELECT Name FROM Account WHERE Industry = 'Tech'` | `FIND {SearchTerm} IN ALL FIELDS RETURNING Account(Name), Contact(FirstName)` |
| **Performance** | Faster for single-object lookups with indexed filters. | Optimized for text matching and multi-object searches. |
| **Return Type** | List of SObjects. | List of List of SObjects. |

### 3. Integration Patterns

*   **REST API**: Best for synchronous, real-time CRUD operations. Uses OAuth 2.0.
*   **Bulk API 2.0**: Designed for "Big Data" (millions of records). It processes data in the background (asynchronous).
*   **Platform Events**: An event-driven bus (based on CometD/Pub-Sub) for near real-time integration. Very useful for decoupled architectures.

## Go Implementation
A concise example using the Salesforce REST API via standard Go libraries and OAuth2.

```go
package main

import (
	"context"
	"encoding/json"
	"fmt"
	"net/http"
	"net/url"
	"golang.org/x/oauth2"
)

type QueryResponse struct {
	TotalSize int `json:"totalSize"`
	Records   []struct {
		Name string `json:"Name"`
	} `json:"records"`
}

func main() {
	// 1. Setup OAuth2 Config (User-Agent Flow or JWT is preferred for Server-to-Server)
	conf := &oauth2.Config{
		ClientID:     "YOUR_CLIENT_ID",
		ClientSecret: "YOUR_CLIENT_SECRET",
		Endpoint: oauth2.Endpoint{
			TokenURL: "https://login.salesforce.com/services/oauth2/token",
		},
	}

	// 2. Get Token (Example using Password Flow - use JWT in Prod)
	token, _ := conf.PasswordCredentialsToken(context.Background(), "user@org.com", "password+security_token")
	client := conf.Client(context.Background(), token)

	// 3. Execute SOQL Query via REST API
	query := url.QueryEscape("SELECT Name FROM Account LIMIT 5")
	apiURL := fmt.Sprintf("https://your-instance.my.salesforce.com/services/data/v62.0/query?q=%s", query)

	resp, _ := client.Get(apiURL)
	defer resp.Body.Close()

	var result QueryResponse
	json.NewDecoder(resp.Body).Decode(&result)

	for _, rec := range result.Records {
		fmt.Printf("Account Name: %s\n", rec.Name)
	}
}
```

## Interview Questions

**Q: What are Governor Limits and why do they exist?**
**A:** Governor Limits are runtime thresholds enforced by the Apex engine (e.g., 100 SOQL queries per transaction). They exist to prevent a single tenant from monopolizing shared resources (CPU, Heap, DB) in the multi-tenant environment.

**Q: Describe the "One Trigger Per Object" / Handler Pattern.**
**A:** To maintain order and predictability, architects recommend having exactly one trigger per object. This trigger delegates logic to a **Handler Class**. This prevents "race conditions" between multiple triggers and makes the logic reusable and testable.

**Q: What is the difference between a "Before" and "After" Trigger?**
**A:** 
*   **Before**: Used for validating or updating fields on the *same* record (faster, no need for an explicit `update` DML).
*   **After**: Used for logic that requires the record ID (which is generated after the save) or for updating *related* records.

**Q: How do you handle large data volumes (LDV) in Salesforce?**
**A:** Use **Skinny Tables** to join fields, ensure fields used in filters are **indexed**, and use **Batch Apex** or the **Bulk API** for processing. Avoid "Non-selective queries" which scan the whole table.

**Q: What are the different types of Asynchronous Apex?**
**A:** 
1.  **Future Methods**: Simple async execution.
2.  **Queueable Apex**: Supports complex types and chaining.
3.  **Batch Apex**: For millions of records.
4.  **Scheduled Apex**: Runs at specific times.
