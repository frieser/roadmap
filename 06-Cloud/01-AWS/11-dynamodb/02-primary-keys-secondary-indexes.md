#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'dynamodb']
---

## Summary
DynamoDB uses primary keys to uniquely identify items and secondary indexes to provide flexible query patterns. A **Primary Key** can be a simple **Partition Key (Hash)** or a composite **Partition Key + Sort Key (Range)**. **Local Secondary Indexes (LSI)** allow alternate sort keys for the same partition key, while **Global Secondary Indexes (GSI)** allow queries across all partitions using entirely different keys. Understanding the trade-offs in consistency, throughput, and storage limits between these indexing strategies is critical for efficient NoSQL data modeling.

## Detailed Explanation

### 1. Primary Keys
Every DynamoDB table must have a primary key defined at creation.

*   **Partition Key (PK):** Also known as the *Hash Attribute*. DynamoDB uses the PK's value as input to an internal hash function to determine the physical partition where the item is stored.
*   **Sort Key (SK):** Also known as the *Range Attribute*. Items with the same PK are stored physically close together and sorted by the SK value. This enables efficient "range" queries (e.g., `begins_with`, `between`, `>`, `<`).

### 2. Secondary Indexes
Secondary indexes are "materialized views" of your table data, automatically synchronized by DynamoDB.

#### Local Secondary Index (LSI)
*   **Scope:** Local to a single partition (same PK as base table).
*   **Constraint:** Must be created during table creation. Cannot be added later.
*   **Consistency:** Supports both **Strongly Consistent** and **Eventually Consistent** reads.
*   **Capacity:** Shares Read/Write Capacity Units (RCU/WCU) with the base table.
*   **Limit:** 10 GB size limit per partition key value (Item Collection Limit).

#### Global Secondary Index (GSI)
*   **Scope:** Spans the entire table (can have a different PK and SK).
*   **Flexibility:** Can be created or deleted at any time.
*   **Consistency:** Supports only **Eventually Consistent** reads.
*   **Capacity:** Has its own dedicated RCU/WCU (independent of the base table).
*   **Limit:** No size limit per partition key.

### 3. Key Indexing Concepts
*   **Projections:** When creating an index, you choose which attributes to copy from the base table:
    *   `KEYS_ONLY`: Only the index and primary keys.
    *   `INCLUDE`: Keys plus specific named attributes.
    *   `ALL`: The entire item.
*   **Sparse Indexes:** If an item in the base table does not contain the attribute(s) used as index keys, that item is not included in the index. This is useful for creating "filtered" views (e.g., an index of only "Open" orders).
*   **Overloading:** In "Single Table Design," GSIs are often overloaded by using generic attribute names (e.g., `GSI1PK`, `GSI1SK`) to support multiple entity types in the same index.

### 4. Go Implementation: Querying a GSI
Using the AWS SDK for Go v2, here is how you query a table using a Global Secondary Index.

```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/aws/aws-sdk-go-v2/aws"
	"github.com/aws/aws-sdk-go-v2/config"
	"github.com/aws/aws-sdk-go-v2/feature/dynamodb/attributevalue"
	"github.com/aws/aws-sdk-go-v2/service/dynamodb"
	"github.com/aws/aws-sdk-go-v2/service/dynamodb/types"
)

type Movie struct {
	ID    string `dynamodbav:"ID"`
	Title string `dynamodbav:"Title"`
	Year  int    `dynamodbav:"Year"`
	Genre string `dynamodbav:"Genre"`
}

func main() {
	ctx := context.TODO()
	cfg, err := config.LoadDefaultConfig(ctx, config.WithRegion("us-east-1"))
	if err != nil {
		log.Fatalf("unable to load SDK config, %v", err)
	}

	client := dynamodb.NewFromConfig(cfg)

	tableName := "Movies"
	indexName := "GenreYearIndex" // GSI: PK=Genre, SK=Year

	// Querying by Genre = "Sci-Fi" and Year > 2020
	input := &dynamodb.QueryInput{
		TableName:              aws.String(tableName),
		IndexName:              aws.String(indexName),
		KeyConditionExpression: aws.String("Genre = :g AND #y > :year"),
		ExpressionAttributeNames: map[string]string{
			"#y": "Year", // "Year" is a reserved keyword in DynamoDB
		},
		ExpressionAttributeValues: map[string]types.AttributeValue{
			":g":    &types.AttributeValueMemberS{Value: "Sci-Fi"},
			":year": &types.AttributeValueMemberN{Value: "2020"},
		},
	}

	result, err := client.Query(ctx, input)
	if err != nil {
		log.Fatalf("failed to query index, %v", err)
	}

	var movies []Movie
	err = attributevalue.UnmarshalListOfMaps(result.Items, &movies)
	if err != nil {
		log.Fatalf("failed to unmarshal results, %v", err)
	}

	for _, movie := range movies {
		fmt.Printf("Found: %s (%d)\n", movie.Title, movie.Year)
	}
}
```

## Interview Questions

**Q: What is the main difference between LSI and GSI regarding consistency?**
**A:** LSIs support both strong and eventual consistency because they reside in the same partition as the base table data. GSIs only support eventual consistency because data is replicated asynchronously to a separate partition space.

**Q: Why would you choose a "Sparse Index"?**
**A:** To optimize costs and performance. Since DynamoDB only populates an index if the key attributes exist in the item, you can create an index on an optional attribute to effectively "filter" the table (e.g., an index for `is_admin` to quickly find admins without scanning the whole user table).

**Q: What happens if a GSI's provisioned write capacity is too low compared to the base table?**
**A:** This can lead to "backpressure." If the GSI cannot keep up with writes from the base table, DynamoDB will throttle writes on the **base table**, even if the base table itself has enough WCU. This is a critical design consideration.

**Q: Can you add an LSI to an existing DynamoDB table?**
**A:** No. LSIs must be defined at table creation time. If you need a new local index later, you must create a new table and migrate the data. GSIs, however, can be added or removed at any time.

**Q: What is the "Item Collection Limit" for LSIs?**
**A:** For tables with one or more LSIs, there is a 10 GB limit on the total size of all items sharing the same partition key. This limit does not apply if the table does not have LSIs or for GSIs.
