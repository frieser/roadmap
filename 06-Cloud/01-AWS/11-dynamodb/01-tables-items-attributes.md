#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'dynamodb']
---

## Summary
Amazon DynamoDB is a fully managed NoSQL database service that stores data in **Tables**, which are collections of **Items**, each composed of multiple **Attributes**. It is designed for high-scale applications, offering a schemaless structure where only the primary key must be defined upfront, providing flexibility for evolving data models.

## Detailed Explanation

### Core Components
DynamoDB's data model is hierarchical and consists of three fundamental units:

1.  **Tables**: The top-level containers for data. Every table must have a unique name and a defined primary key.
2.  **Items**: Similar to rows or records in a relational database. Each item is a group of attributes that is uniquely identifiable via the primary key. There is no limit to the number of items in a table, but each individual item has a **size limit of 400 KB**.
3.  **Attributes**: The smallest unit of data, similar to fields or columns. Attributes can be simple scalars or complex nested structures.

### Schema-less Nature
Unlike traditional RDBMS, DynamoDB is **schemaless**. While you must define the table name and primary key attributes (and their types) at creation time, you do not define any other attributes. This allows:
*   Items within the same table to have different attributes.
*   The data model to evolve without expensive `ALTER TABLE` operations.
*   Nested data structures (Maps and Lists) up to 32 levels deep.

### Data Types
DynamoDB supports several categories of data types:
*   **Scalar Types**: String, Number, Binary, Boolean, and Null.
*   **Document Types**: 
    *   **List**: An ordered collection of values (like a JSON array).
    *   **Map**: An unordered collection of name-value pairs (like a JSON object).
*   **Set Types**: String Set, Number Set, and Binary Set (unique values of the same type).

### Primary Keys
The primary key uniquely identifies each item and determines its physical placement in the database.
*   **Partition Key (Hash Key)**: A simple primary key. DynamoDB uses the value to distribute items across physical partitions.
*   **Composite Primary Key (Partition Key + Sort Key)**: Also known as Hash + Range. It allows multiple items to share the same partition key but requires a unique sort key for each.

### Go Implementation (PutItem)
Using the AWS SDK for Go (v2), here is how you create or replace an item in a DynamoDB table.

```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/aws/aws-sdk-go-v2/aws"
	"github.com/aws/aws-sdk-go-v2/config"
	"github.com/aws/aws-sdk-go-v2/service/dynamodb"
	"github.com/aws/aws-sdk-go-v2/service/dynamodb/types"
)

func main() {
	// Load the SDK configuration
	cfg, err := config.LoadDefaultConfig(context.TODO(), config.WithRegion("us-east-1"))
	if err != nil {
		log.Fatalf("unable to load SDK config, %v", err)
	}

	// Create a DynamoDB client
	client := dynamodb.NewFromConfig(cfg)

	// Define the item attributes
	item := map[string]types.AttributeValue{
		"PersonID":  &types.AttributeValueMemberN{Value: "101"},
		"FirstName": &types.AttributeValueMemberS{Value: "John"},
		"LastName":  &types.AttributeValueMemberS{Value: "Doe"},
		"Email":     &types.AttributeValueMemberS{Value: "john.doe@example.com"},
		"Tags": &types.AttributeValueMemberSS{
			Value: []string{"Developer", "Go", "AWS"},
		},
	}

	// Execute PutItem
	_, err = client.PutItem(context.TODO(), &dynamodb.PutItemInput{
		TableName: aws.String("People"),
		Item:      item,
	})

	if err != nil {
		log.Fatalf("failed to put item, %v", err)
	}

	fmt.Println("Successfully added item to People table")
}
```

## Interview Questions

**Q: What is the main difference between a Partition Key and a Sort Key?**
**A:** The Partition Key is used by DynamoDB's internal hash function to distribute data across physical partitions. The Sort Key is used to store items with the same Partition Key physically close together, sorted by the Sort Key value, enabling efficient range queries.

**Q: What is the maximum size limit for a single DynamoDB item, and what happens if you exceed it?**
**A:** The limit is 400 KB, which includes both attribute names and values. If you attempt to save an item exceeding this, the request will fail with a `ValidationException`. To handle larger data, you should store the payload in S3 and save the S3 URL in DynamoDB.

**Q: Is DynamoDB truly "schema-free"?**
**A:** Not entirely. You must define the Primary Key (Partition Key and optionally a Sort Key) and its data type when creating the table. This schema is rigid and cannot be changed for the life of the table. However, all other "non-key" attributes are schema-free.

**Q: What are "Document" data types in DynamoDB?**
**A:** Document types are Maps and Lists. They allow for nested attributes, enabling you to store complex, hierarchical data (like JSON objects) directly within an item, supporting up to 32 levels of nesting.

**Q: Why would you use a Sort Key in a DynamoDB table?**
**A:** A Sort Key allows you to perform complex queries on a specific partition. For example, in a "Orders" table with `CustomerID` as the Partition Key and `OrderDate` as the Sort Key, you can retrieve all orders for a customer sorted by date or within a specific date range.
