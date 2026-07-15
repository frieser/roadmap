---
---

## Encoding Comparison

| Encoding | Bytes/Char | Fixed? | Range | ASCII Compat | BOM | Endian |
|----------|-----------|--------|-------|-------------|-----|--------|
| ASCII | 1 | Yes (7-bit) | 0–127 | — | No | — |
| UTF-8 | 1–4 | No | U+0000–U+10FFFF | Yes | Optional | Independent |
| UTF-16 | 2 or 4 | No | U+0000–U+10FFFF | No | Required | LE/BE |
| UTF-32 | 4 | Yes | U+0000–U+10FFFF | No | Required | LE/BE |

## UTF-8 Byte Layout

| Range | Bytes | Byte 1 Pattern | Continuation |
|-------|-------|----------------|-------------|
| U+0000–U+007F | 1 | `0xxxxxxx` | — |
| U+0080–U+07FF | 2 | `110xxxxx` | `10xxxxxx` |
| U+0800–U+FFFF | 3 | `1110xxxx` | `10xxxxxx` ×2 |
| U+10000–U+10FFFF | 4 | `11110xxx` | `10xxxxxx` ×3 |

## Go Rune/Byte

- `byte` = `uint8` (raw data / ASCII) | `rune` = `int32` (Unicode code point)
- `len(s)` → byte count | `utf8.RuneCountInString(s)` → character count
- `for i, r := range s` → iterates runes (auto-decodes UTF-8); `i` = byte offset, `r` = code point
- `s[i]` → single byte, NOT i-th character | `utf8.ValidString(s)` → validate UTF-8
- Known-ASCII data: byte ops O(1) | Non-ASCII: always use `range` + `unicode/utf8`

## UTF-16 Surrogates

| Type | Range |
|------|-------|
| High surrogate | U+D800–U+DBFF |
| Low surrogate | U+DC00–U+DFFF |
| BMP direct | U+0000–U+FFFF (excl. surrogates) |

## ASCII Ranges

| Chars | Decimal |
|-------|---------|
| Ctrl | 0–31 |
| Space | 32 |
| 0–9 | 48–57 |
| A–Z | 65–90 |
| a–z | 97–122 |
| DEL | 127 |
