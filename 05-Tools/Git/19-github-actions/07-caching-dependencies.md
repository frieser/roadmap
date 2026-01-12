# Caching Dependencies

## Summary
Caching speeds up workflows by reusing files (like downloaded dependencies) from previous runs. This significantly reduces build time.

## Detailed Explanation

### Usage
Use the `actions/cache` action. You need a **key** (usually a hash of a lockfile) and a **path** (folder to cache).

### Go-specific Context
Go modules are perfect for caching. You cache the `~/go/pkg/mod` directory based on the hash of `go.sum`.

```yaml
- uses: actions/cache@v3
  with:
    path: ~/go/pkg/mod
    key: ${{ runner.os }}-go-${{ hashFiles('**/go.sum') }}
    restore-keys: |
      ${{ runner.os }}-go-
```
If `go.sum` hasn't changed, `go mod download` will be nearly instant.
