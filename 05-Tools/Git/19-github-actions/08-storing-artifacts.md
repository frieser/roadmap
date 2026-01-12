# Storing Artifacts

## Summary
Artifacts are files produced by a workflow run (like binaries, logs, or test reports) that you want to persist after the run finishes.

## Detailed Explanation

### Uploading
```yaml
- uses: actions/upload-artifact@v4
  with:
    name: my-app-binary
    path: bin/my-app
```

### Downloading
You can download artifacts from the UI (Actions tab) or in a subsequent job (if you have a build job and a deploy job) using `download-artifact`.

### Go-specific Context
Common workflow:
1.  **Build Job**: `go build -o my-app`. Upload `my-app` artifact.
2.  **Release Job**: Download `my-app` artifact. Upload it to GitHub Releases.
