# Avoid guidance of "Concurrency Practice"

## Propagating an inappropriate context
A goroutine outliving the request is canceled the moment the parent context (e.g. `r.Context()`) is done. Detach the context so the background work keeps its values but loses cancellation/deadline.

NG:
```go
// canceled as soon as the HTTP response is written
go func() { audit(r.Context(), entry) }()
```

OK:
```go
// custom context keeps Values but never cancels
type detached struct{ ctx context.Context }
func (d detached) Deadline() (time.Time, bool) { return time.Time{}, false }
func (d detached) Done() <-chan struct{}       { return nil }
func (d detached) Err() error                  { return nil }
func (d detached) Value(key any) any           { return d.ctx.Value(key) }

go func() { audit(detached{ctx: r.Context()}, entry) }()
```

## Starting a goroutine without knowing when to stop it
Every started goroutine needs a defined stop point, or it leaks. Give the caller a way to signal/close it.

NG:
```go
func newPoller() {
    p := poller{}
    go p.poll() // never stopped → leak
}
```

OK:
```go
// return the poller so the caller can defer p.close()
func newPoller() poller {
    p := poller{}
    go p.poll()
    return p
}
// p := newPoller(); defer p.close()
```

## Not being careful with goroutines and loop variables
Closing over a shared loop variable (pre-Go 1.22) makes all goroutines read its final value. Pass it as a parameter or copy it inside the loop.

NG:
```go
for _, id := range ids {
    go func() { fmt.Print(id) }() // captures shared id
}
```

OK-1:
```go
// pass as an argument
for _, id := range ids {
    go func(v int) { fmt.Print(v) }(id)
}
```

OK-2:
```go
// or copy into a local variable
for _, id := range ids {
    v := id
    go func() { fmt.Print(v) }()
}
```

## Expecting deterministic behavior from select with multiple channels
With several ready cases `select` picks one at random, so a "disconnect" case may fire before pending messages are drained. Drain the other channels before returning.

NG:
```go
select {
case e := <-eventCh:
    fmt.Println(e)
case <-quitCh: // may win even with events queued
    return
}
```

OK:
```go
case <-quitCh:
    for { // drain remaining events first
        select {
        case e := <-eventCh:
            fmt.Println(e)
        default:
            return
        }
    }
```

## Not using nil channels
A closed channel is always ready and yields zero values, so a naive `select` busy-loops. Set the channel to `nil` once closed so its case is disabled.

NG:
```go
// after src1 closes this select spins reading zero values
select {
case v := <-src1:
    out <- v
case v := <-src2:
    out <- v
}
```

OK:
```go
for src1 != nil || src2 != nil {
    select {
    case v, ok := <-src1:
        if !ok { src1 = nil; break } // disable this case
        out <- v
    case v, ok := <-src2:
        if !ok { src2 = nil; break }
        out <- v
    }
}
close(out)
```

## Forgetting possible side effects with string formatting (deadlock/data race)
Formatting a receiver that itself takes the lock (via its `String()` method) while already holding that lock causes a self-deadlock. Format only fields that don't re-lock.

NG:
```go
func (a *Account) SetLimit(limit int) error {
    a.mutex.Lock()
    defer a.mutex.Unlock()
    if limit < 0 {
        // %v calls a.String(), which RLocks → deadlock
        return fmt.Errorf("limit should be positive for account %v", a)
    }
    ...
}
```

OK:
```go
// format a plain field instead of the whole receiver
return fmt.Errorf("limit should be positive for account id %s", a.id)
```

## Creating data races with append
`append` on a shared slice with spare capacity writes into the same backing array from multiple goroutines. Give each goroutine its own copy.

NG:
```go
buf := make([]int, 0, 1)
go func() { _ = append(buf, 9) }() // shared backing array
go func() { _ = append(buf, 9) }()
```

OK:
```go
// copy before appending so backing arrays differ
go func() {
    local := make([]int, len(buf), cap(buf))
    copy(local, buf)
    _ = append(local, 9)
}()
```

## Using mutexes inaccurately with slices and maps
Copying a map/slice header under the lock and then iterating outside it still shares the underlying data, so concurrent writers race. Deep-copy under the lock, or do all the work while holding it.

NG:
```go
r.mu.RLock()
scores := r.scores // just copies the map header
r.mu.RUnlock()
for _, s := range scores { ... } // races with writers
```

OK-1:
```go
// hold the lock for the whole read
r.mu.RLock()
defer r.mu.RUnlock()
for _, s := range r.scores { sum += s }
```

OK-2:
```go
// or copy the contents under the lock
r.mu.RLock()
snapshot := make(map[string]int, len(r.scores))
for k, v := range r.scores { snapshot[k] = v }
r.mu.RUnlock()
```

## Misusing sync.WaitGroup
Calling `wg.Add(1)` inside the goroutine races with `wg.Wait()`, which may return before the counter is set. Add before launching, or add the total up front.

NG:
```go
for i := 0; i < 4; i++ {
    go func() {
        wg.Add(1) // racing with Wait
        atomic.AddUint64(&ops, 1)
        wg.Done()
    }()
}
wg.Wait()
```

OK-1:
```go
// Add the total once before the loop
wg.Add(4)
for i := 0; i < 4; i++ {
    go func() { atomic.AddUint64(&ops, 1); wg.Done() }()
}
```

OK-2:
```go
// or Add(1) before each goroutine starts
for i := 0; i < 4; i++ {
    wg.Add(1)
    go func() { atomic.AddUint64(&ops, 1); wg.Done() }()
}
```

## Forgetting about sync.Cond
Waiting on a condition by spin-looping on a mutex burns CPU; signaling via a channel can't fan out to multiple waiters. Use `sync.Cond` to notify many waiters efficiently.

NG:
```go
// busy spin until the target is reached
stock.mu.RLock()
for stock.count < target {
    stock.mu.RUnlock()
    stock.mu.RLock()
}
stock.mu.RUnlock()
```

OK:
```go
// listener waits; updater Broadcasts to all waiters
stock.cond.L.Lock()
for stock.count < target {
    stock.cond.Wait()
}
stock.cond.L.Unlock()
// updater: count++; cond.Broadcast()
```

## Not using errgroup
A bare `WaitGroup` can't propagate the first error or cancel the other goroutines. `errgroup.WithContext` collects the first error and cancels the shared context.

NG:
```go
wg.Add(len(results))
for i, region := range regions {
    go func() {
        defer wg.Done()
        result, err := fetch(ctx, region) // err can't be returned
        results[i] = result
    }()
}
wg.Wait()
```

OK:
```go
g, ctx := errgroup.WithContext(ctx)
for i, region := range regions {
    i, region := i, region
    g.Go(func() error {
        result, err := fetch(ctx, region)
        if err != nil { return err }
        results[i] = result
        return nil
    })
}
if err := g.Wait(); err != nil { return nil, err }
```

## Copying a sync type
sync types (Mutex, WaitGroup, etc.) must not be copied; a value-receiver method copies the struct and its mutex, so the lock protects nothing. Use a pointer receiver, or store the sync type as a pointer.

NG:
```go
type Tally struct {
    mu     sync.Mutex // copied with the struct
    counts map[string]int
}
func (t Tally) Bump(name string) { // value receiver copies mu
    t.mu.Lock()
    defer t.mu.Unlock()
    t.counts[name]++
}
```

OK:
```go
// pointer receiver, or store the mutex as a pointer
func (t *Tally) Bump(name string) { ... }

type Tally2 struct {
    mu     *sync.Mutex
    counts map[string]int
}
```
