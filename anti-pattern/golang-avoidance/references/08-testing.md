# Avoid guidance of "Testing"

## Not categorizing tests (build tags, -short, env vars)
Classify tests so slow/integration tests can be excluded from fast runs. Use build tags, `testing.Short()`, or env vars to gate them.

NG:
```go
// Slow integration test always runs with the rest
func TestSaveOrder(t *testing.T) { /* hits real DB */ }
```

OK-1:
```go
//go:build integration   // run only with: go test -tags=integration
package repo
```

OK-2:
```go
func TestBulkImport(t *testing.T) {
	if testing.Short() { // skip with: go test -short
		t.Skip("skipping long-running test")
	}
}
```

OK-3:
```go
if os.Getenv("INTEGRATION") != "true" { // gate via env var
	t.Skip("skipping integration test")
}
```

## Not using table-driven tests
Repeating near-identical test functions is verbose and hard to extend. Use a map/slice of cases with subtests via `t.Run`.

NG:
```go
// One function per case, all duplicated boilerplate
func TestTrimTrailingSpaces_Empty(t *testing.T) { /* ... */ }
func TestTrimTrailingSpaces_EndingWithSpace(t *testing.T) { /* ... */ }
```

OK:
```go
tests := map[string]struct{ input, expected string }{
	`empty`:             {"", ""},
	`ending with space`: {"a ", "a"},
}
for name, tc := range tests {
	tc := tc
	t.Run(name, func(t *testing.T) {
		if got := trimTrailingSpaces(tc.input); got != tc.expected {
			t.Errorf("got: %s, expected: %s", got, tc.expected)
		}
	})
}
```

## Sleeping in unit tests
`time.Sleep` makes tests flaky and slow when waiting on async work. Use retries with assertions or synchronize via channels.

NG:
```go
task := h.pickTopTask(7)
time.Sleep(10 * time.Millisecond) // flaky guess at timing
published := sink.Get()
```

OK-1:
```go
// retry assertion until it passes or attempts exhausted
func eventually(t *testing.T, cond func() bool, attempts int, wait time.Duration) {
	for i := 0; i < attempts; i++ {
		if cond() { return }
		time.Sleep(wait)
	}
	t.Fail()
}
```

OK-2:
```go
sink := sinkMock{ch: make(chan []Task)}
h.pickTopTask(7)
if v := len(<-sink.ch); v != 3 { // block on channel, no sleep
	t.Fatalf("expected 3, got %d", v)
}
```

## Not dealing with the time API in tests
Relying on `time.Now()` inside the code makes time-dependent tests flaky. Inject the clock or pass time as an argument.

NG:
```go
func (l *Log) ExpireOlderThan(age time.Duration) {
	cutoff := time.Now().Add(-age) // hidden dependency on real clock
	// ...
}
```

OK-1:
```go
type Log struct{ now func() time.Time } // inject clock
func (l *Log) ExpireOlderThan(age time.Duration) {
	cutoff := l.now().Add(-age)
}
// test: Log{now: func() time.Time { return parseTime(t, "...") }}
```

OK-2:
```go
// Pass time as a parameter; test supplies a fixed instant
func (l *Log) ExpireOlderThan(now time.Time, age time.Duration) {
	cutoff := now.Add(-age)
}
```

## Not using testing utility packages (httptest, iotest)
Stdlib test helpers replace hand-rolled mocks. Use `httptest` for HTTP handlers/clients and `iotest` for `io.Reader` testing.

NG:
```go
// spins up a real listener and hand-rolls the client plumbing
ln, _ := net.Listen("tcp", ":9090")
go http.Serve(ln, PingHandler)
resp, _ := http.Get("http://localhost:9090/ping")
```

OK-1:
```go
req := httptest.NewRequest(http.MethodGet, "http://localhost", strings.NewReader("ping"))
w := httptest.NewRecorder()       // test a handler without a real server
PingHandler(w, req)
srv := httptest.NewServer(http.HandlerFunc(/* ... */)) // test a client
```

OK-2:
```go
iotest.TestReader(myReader, []byte("bdfhj"))             // validate Read behavior
err := parse(iotest.TimeoutReader(strings.NewReader(s))) // simulate read errors
```

## Writing inaccurate benchmarks
Setup, optimizations, and timer scope can corrupt results. Reset/stop the timer around setup, and prevent dead-code elimination.

NG:
```go
func BenchmarkChecksum(b *testing.B) {
	for i := 0; i < b.N; i++ {
		checksum(uint64(i)) // result unused -> may be optimized away
	}
}
```

OK-1:
```go
// Exclude setup from timing
loadFixtures()
b.ResetTimer()
for i := 0; i < b.N; i++ { functionUnderTest() }
// or per-iteration: b.StopTimer(); setup(); b.StartTimer(); fn()
```

OK-2:
```go
var sink uint64
func BenchmarkChecksum(b *testing.B) {
	var v uint64
	for i := 0; i < b.N; i++ { v = checksum(uint64(i)) }
	sink = v // assign to package var so compiler can't elide it
}
```

## Not exploring all the Go testing features
Leverage built-in helpers: black-box `_test` packages, `TestMain`/`t.Cleanup` for setup-teardown, and utility functions that fail the test directly.

NG:
```go
// Test in same package; factory returns error to assert in every test
article, err := buildArticle("draft")
if err != nil { t.Fatal(err) }
```

OK-1:
```go
package ratelimit_test // black-box test: only the exported API
func TestAllow(t *testing.T) { ratelimit.Allow() }
```

OK-2:
```go
func TestMain(m *testing.M) { // package-level setup/teardown
	setupPostgres(); code := m.Run(); teardownPostgres(); os.Exit(code)
}
db := openTestDB(t, dsn) // t.Cleanup registers teardown
```

OK-3:
```go
// Utility helper fails the test itself; callers stay clean
func newArticle(t *testing.T, title string) Article {
	a, err := buildArticle(title)
	if err != nil { t.Fatal(err) }
	return a
}
```
