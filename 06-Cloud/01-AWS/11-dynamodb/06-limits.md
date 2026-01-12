#AWS #Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'dynamodb']
---

## Summary
DynamoDB is a highly scalable NoSQL database, but its performance and design are governed by strict limits regarding item size, partition throughput, and storage. Understanding these constraints is critical for efficient schema design and avoiding "Hot Partition" issues or "Item Size Too Large" errors.

## Detailed Explanation

### 1. Item Size Limit
*   **Max Item Size**: **400 KB**.
*   **What's included**: The size includes both the attribute names (length of the strings) and the attribute values.
*   **Implications**: If your data model requires storing large objects (like images or large JSON blobs), you should store the metadata in DynamoDB and the actual object in **Amazon S3**, saving the S3 URI in the DynamoDB item.

### 2. Partition Limits
DynamoDB manages data in partitions. Each partition has hard limits on throughput and size:
*   **Throughput Limits**:
    *   **3,000 RCU** (Read Capacity Units) per partition.
    *   **1,000 WCU** (Write Capacity Units) per partition.
*   **Storage Limit**: **10 GB** per partition.
*   **Partitioning Logic**: DynamoDB uses the **Partition Key** to distribute items across partitions. If a single Partition Key receives more than 3,000 RCU or 1,000 WCU, it becomes a **Hot Partition**, leading to throttling even if the table's total provisioned throughput is much higher.

### 3. Query and Scan Limits
*   **Result Set Limit**: A single `Query` or `Scan` operation can return a maximum of **1 MB** of data.
*   **Pagination**: If the result set exceeds 1 MB, DynamoDB returns a `LastEvaluatedKey`, which must be used in the next request to retrieve the remaining data.

### 4. Secondary Index Limits
*   **Local Secondary Index (LSI)**:
    *   Max **5** LSIs per table.
    *   **Item Collection Limit**: The total size of all items with the same partition key (main table + all LSIs) cannot exceed **10 GB**.
*   **Global Secondary Index (GSI)**:
    *   Default **20** GSIs per table (can be increased via service quota request).
    *   GSIs do not have the 10 GB item collection limit.

### 5. Batch and Transaction Limits
*   **BatchWriteItem**: Up to **25 items** or **16 MB** per request.
*   **BatchGetItem**: Up to **100 items** or **16 MB** per request.
*   **TransactWriteItems / TransactGetItems**: Up to **100 items** (atomicity across all items).

## Interview Questions

**Q1: What is the maximum size of a single item in DynamoDB, and what happens if you exceed it?**
**A:** The maximum size is 400 KB (including attribute names and values). If you attempt to write an item larger than this, DynamoDB returns a `ValidationException`. To handle larger data, use S3 for the payload and store the S3 link in DynamoDB.

**Q2: How much throughput can a single DynamoDB partition handle?**
**A:** A single partition can handle a maximum of 3,000 Read Capacity Units (RCU) and 1,000 Write Capacity Units (WCU). Exceeding these limits on a single partition key results in throttling.

**Q3: What is the "Item Collection Limit" associated with LSIs?**
**A:** For tables with one or more Local Secondary Indexes (LSIs), there is a 10 GB limit on the total size of all items that share the same Partition Key. This is called an Item Collection. If this limit is reached, you cannot add more items to that partition key.

**Q4: How does DynamoDB handle Query or Scan results that exceed 1 MB?**
**A:** DynamoDB truncates the result at 1 MB and returns a `LastEvaluatedKey`. The client must then perform another request using this key as the `ExclusiveStartKey` to fetch the next "page" of results.

**Q5: What is the maximum number of items allowed in a single DynamoDB transaction?**
**A:** As of current limits, you can include up to 100 items in a single `TransactWriteItems` or `TransactGetItems` request.
