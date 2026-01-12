---
---

## Summary
**Background Jobs** allow you to offload time-consuming tasks from the critical request-response path (e.g., HTTP request) to a background worker. This significantly improves the responsiveness and user experience of the application.

## Detailed Explanation

### Concept
When a user clicks "Generate Report," instead of making them wait 30 seconds:
1.  **Server**: Creates a "Job" record (`status: pending`) and returns `202 Accepted` immediately.
2.  **Worker**: Picks up the job in the background and processes it.
3.  **UI**: Polls for status or waits for a WebSocket/Email notification.

### Benefits
*   **Responsiveness**: User interface remains snappy.
*   **Scalability**: You can scale web servers (for traffic) and worker servers (for heavy lifting) independently.
*   **Reliability**: Failed background jobs can be retried automatically without the user seeing a 500 error.

## Go Context: Goroutines vs Queues
*   **Simple**: Fire a goroutine `go generateReport()`. *Risk*: If the app crashes, the job is lost.
*   **Robust**: Enqueue to Redis/Postgres (`asynq`/`river`). *Benefit*: Persistence and Retries.

```go
// Handler
func GenerateReport(w http.ResponseWriter, r *http.Request) {
    // 1. Enqueue Job
    jobID, _ := taskQueue.Enqueue("generate_pdf", payload)
    
    // 2. Return immediately
    w.WriteHeader(http.StatusAccepted)
    json.NewEncoder(w).Encode(map[string]string{"job_id": jobID})
}
```

## Interview Questions

### Q: Why not just use a Goroutine for background tasks?
**A:** Goroutines are tied to the application process's lifecycle. If the server restarts (deployment) or crashes (OOM), all running goroutines are killed instantly, leading to data loss or half-finished tasks. For critical tasks (e.g., payments), you need a persistent queue.

### Q: How do you handle failed background jobs?
**A:** Implement a **Dead Letter Queue (DLQ)**. If a job fails after N retries (with exponential backoff), it is moved to a DLQ so developers can inspect and debug it manually without clogging the main queue.
