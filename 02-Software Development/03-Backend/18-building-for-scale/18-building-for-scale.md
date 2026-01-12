# 18. Building for Scale

## Summary
Building for scale involves designing systems that can handle growth in users, data, and complexity without sacrificing performance or reliability. This involves choosing the right **Scaling Strategy**, implementing **Mitigation Patterns** to handle failures gracefully, and planning **Migration Strategies** for updates.

## Detailed Explanation

### 1. Types of Scaling
- **Vertical Scaling (Scaling Up)**: Adding more power (CPU, RAM) to an existing server. Limited by hardware caps and single point of failure.
- **Horizontal Scaling (Scaling Out)**: Adding more machines to the pool. Preferred for modern distributed systems as it provides redundancy.

### 2. Mitigation Strategies
- **Graceful Degradation**: Disabling non-essential features (e.g., recommendations) when the system is under heavy load to keep the core functionality (e.g., checkout) alive.
- **Throttling (Rate Limiting)**: Limiting the number of requests a user/service can make in a given timeframe to prevent abuse.
- **Backpressure**: A strategy where a downstream service signals an upstream producer to slow down because it cannot keep up with the data flow.
- **Loadshifting**: Diverting traffic from a high-load region or cluster to one with more capacity.
- **Circuit Breaker**: Preventing cascading failures by "tripping" and stopping calls to a failing downstream service.

### 3. Migration Strategies
- **Blue-Green**: Two identical environments. Switch traffic from Blue (old) to Green (new) instantly. Easy rollback.
- **Canary Deployment**: Gradually roll out changes to a small percentage of users before a full release.
- **Strangler Fig Pattern**: Gradually replace parts of a legacy monolithic system with new microservices until the old system is "strangled."

## Go-specific Context
Go's standard library and ecosystem provide robust tools for building resilient systems.

### Circuit Breaker with `sony/gobreaker`
The `sony/gobreaker` package implements the state machine (Closed, Open, Half-Open).

```go
var cb *gobreaker.CircuitBreaker

func init() {
    cb = gobreaker.NewCircuitBreaker(gobreaker.Settings{
        Name:        "HTTP-GET",
        MaxRequests: 3,
        Interval:    5 * time.Second,
        Timeout:     30 * time.Second,
    })
}

func GetServiceData() ([]byte, error) {
    body, err := cb.Execute(func() (interface{}, error) {
        resp, err := http.Get("http://unstable-service.com")
        // ... handle response
        return body, err
    })
    return body.([]byte), err
}
```

## Interview Questions
**Q: Explain the states of a Circuit Breaker.**
**A:** 
1. **Closed**: Normal state; requests flow through. 
2. **Open**: Service has failed; requests are immediately blocked with an error. 
3. **Half-Open**: After a timeout, the breaker allows a few "test" requests. If they succeed, it returns to **Closed**; if they fail, it goes back to **Open**.

**Q: What is the difference between Horizontal and Vertical scaling?**
**A:** Vertical scaling adds resources to a single machine (limited and expensive), while Horizontal scaling adds more machines to a cluster (unlimited growth and better availability).

**Q: What is Backpressure and why is it important?**
**A:** Backpressure is a feedback mechanism where a consumer tells a producer to slow down. It prevents the consumer's buffers from overflowing and causing the system to crash or become unresponsive under heavy load.
