# Avoid guidance of "Functions and Methods"

## Choosing value vs pointer receiver
Value receivers operate on a copy, so mutations are lost; pointer receivers mutate the original. Choose based on whether the method must modify the receiver.

NG:
```go
// value receiver: mutation lost, points stays 100
func (w wallet) deposit(v int) { w.points += v }
```

OK-1:
```go
// pointer receiver: required when the method mutates the struct
func (w *wallet) deposit(v int) { w.points += v }
```

OK-2:
```go
// value receiver is fine when fields are pointers; mutation via the pointer persists
func (w wallet) deposit(v int) { w.ledger.points += v }
```

## Not using named result parameters when helpful
Named results document the meaning of return values and allow a naked `return`. Use them when they improve readability, e.g. two same-typed results.

NG:
```go
// unclear which float64 is which
parseRange(input string) (float64, float64, error)
```

OK:
```go
// named results clarify intent
parseRange(input string) (low, high float64, err error)
```

## Unintended side effects with named result parameters
A named result starts at its zero value; shadowing it with `:=` can leave the named `err` nil so a naked return masks the real error. Assign to the named result instead.

NG:
```go
// shadows err; the named err result stays nil
if err := ctx.Err(); err != nil { return 0, 0, err }
```

OK:
```go
// assign to the named result, then naked return propagates it correctly
if err = ctx.Err(); err != nil { return }
```

## Returning a nil receiver
A typed nil pointer wrapped in an interface is not nil. Returning a nil `*ValidationErrors` as `error` yields a non-nil interface, so callers' `err != nil` checks wrongly pass.

NG:
```go
// returns a non-nil error interface even when v is nil
var v *ValidationErrors
return v // error holding (*ValidationErrors)(nil) != nil
```

OK:
```go
// return literal nil when there is nothing to report
if v != nil { return v }
return nil
```

## Using a filename instead of io.Reader as function input
Taking a filename couples the function to the filesystem and hurts testability. Accept `io.Reader` so callers can pass files, strings, sockets, etc.

NG:
```go
// only works with files on disk; hard to test
func countBlankLines(path string) (int, error) { f, _ := os.Open(path); /* ... */ }
```

OK:
```go
// any source works; trivial to test with strings.NewReader
func countBlankLines(src io.Reader) (int, error) { scanner := bufio.NewScanner(src); /* ... */ }
```

## Ignoring how defer arguments and receivers are evaluated
`defer` evaluates its arguments and the receiver immediately, not when the deferred call runs. Use a pointer or a closure to capture the final value.

NG:
```go
// state captured as "" at defer time; later assignment ignored
var state string
defer report(state)
state = StateDone
```

OK-1:
```go
// closure reads state when it actually runs
defer func() { report(state) }()
state = StateDone
```

OK-2:
```go
// pass a pointer so the deferred call dereferences the final value
defer reportPtr(&state)
state = StateDone
```
