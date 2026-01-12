---
---

# Splunk

Splunk is the market leader in enterprise log management and SIEM (Security Information and Event Management). It is famous for its powerful **Search Processing Language (SPL)** and its ability to ingest massive volumes of unstructured data.

## Summary

Splunk is a proprietary platform designed to "Google" your logs. It uses a distributed architecture of **Forwarders** (agents), **Indexers** (storage), and **Search Heads** (UI/Query). While expensive, it offers unparalleled depth for security analysis and operational intelligence.

## Detailed Explanation

### 1. Core Architecture
*   **Universal Forwarder (UF)**: A lightweight agent installed on servers to collect logs and ship them to indexers.
*   **Indexer**: Parses incoming data, extracts timestamps, and stores it in "buckets" (Hot/Warm/Cold) on disk.
*   **Search Head**: Distributes user queries to Indexers and aggregates the results.
*   **HEC (HTTP Event Collector)**: A token-based HTTP API for high-throughput, agentless data ingestion. Ideally suited for serverless functions and custom apps.

### 2. SPL (Search Processing Language)
A powerful query language inspired by Unix pipes (`|`).
Example: `index=web_logs status=500 | stats count by host | sort -count`

---

## Go Implementation Example

Using **HEC (HTTP Event Collector)** is the standard way to send logs from Go applications directly to Splunk without installing a file-based forwarder.

```go
package main

import (
	"bytes"
	"crypto/tls"
	"encoding/json"
	"fmt"
	"net/http"
	"time"
)

// HecEvent represents the JSON payload Splunk expects
type HecEvent struct {
	Event      interface{} `json:"event"`
	SourceType string      `json:"sourcetype,omitempty"`
	Host       string      `json:"host,omitempty"`
	Time       int64       `json:"time,omitempty"`
}

func main() {
	// Splunk Configuration
	splunkURL := "https://splunk-indexer:8088/services/collector"
	token := "YOUR-SPLUNK-HEC-TOKEN"

	// Create Payload
	payload := HecEvent{
		Event:      map[string]string{"message": "Login failed", "user": "admin", "level": "ERROR"},
		SourceType: "go:app",
		Host:       "web-server-01",
		Time:       time.Now().Unix(),
	}

	data, _ := json.Marshal(payload)

	// Send Request
	req, _ := http.NewRequest("POST", splunkURL, bytes.NewBuffer(data))
	req.Header.Set("Authorization", "Splunk "+token)
	req.Header.Set("Content-Type", "application/json")

	// Skip SSL verification for self-signed certs (Common in internal setups)
	client := &http.Client{
		Transport: &http.Transport{
			TLSClientConfig: &tls.Config{InsecureSkipVerify: true},
		},
	}

	resp, err := client.Do(req)
	if err != nil {
		fmt.Printf("Error sending logs: %v\n", err)
		return
	}
	defer resp.Body.Close()

	fmt.Printf("Splunk Response: %s\n", resp.Status)
}
```

## Interview Questions

**Q: What is the difference between Index-time and Search-time field extraction?**
**A:**
*   **Index-time**: Fields are extracted when data hits the Indexer. This increases storage usage and is immutable (cannot change extraction logic for old data). Faster search.
*   **Search-time**: Fields are extracted when the user runs a search. Flexible (you can change regex anytime) but slower query performance. Splunk recommends Search-time extraction by default.

**Q: Explain the lifecycle of a Splunk Bucket.**
**A:** Data moves through buckets based on age:
*   **Hot**: Currently being written to. Searchable. High IO.
*   **Warm**: Read-only. Searchable.
*   **Cold**: Older data moved to cheaper storage. Slower search.
*   **Frozen**: Archived (often deleted or moved to AWS Glacier). Not searchable unless "thawed".

**Q: How does Splunk handle High Availability?**
**A:** Splunk uses **Indexer Clustering**. Data sent to one indexer is replicated to others (Replication Factor). If an indexer fails, the cluster master directs searches to the replicas.
