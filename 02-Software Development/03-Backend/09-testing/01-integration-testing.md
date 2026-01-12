---
---

## Summary
Integration testing verifies that different modules or services of an application work together correctly. Unlike unit testing, integration tests focus on the interactions between components, such as the application and its database, cache, or external APIs.

## Detailed Explanation
Integration tests catch bugs that unit tests miss, such as configuration errors, schema mismatches, or incorrect assumptions about how a third-party library behaves.

### Key Characteristics
- **Scope**: Covers multiple components (e.g., Service + Repository + Database).
- **Environment**: Requires a setup that closely resembles production, often using real or containerized dependencies.
- **Speed**: Slower than unit tests due to I/O and setup/teardown overhead.

### Integration Testing in Go
In Go, integration tests often live in the same package or a dedicated `tests/` directory. A common practice is to use **build tags** to prevent integration tests from running during a quick unit test pass.

#### Strategies for External Dependencies
1.  **Docker (Testcontainers-go)**: The modern standard. Automatically spins up a Docker container (e.g., Postgres, Redis) for the duration of the test.
2.  **Shared Test Database**: Running tests against a dedicated local or CI database instance. Requires careful management of state (cleanup before/after tests).
3.  **In-Memory Databases**: Using an in-memory version of a database (e.g., SQLite for Postgres tests). Fast, but might hide provider-specific issues.

#### Build Tags
Using `//go:build integration` at the top of the file allows you to run these tests specifically with `go test -tags=integration ./...`.

## Go-specific Examples

### Integration Test with Build Tags
```go
//go:build integration

package repository_test

import (
    "context"
    "testing"
    "github.com/stretchr/testify/assert"
    "myproject/internal/repository"
)

func TestUserRepository_Create(t *testing.T) {
    // Assuming db is a connection to a real test database
    repo := repository.NewUserRepository(db)
    
    user := &repository.User{Name: "John Doe"}
    err := repo.Create(context.Background(), user)
    
    assert.NoError(t, err)
    assert.NotZero(t, user.ID)
}
```

### Using Testcontainers-go
```go
func TestWithPostgres(t *testing.T) {
    ctx := context.Background()
    container, err := postgres.RunContainer(ctx,
        testcontainers.WithImage("postgres:15-alphine"),
        postgres.WithDatabase("testdb"),
        postgres.WithUsername("user"),
        postgres.WithPassword("pass"),
    )
    if err != nil {
        t.Fatal(err)
    }
    defer container.Terminate(ctx)

    connStr, _ := container.ConnectionString(ctx)
    // Use connStr to connect your repository and run tests...
}
```

## Interview Questions
**Q: How do you separate unit tests from integration tests in a Go project?**
**A:** The most common way is using **build tags**. By adding `//go:build integration` to the top of integration test files, they are excluded by default. You run them explicitly using `go test -tags=integration`. Another way is using the `-short` flag in `testing.T` and checking `if testing.Short() { t.Skip(...) }`.

**Q: What are the trade-offs of using an in-memory database for integration tests?**
**A:** In-memory databases (like SQLite in place of Postgres) are extremely fast and require no external setup. However, they may not support all features of the production database (like specific JSONB operations or triggers), potentially leading to "green tests" that fail in production.

**Q: What is the purpose of `TestMain(m *testing.M)`?**
**A:** `TestMain` is a special function that, if present, Go will run instead of running the tests directly. It allows you to perform global setup (e.g., starting a database container) and teardown (e.g., stopping the container) for all tests in a package.
