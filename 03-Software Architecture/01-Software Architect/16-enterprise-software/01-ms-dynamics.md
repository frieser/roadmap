---
---

# Microsoft Dynamics 365

## Summary
Microsoft Dynamics 365 is a suite of enterprise resource planning (ERP) and customer relationship management (CRM) business applications. It leverages the Power Platform and Dataverse to provide a unified data layer, hosted natively on Microsoft Azure.

---

## 1. Core Concepts

### Dynamics 365 Sales (CRM) vs. Finance & Operations (ERP)
*   **D365 Sales (CRM)**: Focuses on front-office operations: sales pipelines, marketing automation, and customer service. It is built natively on **Dataverse**.
*   **D365 Finance & Operations (ERP)**: Designed for back-office operations: supply chain, manufacturing, and complex financial reporting. Historically a separate stack (X++), it now integrates with the Power Platform via **Virtual Tables** and **Dual-write**.

### The Role of Dataverse (formerly CDS)
Dataverse is the backbone of the Dynamics 365 ecosystem. It is a cloud-based, low-code data platform that provides:
*   **Standardized Schema**: The Common Data Model (CDM) provides predefined tables (Accounts, Contacts).
*   **Logic & Validation**: Server-side business rules, workflows, and calculated columns.
*   **Security**: Rich RBAC (Role-Based Access Control) and row-level security.
*   **API-First**: Every table is automatically exposed via an OData v4 Web API.

---

## 2. Architecture

### Cloud-Native (Azure Backend)
Dynamics 365 is a SaaS offering built on the Azure stack:
*   **Storage**: Azure SQL for structured data, Azure Blob for attachments, and Azure Data Lake for analytics.
*   **Identity**: Microsoft Entra ID (Azure AD) for authentication and service principal management.
*   **Compute**: Microservices-based architecture with global scalability.

### Extensibility Models
1.  **Plugins (Pro-code)**: Compiled C# assemblies (.NET) that run in the Dataverse Event Framework (Synchronous or Asynchronous).
2.  **Power Automate (Low-code)**: Cloud flows for cross-system orchestration and simple automation.
3.  **Client-side**: JavaScript (TypeScript) using the Client API to manipulate forms and UI behavior.

---

## 3. Integration: OData v4 Web API
The **Dataverse Web API** implements OData v4, allowing standard RESTful interactions:
*   **Endpoint**: `https://<org>.api.crm.dynamics.com/api/data/v9.2/`
*   **Queries**: Supports `$select`, `$filter`, `$expand` (joins), and `$orderby`.
*   **Batching**: Supports `$batch` requests to execute multiple operations in a single HTTP request.

---

## 4. Go Implementation: OData Client
Example of a Go client authenticating via **OAuth2 Client Credentials** and querying Dataverse.

```go
package main

import (
	"context"
	"fmt"
	"io"
	"log"
	"net/http"

	"golang.org/x/oauth2/clientcredentials"
)

func main() {
	// 1. Configuration (Environment Details)
	tenantID := "YOUR_TENANT_ID"
	clientID := "YOUR_CLIENT_ID"
	clientSecret := "YOUR_CLIENT_SECRET"
	resource := "https://org.crm.dynamics.com" // Environment URL
	apiURL := resource + "/api/data/v9.2/accounts?$select=name,accountnumber&$top=3"

	// 2. Setup OAuth2 Client Credentials Config
	config := &clientcredentials.Config{
		ClientID:     clientID,
		ClientSecret: clientSecret,
		TokenURL:     fmt.Sprintf("https://login.microsoftonline.com/%s/oauth2/v2.0/token", tenantID),
		Scopes:       []string{resource + "/.default"},
	}

	// 3. Obtain an Authenticated HTTP Client
	client := config.Client(context.Background())

	// 4. Query Dynamics 365 Entities (OData)
	resp, err := client.Get(apiURL)
	if err != nil {
		log.Fatalf("Request failed: %v", err)
	}
	defer resp.Body.Close()

	if resp.StatusCode != http.StatusOK {
		body, _ := io.ReadAll(resp.Body)
		log.Fatalf("Unexpected status: %d\nBody: %s", resp.StatusCode, string(body))
	}

	// 5. Read and Display Results
	body, _ := io.ReadAll(resp.Body)
	fmt.Printf("Dataverse Response:\n%s\n", string(body))
}
```

---

## 5. Interview Preparation

### Q1: What is the difference between a Plugin and a Workflow?
*   **Answer**: Plugins are **C# code** that run synchronously or asynchronously within the Dataverse transaction, offering high performance and complex logic. Workflows (Classic or Power Automate) are **designer-based**; Classic workflows can be synchronous (Real-time), but Power Automate is strictly asynchronous and designed for integration.

### Q2: Explain the Dataverse Security Model.
*   **Answer**: It is a hierarchical model based on **Business Units (BUs)**. **Security Roles** define privileges (Create, Read, Update, etc.) and access levels (Global, Deep, Local, Basic). It also supports **Field-Level Security** for sensitive data and **Hierarchy Security** for manager-based access.

### Q3: What are "Virtual Tables" in the context of D365 F&O integration?
*   **Answer**: Virtual Tables allow Dataverse to surface data from an external source (like D365 Finance & Operations) in real-time without physical data synchronization. This enables Power Apps or CRM to interact with ERP data as if it were native Dataverse tables.

### Q4: How does D365 handle API Throttling?
*   **Answer**: Dataverse enforces **Service Protection Limits**. If a client exceeds the request limit, the API returns a `429 Too Many Requests` status code. Architects should implement **Exponential Backoff** by reading the `Retry-After` header in the response.

### Q5: What is the purpose of "Solutions" in Dynamics 365?
*   **Answer**: Solutions are the containers for Application Lifecycle Management (ALM). They package components (Tables, Plugins, Flows, UI) for transport between environments (Dev → Test → Prod). Solutions can be **Managed** (read-only in target) or **Unmanaged** (editable).
