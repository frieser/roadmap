---
---

# Endianness — Byte ordering of multi-byte types in memory. Strings unaffected.

## Big vs Little
- **BE**: MSB at lowest address. `0x1234` → `12 34`
- **LE**: LSB at lowest address. `0x1234` → `34 12`
- BE: human-readable dumps, sign bit offset 0. LE: fast arithmetic (LSB offset 0), easy casting.
  BE CPUs: SPARC, IBM z. LE CPUs: x86, AMD64, ARM64, RISC-V.

## Network Byte Order
- Big-Endian. IP/TCP/UDP standard. No implicit conversion — always explicit.
- C: `htonl`/`ntohl`. Go: `binary.BigEndian`. Sending LE over network → reversed, wrong value.

## Go `encoding/binary`
- `binary.BigEndian.PutUint32(buf, v)` → MSB first. `binary.LittleEndian.PutUint32(buf, v)` → LSB first.
- `unsafe.Pointer` casting only works on matching host endianness. Portability hazard.

## Detection
- `uint32(1)`: first byte `0` = BE, `1` = LE.

## Architectures
| BE | LE |
|---|---|
| SPARC, IBM z, Motorola 68k | x86, AMD64, ARM64, RISC-V |

## Formats
| BE | LE |
|---|---|
| TCP/IP, JPEG, Java .class | BMP, PE, PNG (some) |
