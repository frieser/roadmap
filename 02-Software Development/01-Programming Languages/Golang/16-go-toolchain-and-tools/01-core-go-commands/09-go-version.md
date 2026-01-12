# Go Version

## Summary
The `go version` command prints the version of the Go runtime installed on the system. It is also used to inspect binaries to see which Go version they were built with, which is critical for debugging reproducibility issues or security audits.

## Detailed Explanation

### Usage

1.  **Check Installed Version**:
    ```bash
    $ go version
    go version go1.21.0 linux/amd64
    ```

2.  **Inspect a Binary**:
    You can point it at any Go executable to see its build info.
    ```bash
    $ go version ./my-app-binary
    ./my-app-binary: go1.20.5
    ```

3.  **Detailed Build Info (`-m`)**:
    The `-m` flag (module info) shows dependencies and build settings.
    ```bash
    $ go version -m ./my-app-binary
    ./my-app-binary: go1.21.0
        path  github.com/my/app
        dep   github.com/gin-gonic/gin v1.9.0
        build -compiler=gc
        build CGO_ENABLED=1
    ```

## Interview Questions

**Q: How can you check what version of Go was used to compile a specific binary file?**
**A:** By running `go version <path-to-binary>`. This reads the build metadata embedded in the executable.

**Q: What does `go version -m` output?**
**A:** It outputs the module version information embedded in the binary, including the main module path, the Go version used, and a list of all dependency modules and their exact versions (hashes). This is effectively a "Bill of Materials" (SBOM) for the binary.
