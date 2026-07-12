# Avoid guidance of "Standard Library"

## Providing a wrong time.Duration
`time` functions taking a `time.Duration` accept an `int64`, so a bare number is interpreted as nanoseconds, not the unit you expect. Always multiply by a unit constant.

NG:
```go
// 500 nanoseconds, not 500 milliseconds
ticker := time.NewTicker(500)
```

OK:
```go
// explicit unit
ticker := time.NewTicker(500 * time.Millisecond)
```

## time.After and memory leaks
In a loop, `time.After` creates a channel/resource that lives until the duration elapses, leaking memory while the loop spins. Reuse a single timer (or a context) instead.

NG:
```go
// new resource each iteration, freed only after the duration
for {
    select {
    case req := <-in:
        serve(req)
    case <-time.After(30 * time.Minute):
        log.Println("warning: no requests")
    }
}
```

OK:
```go
// reuse one timer and Reset it
timer := time.NewTimer(30 * time.Minute)
for {
    timer.Reset(30 * time.Minute)
    select {
    case req := <-in:
        serve(req)
    case <-timer.C:
        log.Println("warning: no requests")
    }
}
```

## Common JSON handling mistakes
Three traps: an embedded type with `MarshalJSON` flattens output unexpectedly; `time.Time` keeps a monotonic clock that breaks equality after a round-trip; `map[string]any` decodes all numbers as `float64`.

NG:
```go
// embedded time.Time promotes its MarshalJSON: output is just a date string
type Receipt struct {
    Code int
    time.Time
}
// "2024-..." instead of {"Code":42,"Time":"..."}
```

OK:
```go
// name the field (or define a custom MarshalJSON wrapper)
type Receipt struct {
    Code   int
    Issued time.Time
}
```

NG:
```go
// monotonic clock present in r1 but lost after round-trip -> not equal
r1 := Receipt{Issued: time.Now()}
// after Marshal/Unmarshal: r1 == r2 is false
```

OK:
```go
// strip monotonic clock with Truncate(0) before comparing
r1 := Receipt{Issued: time.Now().Truncate(0)}
```

NG:
```go
// numbers in the map become float64, not int
var payload map[string]any
json.Unmarshal(raw, &payload)
```

OK:
```go
// unmarshal into a typed struct to control types
type Stats struct{ Count int }
var stats Stats
json.Unmarshal(raw, &stats)
```

## Common SQL mistakes
`sql.Open` may not open a connection, so `Ping` to verify it. Use `sql.NullXxx` for nullable columns, prepared statements for repeated queries, and always check `rows.Err()` after iterating.

NG:
```go
// no connection check; nullable column scanned into plain string fails on NULL
db, _ := sql.Open("postgres", dsn)
var city string
rows.Scan(&city, &score)
```

OK:
```go
// verify connection, use sql.NullString for nullable columns
db, _ := sql.Open("postgres", dsn)
if err := db.Ping(); err != nil { return err }
var city sql.NullString
rows.Scan(&city, &score)
```

NG:
```go
// loop can stop early on a hidden error that goes undetected
for rows.Next() {
    rows.Scan(&city, &score)
}
return city, score, nil
```

OK:
```go
// check rows.Err() after the loop
for rows.Next() {
    rows.Scan(&city, &score)
}
if err := rows.Err(); err != nil { return err }
```

## Not closing transient resources
HTTP response bodies, `os.File`, and `sql.Rows` must be closed to free connections/descriptors. For HTTP keep-alive, drain the body before closing; on writable files, surface the close/`Sync` error.

NG:
```go
// body never closed -> connection leak
resp, _ := c.http.Get(c.endpoint)
data, _ := io.ReadAll(resp.Body)
return string(data), nil
```

OK:
```go
// defer Close; drain body so the connection can be reused
resp, _ := c.http.Get(c.endpoint)
defer resp.Body.Close()
io.Copy(io.Discard, resp.Body)
```

OK:
```go
// on writable files, return the close error
defer func() {
    closeErr := out.Close()
    if err == nil { err = closeErr }
}()
```

## Forgetting the return statement after replying to an HTTP request
`http.Error` does not stop the handler. Without `return`, execution continues and writes again, producing a duplicated/conflicting response.

NG:
```go
// keeps going after the error reply
if err != nil {
    http.Error(w, "save failed", http.StatusInternalServerError)
}
w.Write([]byte("saved"))
```

OK:
```go
// return after replying
if err != nil {
    http.Error(w, "save failed", http.StatusInternalServerError)
    return
}
w.Write([]byte("saved"))
```

## Using the default HTTP client and server
The default `http.Client`/`http.Server` have no timeouts, so a slow peer can hang you indefinitely. Configure explicit timeouts in production.

NG:
```go
// no timeouts -> can block forever
resp, _ := http.Get(endpoint)
```

OK:
```go
// client with timeouts
client := &http.Client{
    Timeout: 10 * time.Second,
    Transport: &http.Transport{
        TLSHandshakeTimeout:   2 * time.Second,
        ResponseHeaderTimeout: 2 * time.Second,
    },
}
```

OK:
```go
// server with timeouts
s := &http.Server{
    Addr:              ":9090",
    ReadHeaderTimeout: time.Second,
    ReadTimeout:       time.Second,
    Handler:           http.TimeoutHandler(mux, 2*time.Second, "timeout"),
}
```
