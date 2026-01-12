# Native Package Manager

## Summary
Zig includes a decentralized package manager. Dependencies are defined in `build.zig.zon` (Zig Object Notation). Zig uses content-addressable storage (multihash) to ensure integrity and reproducibility.

## Detailed Explanation

### `build.zig.zon`
A manifest file.
```zig
.{
    .name = "my_project",
    .version = "0.1.0",
    .dependencies = .{
        .zap = .{
            .url = "https://github.com/...",
            .hash = "1220...",
        },
    },
}
```

### Fetching
`zig build` automatically fetches dependencies. If a hash is missing, it errors and provides the correct hash (Trust-On-First-Use).

### Caching
Dependencies are cached globally (in `~/.cache/zig` or similar), preventing redundant downloads across projects.

### Go Comparison
*   **Go**: `go.mod` / `go.sum`. Centralized proxy (usually).
*   **Zig**: `build.zig.zon`. Decentralized (URLs).

## Interview Questions

**Q: What format is `build.zig.zon`?**
**A:** It is ZON (Zig Object Notation), which is a data-only subset of Zig syntax. It is struct-like and supports comments.

**Q: How does Zig ensure dependency integrity?**
**A:** It requires a generic multihash (usually SHA-256) for every dependency in `.zon`. If the content at the URL changes, the hash mismatch causes a build error.

**Q: What happens if you don't provide a hash for a dependency?**
**A:** The compiler will fetch the package, calculate the hash, and then fail the build, printing the calculated hash. The developer is expected to verify this hash and add it to `build.zig.zon`.
