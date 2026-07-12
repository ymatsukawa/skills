# Avoid guidance of "Concurrency Foundations"

## Thinking concurrency is always faster
Spawning goroutines has cost (scheduling, context switching). Naively parallelizing a recursive algorithm can be slower than sequential; parallelize only above a threshold where the work outweighs the overhead.

NG:
```go
// One goroutine per recursion level — even tiny slices spawn goroutines
func sortConcurrentV1(nums []int) {
    if len(nums) <= 1 {
        return
    }
    mid := len(nums) / 2
    var wg sync.WaitGroup
    wg.Add(2)
    go func() { defer wg.Done(); sortConcurrentV1(nums[:mid]) }()
    go func() { defer wg.Done(); sortConcurrentV1(nums[mid:]) }()
    wg.Wait()
    combine(nums, mid)
}
```

OK:
```go
// Fall back to sequential below a threshold to amortize goroutine cost
const minChunk = 1024
func sortConcurrentV2(nums []int) {
    if len(nums) <= 1 {
        return
    }
    if len(nums) <= minChunk {
        sortSequential(nums)
    } else {
        mid := len(nums) / 2
        var wg sync.WaitGroup
        wg.Add(2)
        go func() { defer wg.Done(); sortConcurrentV2(nums[:mid]) }()
        go func() { defer wg.Done(); sortConcurrentV2(nums[mid:]) }()
        wg.Wait()
        combine(nums, mid)
    }
}
```

## Not understanding race problems (data race vs race condition)
A data race is unsynchronized concurrent access where at least one is a write. Fix with atomics, mutexes, or channels. Even race-free code can still have a race condition: the outcome depends on timing/ordering, so synchronization alone does not guarantee determinism.

NG:
```go
// Data race: two goroutines write hits without synchronization
hits := 0
go func() { hits++ }()
go func() { hits++ }()
```

OK-1:
```go
// Atomic operation
var hits int64
go func() { atomic.AddInt64(&hits, 1) }()
go func() { atomic.AddInt64(&hits, 1) }()
```

OK-2:
```go
// Mutex protects the critical section
hits, mu := 0, sync.Mutex{}
go func() { mu.Lock(); hits++; mu.Unlock() }()
go func() { mu.Lock(); hits++; mu.Unlock() }()
```

OK-3:
```go
// Channel: communicate instead of sharing memory
hits, ch := 0, make(chan int)
go func() { ch <- 1 }()
go func() { ch <- 1 }()
hits += <-ch
hits += <-ch
```

```go
// Race condition: even with a mutex the final value of hits is nondeterministic
// (depends on which goroutine runs last). No data race, but still unstable.
go func() { mu.Lock(); defer mu.Unlock(); hits = 1 }()
go func() { mu.Lock(); defer mu.Unlock(); hits = 2 }()
```

## Ignoring the concurrency impact of the workload type (CPU- vs I/O-bound)
Match the parallelism strategy to the workload. CPU-bound work should fan out roughly to GOMAXPROCS; I/O-bound work benefits from more goroutines that hide latency while the I/O blocks. Don't add concurrency that the workload can't exploit.

NG:
```go
// Sequential: each Read blocks the whole pipeline (I/O-bound work serialized)
func scanSequential(r io.Reader) (int, error) {
    total := 0
    for {
        buf := make([]byte, 1024)
        _, err := r.Read(buf)
        if err == io.EOF { break } else if err != nil { return 0, err }
        total += process(buf)
    }
    return total, nil
}
```

OK:
```go
// Worker pool: reads feed a channel, N goroutines process concurrently
func scanConcurrent(r io.Reader) (int, error) {
    var total int64
    wg := sync.WaitGroup{}
    workers := 8
    ch := make(chan []byte, workers)
    wg.Add(workers)
    for i := 0; i < workers; i++ {
        go func() {
            defer wg.Done()
            for buf := range ch {
                atomic.AddInt64(&total, int64(process(buf)))
            }
        }()
    }
    for {
        buf := make([]byte, 1024)
        _, err := r.Read(buf)
        if err == io.EOF { break } else if err != nil { return 0, err }
        ch <- buf
    }
    close(ch)
    wg.Wait()
    return int(total), nil
}
```

## Misunderstanding Go contexts
A context carries a deadline/cancellation signal and request-scoped values across API boundaries. Use `WithTimeout`/`WithCancel` (and `defer cancel()`) to propagate cancellation, listen on `ctx.Done()` in long-running loops, and use `WithValue` for request-scoped data — not optional parameters.

OK-1:
```go
// Deadline propagation: cancel downstream work after a timeout
func (h pushHandler) pushSample(s metrics.Sample) error {
    ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
    defer cancel()
    return h.sink.Send(ctx, s)
}
```

OK-2:
```go
// Request-scoped value carried through the request's context
ctx := context.WithValue(r.Context(), isTrustedKey, r.Host == "internal")
next.ServeHTTP(w, r.WithContext(ctx))
```

OK-3:
```go
// Respect cancellation in a long-running loop via ctx.Done()
for {
    select {
    case job := <-jobs:
        _ = job // do something
    case <-ctx.Done():
        return ctx.Err()
    }
}
```
