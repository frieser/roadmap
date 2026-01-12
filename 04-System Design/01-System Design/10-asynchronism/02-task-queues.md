---
---

## Summary
**Task Queues** manage background work by distributing tasks across multiple worker nodes. Unlike generic message queues, task queues are specialized for job execution, providing features like scheduling, retries, prioritization, and result storage.

## Detailed Explanation

### Architecture
1.  **Producer**: Adds a job (e.g., "Send Email") to the queue.
2.  **Broker**: Stores the job (often Redis).
3.  **Worker**: Pulls the job, executes it, and reports status (Success/Failure).

### Redis-based Queues
Tools like **Celery** (Python), **Bull** (Node.js), and **Asynq** (Go) use Redis for speed.
*   **Pros**: Extremely fast, easy to set up.
*   **Cons**: If Redis crashes without persistence (AOF/RDB), tasks are lost.

### PostgreSQL-based Queues
Tools like **River** (Go) use the database itself.
*   **Pros**: **Transactional Integrity**. You can enqueue a job in the same transaction as your business logic (e.g., "Create User" + "Enqueue Welcome Email"). If one fails, both rollback.
*   **Cons**: Increases load on the primary DB.

## Go Context: Using `asynq`

```go
// Client: Enqueue
client := asynq.NewClient(asynq.RedisClientOpt{Addr: "localhost:6379"})
task := asynq.NewTask("email:send", []byte("user_id=123"))
info, _ := client.Enqueue(task)

// Server: Process
srv := asynq.NewServer(redisOpt, asynq.Config{Concurrency: 10})
srv.Run(asynq.HandlerFunc(func(ctx context.Context, t *asynq.Task) error {
    // Process email...
    return nil
}))
```

## Interview Questions

### Q: Why use a dedicated Task Queue instead of a simple Go Channel?
**A:** Go Channels are **in-memory** and **process-bound**. If the application crashes or restarts, all queued tasks in the channel are lost. A distributed Task Queue (like Redis) persists tasks outside the app process, allowing them to survive restarts and be shared across multiple worker nodes.

### Q: What is "At-Least-Once" delivery?
**A:** It guarantees that a task will be delivered to a worker, but it might be delivered more than once (e.g., if a worker crashes after processing but before acking). Therefore, task handlers must be **Idempotent**.
