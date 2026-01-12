#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'dynamodb']
---

## Summary
DynamoDB data modeling is a fundamental shift from relational databases, prioritizing **Access Patterns** over normalization. By using **Single Table Design**, developers can store multiple entity types in one table, leveraging overloaded keys (PK/SK) to fetch related data in a single request. This approach is highly scalable and cost-effective but requires a strict understanding of how the application will query data before the schema is finalized.

## Detailed Explanation

### 1. Access Patterns First
In DynamoDB, you do not model the data; you model the **queries**. 
1.  **List all queries**: Before creating a table, define every way your application will read/write data (e.g., "Get user by ID", "Get all orders for a user").
2.  **Define Primary Keys**: Choose Partition Keys (PK) and Sort Keys (SK) that satisfy these patterns.

### 2. Overloading Keys (Single Table Design)
Single Table Design uses generic attribute names to store different types of data:
- **PK (Partition Key)**: Used for horizontal scaling and grouping related items.
- **SK (Sort Key)**: Used for sorting data within a partition and complex filtering.

**Example Schema:**
| PK | SK | Type | Attributes |
| :--- | :--- | :--- | :--- |
| `USER#123` | `USER#123` | User | Name: John, Email: john@example.com |
| `USER#123` | `ORDER#999` | Order | Amount: $50, Status: Shipped |
| `PRODUCT#XYZ` | `PRODUCT#XYZ` | Product | Name: Laptop, Price: $1200 |

### 3. Modeling Relationships

#### 1:N (One-to-Many)
*   **Item Collections**: The most efficient way. Parent and child share the same `PK`. A single `Query` operation on the `PK` returns the parent and all children.
*   **Global Secondary Index (GSI)**: If you need to query children by a different attribute (e.g., "Get all orders by Status").

#### M:N (Many-to-Many)
*   **Adjacency List Pattern**: Represents a graph. For a many-to-many relationship (e.g., Users and Groups):
    1.  Store an item for the relationship: `PK: USER#123`, `SK: GROUP#ABC`.
    2.  Use a **GSI** where the keys are inverted: `GSI1_PK: GROUP#ABC`, `GSI1_SK: USER#123`.
    3.  Query `PK=USER#123` to find groups for a user.
    4.  Query `GSI1_PK=GROUP#ABC` to find users in a group.

### 4. Go Implementation (AWS SDK V2)
Using the `attributevalue` package to map Go structs to DynamoDB items.

```go
package main

import (
	"context"
	"fmt"
	"github.com/aws/aws-sdk-go-v2/aws"
	"github.com/aws/aws-sdk-go-v2/feature/dynamodb/attributevalue"
	"github.com/aws/aws-sdk-go-v2/service/dynamodb"
	"github.com/aws/aws-sdk-go-v2/service/dynamodb/types"
)

type User struct {
	PK   string `dynamodbav:"PK"`
	SK   string `dynamodbav:"SK"`
	Name string `dynamodbav:"Name"`
}

// GetUserAndOrders fetches an item collection (Parent + Children)
func GetUserAndOrders(ctx context.Context, client *dynamodb.Client, userId string) {
	pkValue := fmt.Sprintf("USER#%s", userId)

	out, err := client.Query(ctx, &dynamodb.QueryInput{
		TableName:              aws.String("MySingleTable"),
		KeyConditionExpression: aws.String("PK = :pk"),
		ExpressionAttributeValues: map[string]types.AttributeValue{
			":pk": &types.AttributeValueMemberS{Value: pkValue},
		},
	})
	if err != nil {
		panic(err)
	}

	var results []map[string]interface{}
	err = attributevalue.UnmarshalListOfMaps(out.Items, &results)
	if err != nil {
		panic(err)
	}

	for _, item := range results {
		fmt.Printf("Fetched Item: %v\n", item)
	}
}
```

## Interview Questions
1. **Q: What is the main benefit of Single Table Design?**
   **A:** It allows for fetching multiple related entity types in a single I/O operation (Query), minimizing latency and costs associated with multiple requests.

2. **Q: How do you handle a query that isn't supported by your Primary Key?**
   **A:** You use a Global Secondary Index (GSI). GSIs allow you to define an alternative PK and SK for the same data, enabling different access patterns.

3. **Q: Explain the Adjacency List pattern.**
   **A:** It is a pattern for modeling Many-to-Many relationships or hierarchical data. It involves creating items that represent the links between entities and using GSIs to query those links from either direction.

4. **Q: When should you avoid Single Table Design?**
   **A:** Avoid it when your access patterns are unknown (ad-hoc queries), when you need to perform heavy analytical queries (use Redshift/Athena instead), or when the complexity of managing overloaded keys outweighs the performance benefits.

5. **Q: What is the purpose of the Sort Key in a 1:N relationship?**
   **A:** It allows for sorting children (e.g., by date) and enables range queries (e.g., `begins_with`, `between`) to fetch specific subsets of related items.
