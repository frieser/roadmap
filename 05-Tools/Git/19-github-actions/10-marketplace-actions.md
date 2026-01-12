# Marketplace Actions

## Summary
The GitHub Marketplace contains thousands of pre-built actions created by the community. You can "use" them in your workflow without writing the code yourself.

## Detailed Explanation

### Popular Actions
*   `actions/checkout`: Check out your code.
*   `actions/setup-go`: Install Go.
*   `docker/build-push-action`: Build Docker images.
*   `golangci/golangci-lint-action`: Run the standard Go linter.

### Usage
```yaml
- uses: golangci/golangci-lint-action@v3
  with:
    version: v1.54
```

### Security Warning
Be careful when using actions from unknown authors. They run with access to your code and secrets. Stick to verified creators or pin actions to a specific commit SHA for security.
