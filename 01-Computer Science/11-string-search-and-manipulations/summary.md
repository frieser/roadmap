---
---

# String Search — Summary

## Algorithm Comparison

| Algo | Time (avg) | Time (worst) | Space | Preprocess | Scan | Best For |
|------|-----------|-------------|-------|-----------|------|----------|
| Brute Force | O(N*M) | O(N*M) | O(1) | None | L→R | Tiny inputs, baseline |
| Rabin-Karp | O(N+M) | O(N*M) | O(1) | O(M) | L→R | Multi-pattern, plagiarism |
| KMP | O(N+M) | **O(N+M)** | O(M) | O(M) | L→R | Streams, guaranteed linear, DNA |
| Boyer-Moore | O(N/M) | O(N*M) | O(\|Σ\|) | O(M+\|Σ\|) | **R→L** | Large alphabets, production |

## Mechanics (Condensed)

- **Brute Force**: Slide +1 each mismatch. Compare L→R all M chars. No skip logic.
- **Rabin-Karp**: Rolling hash `(Base*(H - old*Base^(M-1)) + new) % Prime`. Hash match → verify chars. Spurious hit = hash collision.
- **KMP**: LPS[i] = longest proper prefix = suffix of `P[0..i]`. Mismatch → `j = LPS[j-1]`; text pointer never rewinds. LPS build: O(M), two-pointer.
- **Boyer-Moore/Horspool**: Bad char table `shift[c] = M - 1 - lastIdx(c)`. Scan R→L. Char absent → shift M. Horspool = bad char only, simpler.

## Go Strings/Bytes/Runes

- `string` = immutable bytes. `[]byte` = mutable. Conversion copies both ways.
- `len(s)` = bytes. `for _, r := range s` iterates **runes**. `s[i]` → byte, not char.
- Prefer `strings.Builder` for concat. Use `bytes` package for `[]byte` ops (mirrors `strings`).

## Go Stdlib

| Func | Pkg | Note |
|------|-----|------|
| `Index(s, sub)` | `strings` | Boyer-Moore internally |
| `Contains(s, sub)` | `strings` | Wraps Index |
| `RuneCountInString(s)` | `unicode/utf8` | Rune count |
| `Builder` | `strings` | Zero-copy concat |

## Patterns

- **Sliding window**: Fixed → rolling hash or O(N) scan. Variable → two-pointer expand/shrink.
- **Palindrome**: L/R pointers converge. Reverse: `[]rune` swap (string immutable).
- **Trie**: Prefix matching, autocomplete.
- **Regex in Go**: `regexp.MustCompile("pat")`. `FindStringIndex` / `FindAllStringIndex`.

## Pick Algorithm

| Scenario | Algo |
|----------|------|
| Tiny, quick script | Brute Force |
| Multi-pattern search | Rabin-Karp |
| Streaming, linear guarantee | KMP |
| Production, large alphabet | Boyer-Moore |
| DNA (small Σ) | KMP (stable worst-case) |
