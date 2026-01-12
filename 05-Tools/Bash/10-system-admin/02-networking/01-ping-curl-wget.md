---
---

## Summary
`ping`, `curl`, and `wget` are the foundational tools for network troubleshooting and data retrieval. `ping` checks connectivity (ICMP), while `curl` and `wget` interact with web servers (HTTP/HTTPS/FTP).

## Detailed Explanation

### Ping
*   **Purpose**: Test reachability and measure round-trip time (RTT).
*   **Usage**: `ping google.com`.
*   **Output**: `64 bytes from ... time=14.2 ms`.

### Curl vs Wget
*   **curl**: "Client URL". Designed for transferring data. Supports many protocols (HTTP, FTP, SCP). Outputs to stdout by default. Great for API testing.
    *   `curl -X POST -d "data" https://api.site.com`
*   **wget**: "World Wide Web Get". Designed for downloading files. Supports recursive download (`-r`), resuming (`-c`), and works well on unstable connections. Outputs to file by default.
    *   `wget https://site.com/file.zip`

## Go-Specific Context/Examples

Go's standard library `net/http` provides `Get`, `Post`, and a full Client.

### Analogy
*   **Bash**: `curl https://site.com`
*   **Go**:
    ```go
    resp, _ := http.Get("https://site.com")
    defer resp.Body.Close()
    io.Copy(os.Stdout, resp.Body)
    ```

## Interview Questions

**Q: Why might `ping` fail even if the website is accessible via browser?**
**A:** Because `ping` uses **ICMP** (Internet Control Message Protocol), while browsers use **TCP** (Port 80/443). Many firewalls block ICMP packets to prevent flooding/scanning but allow TCP web traffic.

**Q: How do you inspect HTTP headers with curl?**
**A:** `curl -I https://site.com` (Fetch headers only) or `curl -v` (Verbose mode, shows request and response headers).

**Q: Which tool is better for downloading a 10GB file over a flaky connection?**
**A:** **wget**. It has a robust retry mechanism and resume capability (`wget -c`). While `curl` can resume (`-C -`), `wget` is generally preferred for this specific use case out of the box.
