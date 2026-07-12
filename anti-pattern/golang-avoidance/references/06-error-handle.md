# Avoid guidance of "Error Management"

## Mishandling panic
Reserve `panic` for truly unrecoverable conditions or programmer errors; recover from it at a boundary instead of letting it crash the process.

NG:
```go
// panic propagates and crashes if no recover is set
func load() {
	panic("bad state")
}
```

OK:
```go
// recover at a boundary; panic only for genuinely invalid state
defer func() {
	if r := recover(); r != nil {
		fmt.Println("recover", r)
	}
}()
load()
```

## Ignoring when to wrap an error
Wrap with `%w` to add context while keeping the source unwrappable; use `%v` only when you want to obscure it; return as-is when you add nothing.

NG:
```go
// adds context but breaks errors.Is/As (loses source error)
return fmt.Errorf("fetch config failed: %v", err)
```

OK-1:
```go
// wrap with %w: adds context and stays unwrappable
return fmt.Errorf("fetch config failed: %w", err)
```

OK-2:
```go
// no context to add: return the error directly
return err
```

## Comparing an error type imprecisely (use errors.As)
A wrapped error fails a type assertion or switch; use `errors.As` to match a target type anywhere in the chain.

NG:
```go
// fails once the error is wrapped with %w
switch err := err.(type) {
case timeoutError:
	// ...
}
```

OK:
```go
// matches timeoutError even when wrapped
if errors.As(err, &timeoutError{}) {
	// ...
}
```

## Comparing an error value imprecisely (use errors.Is)
Comparing with `==` misses sentinel errors hidden behind a wrap; use `errors.Is` to walk the chain.

NG:
```go
// fails if err is wrapped around os.ErrNotExist
if err == os.ErrNotExist {
	// ...
}
```

OK:
```go
// matches os.ErrNotExist anywhere in the chain
if errors.Is(err, os.ErrNotExist) {
	// ...
}
```

## Handling an error twice
Logging an error and also returning it duplicates the report; either log it once or wrap it with context and return.

NG:
```go
// logs AND returns: the same error is reported twice
log.Println("failed to load user profile")
return Profile{}, err
```

OK:
```go
// wrap with context and return once; caller logs
return Profile{}, fmt.Errorf("failed to load user profile: %w", err)
```

## Not handling an error
Silently dropping a returned error hides failures; if ignoring is intentional, make it explicit with `_` and a comment.

NG:
```go
func shutdown(w *worker) {
	// flush's returned error silently discarded
	w.flush()
	w.stop()
}
```

OK:
```go
func shutdown(w *worker) {
	// intentionally ignored, documented as best-effort
	_ = w.flush()
	w.stop()
}
```

## Not handling errors returned by deferred calls
A deferred call's error is easy to lose; ignore it explicitly with `_`, or capture it via a named return value when it matters.

NG:
```go
// Close's error is dropped (linter warning)
defer f.Close()
```

OK-1:
```go
// explicitly ignore the error
defer func() { _ = f.Close() }()
```

OK-2:
```go
// propagate close error via named return when no prior error
func writeReport(...) (n int, err error) {
	defer func() {
		closeErr := f.Close()
		if err == nil {
			err = closeErr
		}
	}()
}
```
