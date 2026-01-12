#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'dynamodb']
---

## Summary
**DynamoDB Local** is a downloadable version of Amazon DynamoDB that enables developers to build, test, and debug applications offline. It provides a client-side database that supports the DynamoDB API, allowing for rapid iteration without incurring AWS costs, consuming provisioned throughput, or requiring an internet connection. When moving to production, you simply switch the endpoint URL from `localhost` to the actual AWS service.

## Detailed Explanation

### 1. Running DynamoDB Local
DynamoDB Local can be deployed in several ways depending on your development environment:

*   **Docker (Recommended):** The easiest way to get started. Use the official Amazon image:
    ```bash
    docker run -p 8000:8000 amazon/dynamodb-local
    ```
*   **Executable JAR:** Requires Java Runtime Environment (JRE). You download a `.zip` or `.tar.gz`, extract it, and run:
    ```bash
    java -Djava.library.path=./DynamoDBLocal_lib -jar DynamoDBLocal.jar -sharedDb
    ```
*   **NoSQL Workbench:** A desktop application for data modeling and visualization that includes a built-in DynamoDB Local instance.

### 2. Key Command-Line Options
*   `-sharedDb`: DynamoDB uses a single database file (`shared-local-instance.db`). If omitted, it creates separate files based on the Access Key ID and Region.
*   `-inMemory`: Runs the database entirely in RAM. All data is lost when the process terminates.
*   `-port`: Changes the default port from `8000`.
*   `-dbPath`: Specifies the directory where the database file should be saved.

### 3. Usage and Connectivity
To connect to the local instance, you must override the **Endpoint URL** in your tools and SDKs.

*   **AWS CLI:**
    ```bash
    aws dynamodb list-tables --endpoint-url http://localhost:8000
    ```
*   **Credentials:** You can use any dummy strings for Access Key and Secret Key, but they must be present.

### 4. Differences from the Web Service
While DynamoDB Local aims for high compatibility, there are critical differences:
*   **Throughput:** Provisioned throughput settings (RCUs/WCUs) are accepted in API calls but **ignored**. Throttling does not occur.
*   **Case Sensitivity:** Table names are **case-insensitive** in Local (e.g., `Users` and `users` are the same), whereas they are case-sensitive in the real service.
*   **Scanning:** Parallel scans are not supported; they are performed sequentially.
*   **Transactions:** `TransactionConflictExceptions` are not thrown by Local.
*   **Query Filtering:** When querying an index, Local calculates the size of the **entire item** against the 1 MB limit, whereas the web service only calculates the size of projected attributes.

### 5. Go (Golang) Implementation
To use DynamoDB Local with the AWS SDK for Go v2, you need to provide a custom endpoint and static dummy credentials.

```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/aws/aws-sdk-go-v2/aws"
	"github.com/aws/aws-sdk-go-v2/config"
	"github.com/aws/aws-sdk-go-v2/credentials"
	"github.com/aws/aws-sdk-go-v2/service/dynamodb"
)

func main() {
	ctx := context.TODO()

	// 1. Load configuration with static dummy credentials
	cfg, err := config.LoadDefaultConfig(ctx,
		config.WithRegion("us-west-2"),
		config.WithCredentialsProvider(credentials.StaticCredentialsProvider{
			Value: aws.Credentials{
				AccessKeyID:     "local-dev",
				SecretAccessKey: "local-dev-secret",
				Source:          "Local Development",
			},
		}),
	)
	if err != nil {
		log.Fatalf("unable to load SDK config, %v", err)
	}

	// 2. Initialize client with the Local Endpoint
	client := dynamodb.NewFromConfig(cfg, func(o *dynamodb.Options) {
		o.BaseEndpoint = aws.String("http://localhost:8000")
	})

	// 3. Perform operations
	resp, err := client.ListTables(ctx, &dynamodb.ListTablesInput{})
	if err != nil {
		log.Fatalf("failed to list tables, %v", err)
	}

	fmt.Println("Successfully connected to DynamoDB Local!")
	fmt.Printf("Local Tables: %v\n", resp.TableNames)
}
```

## Interview Questions

**Q: Why would you use DynamoDB Local instead of a dedicated "Dev" table in AWS?**
**A:** DynamoDB Local allows for completely offline development, eliminates latency costs associated with network calls, and prevents any accidental charges or throughput consumption during intensive testing or local CI/CD pipelines.

**Q: How do you ensure data persists across restarts when using DynamoDB Local in Docker?**
**A:** You must mount a volume to the container's data directory (usually `/home/dynamodblocal/data`) and ensure you run the process with the `-dbPath ./data` option (or similar) within the container. By default, Docker containers are ephemeral.

**Q: What is a major pitfall when testing Scan operations locally vs in production?**
**A:** DynamoDB Local does not support parallel scans. If your application relies on `TotalSegments` and `Segment` parameters to speed up large scans, you won't be able to test that specific concurrency logic locally; it will execute sequentially.

**Q: Does DynamoDB Local enforce IAM policies?**
**A:** No. DynamoDB Local ignores IAM policies and permissions. Any validly formatted credential pair will grant full access to the local instance.

**Q: How does `-sharedDb` affect data isolation?**
**A:** If `-sharedDb` is used, all connections share the same data file regardless of credentials. If it's NOT used, DynamoDB creates separate data files for each unique Access Key and Region combination, allowing for isolated "environments" within the same local process.
