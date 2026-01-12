#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'dynamodb']
---

## Summary
DynamoDB offers two robust backup solutions: **On-Demand Backups** and **Point-in-Time Recovery (PITR)**. On-Demand backups provide full table snapshots for long-term retention and compliance, while PITR offers continuous data protection for the last 35 days, allowing restoration to any specific second. Both methods are designed to have zero impact on table performance.

## Detailed Explanation

### 1. On-Demand Backups
On-Demand backups allow you to create full snapshots of your tables for archival and long-term retention.
- **Full Snapshots**: Each backup is a complete copy of the table data as it existed at the time of the request.
- **No Performance Impact**: Backing up a table does not consume its provisioned throughput (RCU/WCU) or affect its availability.
- **Retention**: These backups persist until you manually delete them. They are ideal for regulatory compliance and periodic data archiving.
- **Integration**: While they can be triggered manually, they are often managed at scale using **AWS Backup**, which allows for cross-account and cross-region backup policies.

### 2. Point-in-Time Recovery (PITR)
PITR provides continuous backup for your DynamoDB table data.
- **Continuous Backup**: Once enabled, DynamoDB maintains incremental backups of your data for the last 35 days.
- **Granular Recovery**: You can restore your table to any point in time (to the second) within the 35-day window (`EarliestRestorableDateTime` to `LatestRestorableDateTime`).
- **Operational Safety**: PITR is the primary defense against accidental delete or update operations caused by application bugs or human error.
- **State**: PITR is disabled by default and must be enabled per table.

### 3. Restoration Considerations
Restoring a table (from either On-Demand or PITR) always results in a **new table**. 
- **Metadata**: The restored table includes the base data and all Global Secondary Indexes (GSIs).
- **Manual Setup Required**: Several settings are **not** restored and must be reconfigured on the new table:
    - IAM policies
    - CloudWatch alarms
    - Auto Scaling policies
    - Time to Live (TTL) settings
    - DynamoDB Streams settings
    - Tags

### 4. Go Implementation Example
The following example demonstrates how to enable PITR and create an on-demand backup using the AWS SDK for Go v2.

```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/aws/aws-sdk-go-v2/config"
	"github.com/aws/aws-sdk-go-v2/service/dynamodb"
	"github.com/aws/aws-sdk-go-v2/service/dynamodb/types"
)

func main() {
	ctx := context.TODO()
	cfg, err := config.LoadDefaultConfig(ctx, config.WithRegion("us-east-1"))
	if err != nil {
		log.Fatalf("unable to load SDK config, %v", err)
	}

	client := dynamodb.NewFromConfig(cfg)
	tableName := "MyUserTable"

	// 1. Enable Point-in-Time Recovery (PITR)
	_, err = client.UpdateContinuousBackups(ctx, &dynamodb.UpdateContinuousBackupsInput{
		TableName: &tableName,
		PointInTimeRecoverySpecification: &types.PointInTimeRecoverySpecification{
			PointInTimeRecoveryEnabled: true,
		},
	})
	if err != nil {
		log.Printf("Failed to enable PITR: %v", err)
	} else {
		fmt.Println("PITR enabled successfully")
	}

	// 2. Create an On-Demand Backup
	backupName := "ManualBackup-2026-01-10"
	resp, err := client.CreateBackup(ctx, &dynamodb.CreateBackupInput{
		TableName:  &tableName,
		BackupName: &backupName,
	})
	if err != nil {
		log.Printf("Failed to create backup: %v", err)
	} else {
		fmt.Printf("Backup created: %s\n", *resp.BackupDetails.BackupArn)
	}
}
```

## Interview Questions

**Q: Does enabling PITR or taking an On-Demand backup impact my table's RCU/WCU?**
**A:** No. DynamoDB backup and restore operations have zero impact on the performance or availability of your production table. They do not consume any of your provisioned or on-demand capacity.

**Q: If I accidentally delete a table, can I use PITR to recover it?**
**A:** No. PITR only works as long as the table exists. If a table is deleted, all its PITR data is also deleted. To protect against table deletion, you should use **On-Demand backups** (which persist after table deletion) or **AWS Backup** with a vault lock.

**Q: Can I restore a backup to an existing table?**
**A:** No. A restore operation always creates a new table. You must then point your application to the new table's endpoint or rename the tables (which involves deleting the old one first).

**Q: How long does it take to restore a DynamoDB table?**
**A:** Restoration time varies based on the size of the table and the number of secondary indexes. While it's generally fast, for multi-terabyte tables, it can take several hours as DynamoDB must provision the new table and populate it with data and indexes.
