---
---

# Microsoft Azure

Azure is Microsoft's public cloud computing platform. It is a strong competitor to AWS, particularly in enterprise environments that already rely heavily on the Microsoft ecosystem (Active Directory, Windows Server, .NET).

## Summary

Azure offers a similar breadth of services to AWS but uses different terminology (e.g., Resource Groups). Its key differentiator is its integration with enterprise identity (Azure AD / Entra ID) and its hybrid cloud capabilities (Azure Arc). The **Azure SDK for Go** is modern and idiomatic, making it easy to automate resource management.

## Detailed Explanation

### 1. Core Services (vs AWS)
*   **Virtual Machines**: Equivalent to EC2.
*   **Blob Storage**: Equivalent to S3.
*   **Azure SQL Database**: Managed SQL Server.
*   **AKS (Azure Kubernetes Service)**: Managed Kubernetes (often considered easier to use than EKS).
*   **Resource Groups**: Logical containers for resources. Every resource *must* belong to a Resource Group.

### 2. DevOps on Azure
*   **Azure DevOps**: A complete suite (Boards, Repos, Pipelines, Artifacts) that predates GitHub Actions but remains widely used.
*   **ARM Templates / Bicep**: Azure's native IaC languages.

---

## Go Implementation Example

Azure's Go SDK uses the `azidentity` package for authentication (supporting CLI login, Managed Identity, etc.) and service-specific modules.

```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/Azure/azure-sdk-for-go/sdk/azidentity"
	"github.com/Azure/azure-sdk-for-go/sdk/resourcemanager/resources/armresources"
)

func main() {
	subscriptionID := "your-subscription-id"
	resourceGroupName := "devops-demo-rg"
	location := "eastus"

	// 1. Authenticate (Uses Environment Vars, CLI login, or Managed Identity)
	cred, err := azidentity.NewDefaultAzureCredential(nil)
	if err != nil {
		log.Fatalf("Authentication failure: %v", err)
	}

	// 2. Create Resource Group Client
	client, err := armresources.NewResourceGroupsClient(subscriptionID, cred, nil)
	if err != nil {
		log.Fatal(err)
	}

	// 3. Create Resource Group
	ctx := context.Background()
	resp, err := client.CreateOrUpdate(ctx, resourceGroupName, armresources.ResourceGroup{
		Location: &location,
	}, nil)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("Resource Group Created: %s (ID: %s)\n", *resp.Name, *resp.ID)
}
```

## Interview Questions

**Q: What is a Resource Group and why is it important in Azure?**
**A:** A Resource Group is a logical container that holds related resources for an Azure solution. Resources in a group share the same lifecycle: you can deploy, update, and delete them as a single unit. For DevOps, this is crucial for environment cleanup—deleting the Resource Group `dev-env` deletes everything inside it (VMs, IPs, Storage) instantly.

**Q: How does Azure AD (Entra ID) integration benefit DevOps?**
**A:** It allows for centralized identity management. Instead of creating local users on Linux VMs or database users in SQL, you can use Entra ID identities. Developers can SSH into VMs or connect to databases using their corporate credentials, and access can be revoked centrally.

**Q: What is the difference between Azure Functions and Azure App Service?**
**A:** **App Service** is a PaaS for hosting web applications (like a standard Go web server) where you pay for reserved compute power (App Service Plan). **Azure Functions** is a Serverless (FaaS) platform where you run event-driven code snippets and pay only for execution time.
