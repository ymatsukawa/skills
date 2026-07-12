# Avoid guidance of "Strings"

## Not understanding the concept of a rune
A string is a sequence of bytes (UTF-8), not characters; `len` returns byte count, not rune count. A rune is a Unicode code point.

NG:
```go
// "汉" is one character but 3 bytes — surprising if you expect 1
word := "汉"
fmt.Println(len(word)) // 3, not 1
```

OK:
```go
// count runes, not bytes, when you mean characters
word := "汉"
fmt.Println(utf8.RuneCountInString(word)) // 1
```

## Inaccurate string iteration (bytes vs runes)
`range` over a string yields rune start positions and the rune value; indexing `s[i]` gives a byte. Use the `range` rune value, not `s[i]`, to iterate over characters.

NG:
```go
// name[i] is a byte at a rune-start index → wrong for multibyte runes
for i := range name {
    fmt.Printf("%c", name[i]) // prints byte, not rune
}
```

OK:
```go
// the second range value ch is the rune
for _, ch := range name {
    fmt.Printf("%c", ch)
}
```

## Misusing trim functions (TrimRight vs TrimSuffix)
`TrimRight`/`TrimLeft` strip any trailing/leading chars in the cutset (repeatedly); `TrimSuffix`/`TrimPrefix` strip one exact substring. Pick by intent.

NG:
```go
// TrimRight removes every trailing 'k'/'a', not the "ka" suffix
func stripMarker(id string) string {
    return strings.TrimRight(id, "ka") // "789kaka" -> "789"
}
```

OK:
```go
// TrimSuffix removes the exact "ka" suffix once
func stripMarker(id string) string {
    return strings.TrimSuffix(id, "ka") // "789kaka" -> "789ka"
}
```

## Under-optimized string concatenation
Repeated `+=` reallocates on every iteration. Use `strings.Builder`, and preallocate with `Grow` when the total size is known.

NG:
```go
// reallocates each loop — O(n^2)
joined := ""
for _, p := range parts { joined += p }
```

OK:
```go
// Builder + Grow(size) avoids repeated allocations
var sb strings.Builder
sb.Grow(size)
for _, p := range parts { sb.WriteString(p) }
return sb.String()
```

## Useless string conversions
Converting `[]byte` to `string` (and back) just to use a string helper costs an extra copy. The `bytes` package mirrors `strings`, so operate on bytes directly.

NG:
```go
// extra []byte<->string conversions
return []byte(strings.TrimSpace(string(raw))), nil
```

OK:
```go
// bytes package avoids the conversions
return bytes.TrimSpace(raw), nil
```

## Substrings and memory leaks
A substring shares the backing array of the original string, keeping the whole (possibly huge) string alive. Copy the substring to release the original.

NG:
```go
// token keeps the entire large payload alive
token := payload[:16]
c.store(token)
```

OK:
```go
// strings.Clone copies, freeing the original backing array
token := strings.Clone(payload[:16])
c.store(token)
```
