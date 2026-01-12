---
tags: ['aws', 'roadmap', 'rds']
---

# RDS Backup and Restore (Automated vs Snapshots)

## Summary
Amazon RDS provides two primary methods for backing up and restoring database instances: **Automated Backups** and **Manual Snapshots**. Automated backups enable Point-in-Time Recovery (PITR) by combining daily snapshots with transaction logs, while Manual Snapshots are user-initiated backups that persist even after the database instance is deleted. Both methods use incremental storage to minimize backup time and storage costs.

## Detailed Explanation

### 1. Automated Backups
Automated backups are enabled by default when you create a DB instance.
- **Retention Period**: You can set the retention period from **0 to 35 days**. Setting it to 0 disables automated backups.
- **Backup Window**: RDS performs a full daily backup of your data during a user-defined 30-minute window.
- **Transaction Logs**: RDS uploads transaction logs (e.g., MySQL binary logs, PostgreSQL WAL) to S3 every 5 minutes.
- **Point-in-Time Recovery (PITR)**: Allows you to restore your database to any second within the retention period. A PITR operation always creates a **new DB instance**.
- **Instance Deletion**: By default, automated backups are deleted when the DB instance is deleted, though you can choose to retain them.

### 2. Manual DB Snapshots
Manual snapshots are initiated by the user and offer more flexibility for long-term storage or migration.
- **Persistence**: They are **never deleted automatically**. They persist even after the source DB instance is deleted.
- **Sharing**: Manual snapshots can be shared with other AWS accounts or made public (not recommended for sensitive data).
- **Copying**: They can be copied across AWS Regions for disaster recovery or migration purposes.
- **Limit**: You can have up to 100 manual snapshots per region by default.

### 3. Comparison Table

| Feature | Automated Backups | Manual Snapshots |
| :--- | :--- | :--- |
| **Trigger** | Scheduled (System) | On-demand (User) |
| **Retention** | 1 - 35 days (0 to disable) | Indefinite (until manual deletion) |
| **PITR Support** | Yes (Daily snapshot + Logs) | No (Snapshot only) |
| **Deletion** | Usually deleted with instance | Persists after instance deletion |
| **Sharing** | Not directly sharable | Sharable across accounts |

### 4. Performance Impact
During the backup window, storage I/O might be suspended briefly while the snapshot is initialized.
- **Multi-AZ**: Backups are taken from the standby instance to avoid performance impact on the primary.
- **Amazon Aurora**: Backups are continuous and incremental, with no impact on database performance.

### Go Implementation Example
Using the AWS SDK for Go v2 to create a manual snapshot.

```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/aws/aws-sdk-go-v2/config"
	"github.com/aws/aws-sdk-go-v2/service/rds"
)

func main() {
	// Load AWS configuration
	cfg, err := config.LoadDefaultConfig(context.TODO(), config.WithRegion("us-east-1"))
	if err != nil {
		log.Fatalf("unable to load SDK config, %v", err)
	}

	// Create RDS client
	client := rds.NewFromConfig(cfg)

	// Define snapshot parameters
	input := &rds.CreateDBSnapshotInput{
		DBInstanceIdentifier: fmt.Sprintf("my-database-instance"),
		DBSnapshotIdentifier: fmt.Sprintf("manual-snapshot-%d", 123456789),
	}

	// Trigger snapshot creation
	output, err := client.CreateDBSnapshot(context.TODO(), input)
	if err != nil {
		log.Fatalf("failed to create snapshot: %v", err)
	}

	fmt.Printf("Snapshot initiated: %s\n", *output.DBSnapshot.DBSnapshotIdentifier)
}
```

## Interview Questions

**Q: What is the maximum retention period for RDS automated backups?**
**A:** The maximum retention period is 35 days. If you need to keep backups for longer, you must create manual snapshots or use AWS Backup.

**Q: How does Point-in-Time Recovery (PITR) work in RDS?**
**A:** PITR uses a combination of the most recent daily backup (snapshot) and the transaction logs (like binlogs or WAL) uploaded to S3. When you restore, RDS applies the transaction logs to the snapshot to reach the exact second requested, creating a new DB instance.

**Q: Can you share an automated backup with another AWS account?**
**A:** No, you cannot share automated backups directly. To share the data, you must first create a manual snapshot from the automated backup (or copy it as a manual snapshot) and then share that manual snapshot.

**Q: Does taking a snapshot of an RDS instance encrypt the backup if the DB is unencrypted?**
**A:** No. If the DB instance is not encrypted, the snapshot will also be unencrypted. However, you can encrypt an unencrypted snapshot during a **Copy Snapshot** operation.

**Q: What is the "Final Snapshot" option when deleting an RDS instance?**
**A:** It is a manual snapshot created immediately before the instance is deleted. It ensures you have a consistent backup of the data at the moment of deletion, which is critical since automated backups are often deleted by default.
