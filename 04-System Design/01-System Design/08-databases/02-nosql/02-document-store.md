---
---

## Summary
Document databases store data in documents, typically using formats like JSON, BSON, or XML. Unlike relational databases, documents are self-describing and can have different structures (flexible schema). This model is highly intuitive for developers as it maps directly to objects in application code.

## Detailed Explanation

### Core Concepts
*   **Encapsulation**: All data related to an entity is often stored in a single document (denormalization).
*   **Flexible Schema**: You can add new fields to a document without affecting other documents in the same collection.
*   **Indexing**: Powerful indexing capabilities on any field, including nested attributes.

### Use Cases
1.  **Content Management Systems (CMS)**: Storing blog posts, pages, and media metadata with varying attributes.
2.  **Product Catalogs**: E-commerce catalogs where different products (e.g., a shirt vs. a laptop) have entirely different attributes.
3.  **User Profiles**: Storing user data that evolves over time (e.g., adding social media handles or preferences).
4.  **Real-time Analytics**: Storing log data or events that are semi-structured.

### Notable Examples
*   **MongoDB**: The most popular document database, using BSON (Binary JSON).
*   **CouchDB**: Uses JSON and provides a web-based interface (Futon).
*   **RavenDB**: A document database for the .NET platform.

### Go Application
Interacting with MongoDB in Go using the official driver.

```go
package main

import (
	"context"
	"fmt"
	"go.mongodb.org/mongo-driver/bson"
	"go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"
)

type Article struct {
	Title   string   `bson:"title"`
	Tags    []string `bson:"tags"`
	Content string   `bson:"content"`
}

func main() {
	client, _ := mongo.Connect(context.TODO(), options.Client().ApplyURI("mongodb://localhost:27017"))
	collection := client.Database("blog").Collection("articles")

	// Insert a document with nested data
	newArticle := Article{
		Title:   "NoSQL vs SQL",
		Tags:    []string{"database", "architecture"},
		Content: "A deep dive into database types...",
	}
	collection.InsertOne(context.TODO(), newArticle)

	// Query using a filter
	var result Article
	filter := bson.M{"tags": "architecture"}
	collection.FindOne(context.TODO(), filter).Decode(&result)
	fmt.Println("Found article:", result.Title)
}
```

## Interview Questions

**Q: What is the benefit of "denormalization" in Document DBs?**
**A:** Denormalization allows you to store related data together in a single document. This eliminates the need for expensive JOIN operations at query time, significantly improving read performance for specific use cases.

**Q: When is a Document DB a POOR choice?**
**A:** When your data is highly relational and requires complex transactions across multiple entities, or when you need to perform frequent, deep analytical queries (joins) across the entire dataset.

**Q: How does MongoDB handle schema evolution?**
**A:** Since it is schemaless, the application code is responsible for handling different versions of documents. Common strategies include providing default values for missing fields or running background migration scripts to update old documents.
