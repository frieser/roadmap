---
---

# DynamoDB

Amazon DynamoDB is a fully managed NoSQL database service that provides fast and predictable performance with seamless scalability.

## 1. Data Modeling: Partition vs. Sort Keys

DynamoDB uses a **Composite Primary Key** structure:
- **Partition Key (PK)**: Used as input to an internal hash function to determine the physical partition where the item is stored. Crucial for data distribution and avoiding "hot partitions".
- **Sort Key (SK)**: Items with the same PK are stored together, sorted by SK. Allows for efficient range queries (`begins_with`, `between`, `>`, `<`).

## 2. Indices: GSI vs. LSI

- **Local Secondary Index (LSI)**:
    - Shares the same PK as the table but has a different SK.
    - Must be created when the table is created.
    - Shares throughput capacity with the main table.
- **Global Secondary Index (GSI)**:
    - Can have a completely different PK and SK.
    - Can be created or deleted at any time.
    - Has its own provisioned throughput capacity.

## 3. Single-Table Design

A design pattern where multiple entity types (e.g., Users, Orders, Products) are stored in a single table to enable complex queries in a single request.
- **Key Overloading**: Using generic names like `PK` and `SK` instead of `UserID` or `OrderID`.
- **Relationships**: Modeled using SK prefixing (e.g., `PK: USER#123`, `SK: ORDER#456`).

## 4. Consistency Models

- **Eventually Consistent Reads (Default)**: Returns data that might be stale but offers double the throughput (2 reads per RCU).
- **Strongly Consistent Reads**: Returns the most up-to-date data. Consumes double the RCU (1 read per RCU).
- **Transactional Reads/Writes**: Supports ACID transactions within a single AWS region across multiple items.

## 5. Capacity Modes

- **Provisioned Mode**: You specify the number of **Read Capacity Units (RCU)** and **Write Capacity Units (WCU)**. Cost-effective for predictable workloads.
- **On-Demand Mode**: You pay per request. Scales instantly with traffic. Ideal for unpredictable or spikey workloads.

## 6. Go Integration (aws-sdk-go-v2)

The modern SDK is `github.com/aws/aws-sdk-go-v2/service/dynamodb`.

### Example: Querying an Index in Go
```go
import (
    "context"
    "github.com/aws/aws-sdk-go-v2/service/dynamodb"
    "github.com/aws/aws-sdk-go-v2/aws"
)

func QueryIndex(client *dynamodb.Client) {
    out, err := client.Query(context.TODO(), &dynamodb.QueryInput{
        TableName: aws.String("MyTable"),
        IndexName: aws.String("MyGSI"),
        KeyConditionExpression: aws.String("GSI_PK = :pk"),
        ExpressionAttributeValues: map[string]types.AttributeValue{
            ":pk": &types.AttributeValueMemberS{Value: "STATUS#ACTIVE"},
        },
    })
}
```

## Interview Questions (Senior Level)

1. **Q: What is a "Hot Partition" and how do you fix it?**
   - **A**: Occurs when a single PK receives disproportionate traffic, exceeding the partition's limit (3000 RCU / 1000 WCU). Fix by choosing a PK with higher cardinality (e.g., UUID instead of Country) or adding a random suffix (salting).
2. **Q: When should you use GSI instead of LSI?**
   - **A**: Use GSI for most cases as it is more flexible (can be added later, different PK). Use LSI only if you need Strong Consistency on the index query.
3. **Q: Explain Single-Table Design vs. Multiple Tables.**
   - **A**: Single-Table Design reduces the number of round-trips to the DB by fetching related items in one query, but it increases modeling complexity and makes data migration harder.
