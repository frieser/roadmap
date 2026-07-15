---
---

# Networking — Ultra-Compact Summary

## OSI 7-Layer Model

| # | Layer | Function | Protocols / Devices |
|:-:|:------|:---------|:---------------------|
| 7 | Application | User-facing services, resource ID | HTTP, DNS, SSH, SMTP, FTP |
| 6 | Presentation | Translation, encryption, compression | TLS/SSL, ASCII, JPEG, gzip |
| 5 | Session | Session open/close, sync | SOCKS, RPC, NetBIOS |
| 4 | Transport | End-to-end delivery, flow/error control | TCP (segments), UDP (datagrams) |
| 3 | Network | Logical addressing, routing | IP (v4/v6), ICMP, Routers |
| 2 | Data Link | Physical addressing, error detect, media access | Ethernet, MAC, Switches |
| 1 | Physical | Raw bit transmission | Copper, Fiber, Radio, Hubs |

**L4 vs L7 LB:** L4 = IP+port only, fast, blind. L7 = inspects headers/URLs, smart routing, SSL term, higher CPU.

## TCP vs UDP — 5 Key Differences

- TCP: connection-oriented (3-way: SYN→SYN-ACK→ACK). UDP: connectionless, fire-and-forget.
- TCP: guaranteed delivery via ACK + retransmission. UDP: best-effort, no delivery guarantees.
- TCP: ordered delivery (sequence numbers). UDP: no ordering, packets may arrive out of sequence.
- TCP: flow control (sliding window) + congestion control (slow start, AIMD). UDP: none.
- TCP: SOCK_STREAM, 20–60B header, slower. UDP: SOCK_DGRAM, 8B header, fast → DNS, VoIP, streaming, gaming.

## Key Protocols

| Protocol | Port | Purpose | TLS / Secure Variant |
|:---------|:----:|:--------|:---------------------|
| HTTP | 80 | REST, web, APIs | HTTPS (443) via TLS |
| DNS | 53 | Name → IP resolution | DoT (853), DoH (443) |
| SSH | 22 | Remote shell, tunneling, SFTP | Native encryption (no TLS needed) |
| SMTP | 25/587 | Sending email | SMTPS (465), STARTTLS (587) |
| IMAP | 143 | Receive email, multi-device sync | IMAPS (993) |
| POP3 | 110 | Receive email, download+delete | POP3S (995) |
| FTP | 21 | File transfer (legacy) | FTPS (TLS), SFTP (SSH, port 22) |

**TLS Handshake:** 1.2 = 2-RTT, RSA or DH. 1.3 = 1-RTT, ECDHE mandatory, 0-RTT resumption, AEAD only.
**PKI Chain:** Root CA (self-signed, in OS/browser store) → Intermediate → Leaf cert.
**SNI:** hostname in ClientHello, enables multi-domain on single IP.
**mTLS:** client also presents cert — service-to-service, Zero Trust.

**DNS:** UDP default (fast, <512B), TCP if TC bit or zone transfer (AXFR). Hierarchy: Root (.) → TLD (.com) → Authoritative.
**Records:** A (IPv4), AAAA (IPv6), CNAME (alias), MX (mail), NS (zone delegation), PTR (reverse), TXT (SPF/DKIM).

**Email Anti-Spoof:** SPF (auth IPs in TXT), DKIM (body+header signature in TXT), DMARC (policy: none/quarantine/reject).

**HTTP Methods:** GET/PUT/DELETE/HEAD/OPTIONS = idempotent. POST/PATCH = not.
**Status:** 2xx=success, 3xx=redirect, 4xx=client error, 5xx=server error.
502 = bad upstream response. 504 = upstream timeout.

**HTTP/1.1 vs HTTP/2:** 1.1 = text, head-of-line blocking. 2 = binary, multiplexing, HPACK, server push.

## Sockets — OS Interface

| Call | Role | Side |
|:-----|:-----|:----:|
| `socket()` | Create endpoint, return FD | Both |
| `bind()` | Assign local IP:port | Server |
| `listen()` | Mark passive, set backlog | Server |
| `accept()` | Block, return new FD per conn | Server |
| `connect()` | Active open to remote | Client |
| `send()`/`recv()` | Transmit / receive data | Both |
| `close()` | Release FD | Both |

**I/O Multiplex:** `select()` (1024 FD, O(N), legacy) → `poll()` (high FD, O(N)) → `epoll()` (Linux, O(1), event-driven, Nginx/Redis).

## Go `net` — Quick Reference

- `net.DialTimeout("tcp", "host:port", timeout)` — L4 connectivity check.
- `net.Listen("tcp", ":8080")` → `Accept()` → goroutine per conn — Go netpoller (epoll/kqueue) makes this scale.
- `tls.LoadX509KeyPair(crt, key)` → `tls.Config` → `server.ListenAndServeTLS("","")` — HTTPS server.
