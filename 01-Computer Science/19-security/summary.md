---
---

# Security — Summary

## CIA Triad
- Confidentiality: Encryption, access control — no unauthorised read.
- Integrity: Hashing, signatures — no unauthorised modification.
- Availability: Redundancy, rate-limiting — system stays reachable.

## Attacks
| Attack               | Type          | Mitigation                       |
|----------------------|---------------|----------------------------------|
| SQLi                 | Injection     | Parameterised queries            |
| XSS                  | Injection     | Context-aware output encoding    |
| CSRF                 | Request forgery | CSRF tokens, SameSite cookies  |
| SSRF                 | Request forgery | URL allow-lists, disable redirects |
| Buffer overflow      | Memory        | NX bit, stack canaries           |
| ROP                  | Code reuse    | ASLR, shadow stacks (CET)        |
| Credential stuffing  | Brute-force   | Rate limiting, MFA               |
| Supply chain         | Dependency    | SBOM, `govulncheck`              |

## AuthN vs AuthZ
- AuthN: Who you are — verified via password, JWT, Passkey, OIDC.
- AuthZ: What you can do — enforced via ACL, RBAC, OAuth scopes.

## Hash / Encrypt / Encode
- Hash: One-way, fixed output, integrity (SHA-256, bcrypt, Argon2).
- Encrypt: Two-way, key required, confidentiality (AES, RSA, ECC).
- Encode: Two-way, no key, compatibility (Base64, URL encoding).

## PKC
- RSA: integer factorisation. ECC: elliptic curve discrete log.
- ECC wins — smaller keys, faster, same security (256b ECC ≈ 3072b RSA).
- TLS uses hybrid: asymmetric for session key, symmetric for data.

## Password Hashing
- Never SHA-256 (too fast). Use bcrypt (cost factor) or Argon2 (memory-hard).
- Salt: random per-user. Pepper: secret stored outside DB.
- Re-hash on login when upgrading cost/algo.

## Memory Protections
- NX/DEP: stack/heap non-executable — kills shellcode injection.
- ASLR: randomises addresses — defeats hardcoded ROP gadgets.
- Stack canary: secret value before return address — detects overflows.

## Auth Strategies
- Sessions: stateful, easy revoke, CSRF risk. JWT: stateless, scalable, XSS risk.
- OAuth 2.0 = authorisation (Access Token). OIDC = authentication (ID Token).
- Passkeys/WebAuthn: asymmetric, phishing-resistant, private key never leaves device.
