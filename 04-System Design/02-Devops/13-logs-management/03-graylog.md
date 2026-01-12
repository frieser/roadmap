---
---

# Graylog

Graylog is an open-source log management tool designed specifically for logs (unlike Splunk/ELK which are broader analytics platforms). It is known for its speed, simplicity, and ease of setup.

## Summary

Graylog uses **Elasticsearch** for storage and **MongoDB** for metadata configuration. Its key innovation is **GELF (Graylog Extended Log Format)**, a structured JSON format that eliminates the pain of parsing syslog messages. It excels at fast searching and alerting without the steep learning curve of ELK.

## Detailed Explanation

### 1. Architecture
*   **Graylog Server**: The core application (Java). Receives logs, processes streams, handles alerts.
*   **Elasticsearch**: Used purely as the storage engine for log data.
*   **MongoDB**: Stores configuration (Users, Dashboards, Streams, Alerts).

### 2. GELF (Graylog Extended Log Format)
Standard Syslog is limited (only 1KB size, unstructured text). GELF supports:
*   Structured JSON fields (`{"user_id": 123}`).
*   Compression (GZIP/ZLIB).
*   Chunking (Large messages split over UDP).

### 3. Streams and Pipelines
*   **Streams**: Categories of logs (e.g., "Production Errors"). You route logs into streams based on rules.
*   **Pipelines**: Processing rules (written in a Java-like DSL) to modify/enrich logs before storage.

---

## Go Implementation Example

Using a GELF library (`github.com/Graylog2/go-gelf/gelf`) to send structured logs over UDP.

```go
package main

import (
	"log"
	"time"

	"github.com/Graylog2/go-gelf/gelf"
)

func main() {
	// 1. Connect to Graylog UDP Input
	writer, err := gelf.NewWriter("graylog-server:12201")
	if err != nil {
		log.Fatalf("Failed to connect to Graylog: %v", err)
	}
	// Note: UDP is connectionless, so this error usually means DNS failure
	defer writer.Close()

	// 2. Send structured message
	// WriteMessage sends a structured GELF message
	err = writer.WriteMessage(&gelf.Message{
		Version:  "1.1",
		Host:     "api-service-01",
		Short:    "Database timeout exception",
		Full:     "Stack trace: ... connection failed at db.go:42",
		TimeUnix: float64(time.Now().Unix()),
		Level:    3, // Syslog Error Level
		Extra: map[string]interface{}{
			"environment": "production",
			"request_id":  "req-abc-123",
			"user_id":     42,
		},
	})

	if err != nil {
		log.Printf("Failed to send log: %v", err)
	} else {
		log.Println("Log sent to Graylog")
	}
}
```

## Interview Questions

**Q: Why use GELF over Syslog?**
**A:** Syslog has a 1024-byte limit (traditionally) and sends unstructured text strings, forcing the server to use expensive regex to parse data. GELF is structured JSON (no parsing needed), supports compression to save bandwidth, and handles large payloads via chunking.

**Q: How does Graylog differ from ELK?**
**A:**
*   **ELK**: A generic search platform. You build the logging solution on top of it (Kibana requires manual dashboard creation).
*   **Graylog**: An "out-of-the-box" logging solution. It has built-in user management, alerting, and stream routing specifically designed for logs. It is generally easier to set up but less flexible for non-log data (like metrics).

**Q: What is a Graylog "Sidecar"?**
**A:** A Sidecar is a lightweight process that manages the configuration of log collectors (like Filebeat or NXLog) on client machines. Instead of manually editing `filebeat.yml` on 100 servers, you define the config in the Graylog UI, and the Sidecar pulls and applies it to the collectors locally.
