---
---

# Retry Storm (Thundering Herd)

## Summary
When a downstream service fails or slows down, upstream clients aggressively retry their requests simultaneously. Without proper backoff and jitter, these retries act as a Distributed Denial of Service (DDoS) attack on the already struggling service, preventing its recovery.

## Detailed Development
Retries are essential for transient failures, but dangerous if not managed:
- **Exponential Backoff**: Increasing the wait time between retries (e.g., 1s, 2s, 4s, 8s).
- **Jitter**: Adding randomness to the backoff time to prevent all clients from retrying at exactly the same microsecond.
- **Circuit Breaker**: Stopping all requests to a failing service for a period to allow it to recover.

## Go-Specific Application

### 1. Exponential Backoff with Jitter
While you can use libraries like `github.com/cenkalti/backoff`, a simple manual implementation in Go looks like this:

```go
func CallServiceWithRetry(ctx context.Context) error {
    baseDelay := 100 * time.Millisecond
    maxRetries := 5

    for i := 0; i < maxRetries; i++ {
        err := doCall()
        if err == nil {
            return nil
        }

        // Calculate backoff: baseDelay * 2^i
        backoff := baseDelay * time.Duration(math.Pow(2, float64(i)))
        
        // Add Jitter: +/- 10% randomness
        jitter := time.Duration(rand.Int63n(int64(backoff / 5)))
        sleepTime := backoff + jitter

        select {
        case <-time.After(sleepTime):
            continue
        case <-ctx.Done():
            return ctx.Err()
        }
    }
    return fmt.Errorf("failed after %d retries", maxRetries)
}
```

### 2. The `cenkalti/backoff` Library
The community standard for backoff in Go:

```go
operation := func() error {
    return doCall()
}

err := backoff.Retry(operation, backoff.NewExponentialBackOff())
```

### 3. Circuit Breakers
Use libraries like `github.com/sony/gobreaker` to prevent the "storm" from starting in the first place.

```go
var cb *gobreaker.CircuitBreaker

func init() {
    cb = gobreaker.NewCircuitBreaker(gobreaker.Settings{
        Name:        "my-service",
        MaxRequests: 3,
        Interval:    5 * time.Second,
        Timeout:     30 * time.Second,
    })
}

func WrappedCall() {
    result, err := cb.Execute(func() (interface{}, error) {
        return doCall()
    })
}
```

## Interview Preparation

### Questions
1. **Why is "Jitter" important?**
   - Without jitter, if a load balancer goes down, 10,000 clients will wait exactly 1 second and then hit the server at the exact same time, causing a synchronized spike that crashes the server again.

2. **What is a "Circuit Breaker"?**
   - A design pattern used to detect failures and encapsulate the logic of preventing a failure from constantly recurring during maintenance or temporary external system failure. It has three states: **Closed** (allowing requests), **Open** (rejecting requests), and **Half-Open** (testing if service is back).

3. **What is the difference between a Retry Storm and a Thundering Herd?**
   - **Retry Storm**: Clients retrying failed requests.
   - **Thundering Herd**: Many processes waiting for an event (e.g., a cache key expiration) and all trying to compute/fetch the same data at once when it happens.
