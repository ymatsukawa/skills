# Avoid guidance of "Coding"

## Unintended variable shadowing
Avoid shadowing. It makes unintended nil bug.

NG:
```go
var conn *sql.DB
if tracing {
    conn, err := newTracedConn() // create conn for internal
    ...
}
// external conn is nil
```

OK-1:
```go
// Using temporary variable
var conn *sql.DB
if tracing {
    c, err := newTracedConn()
    if err != nil {
        return err
    }
    conn = c
}
```

OK-2:
```go
// Use '=' for internal declaration
var conn *sql.DB
var err error
if tracing {
    conn, err = newTracedConn()
    ...
}
```

## Unnecessary nested code
Keep the happy path left-aligned. Return early on errors instead of nesting else blocks; it improves readability.

NG:
```go
// deeply nested if/else
if base == "" {
    return "", errors.New("base is empty")
} else {
    if rel == "" {
        return "", errors.New("rel is empty")
    } else {
        joined, err := joinPath(base, rel)
        if err != nil {
            return "", err
        } else {
            return joined, nil
        }
    }
}
```

OK:
```go
// flat: return early on each error
if base == "" {
    return "", errors.New("base is empty")
}
if rel == "" {
    return "", errors.New("rel is empty")
}
joined, err := joinPath(base, rel)
if err != nil {
    return "", err
}
return joined, nil
```

## Misusing init functions
Avoid init functions for fallible setup; they cannot return errors (forcing panic), run before main, and hurt testability. Prefer explicit constructors.

NG:
```go
// init can't return errors -> must panic, hard to test
var conn *sql.DB
func init() {
    c, err := sql.Open("postgres", os.Getenv("DB_URL"))
    if err != nil {
        log.Panic(err)
    }
    conn = c
}
```

OK:
```go
// explicit constructor returns error to the caller
func openConn(dbURL string) (*sql.DB, error) {
    conn, err := sql.Open("postgres", dbURL)
    if err != nil {
        return nil, err
    }
    return conn, nil
}
```

## Interface pollution
Don't create interfaces speculatively. Add an abstraction only when a real need (decoupling, restricting behavior) appears; otherwise depend on the concrete type.

NG:
```go
// premature interface with a single producer-side implementation
type articleSaver interface {
    SaveArticle(Article) error
}
type ArticleService struct {
    saver articleSaver
}
```

OK:
```go
// discover interfaces; here just use the concrete type until abstraction is needed
type ArticleService struct {
    repo Repo // concrete
}
// or restrict behavior by accepting only what you use:
func pipeSourceToSink(src io.Reader, sink io.Writer) error {
    _, err := io.Copy(sink, src)
    return err
}
```

## Interface on the producer side
Interfaces should be defined where they're consumed, not exported by the producer. Let the client declare the minimal interface it needs.

NG:
```go
// producer package exports a fat interface
package repo
type ArticleStorage interface {
    SaveArticle(Article) error
    FindArticle(id string) (Article, error)
    ListArticles() ([]Article, error)
    // ...many more
}
```

OK:
```go
// consumer package declares the minimal interface it actually uses
package feed
type articlesLister interface {
    ListArticles() ([]repo.Article, error)
}
```

## Overusing any
`any` removes compile-time type safety and meaning. Prefer concrete types or generics; reserve `any` for genuinely type-agnostic cases.

NG:
```go
// loses type info; callers must type-assert
func (r *Repo) Find(id string) (any, error) { ... }
func (r *Repo) Save(id string, v any) error { ... }
```

OK:
```go
// explicit, type-safe methods per type
func (r *Repo) FindArticle(id string) (Article, error) { ... }
func (r *Repo) SaveArticle(id string, a Article) error { ... }
```

## Misusing / overusing generics
Use generics to remove boilerplate over multiple types, not where a concrete type or simple interface suffices.

NG:
```go
// hand-written type switch per map type
func collectKeys(m any) ([]any, error) {
    switch t := m.(type) {
    case map[string]float64:
        // ...
    case map[int]bool:
        // ...
    }
}
```

OK:
```go
// one generic function, type-safe for any map
func collectKeys[K comparable, V any](m map[K]V) []K {
    var ks []K
    for k := range m {
        ks = append(ks, k)
    }
    return ks
}
```

## Type embedding pitfalls
Embed only to promote methods you intentionally want as part of the API. Embedding can leak unwanted methods (e.g. exporting an embedded `sync.Mutex`); use a named field when you don't want promotion.

NG:
```go
// embedded Mutex promotes Lock/Unlock into MemCache's public API
type MemCache struct {
    sync.Mutex
    items map[string]int
}
```

OK:
```go
// unexported named field keeps the lock internal
type MemCache struct {
    mu    sync.Mutex
    items map[string]int
}
func (c *MemCache) Lookup(key string) (int, bool) {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.items[key], true
}
```

## Not using the functional options pattern
For optional/expanding configuration, avoid a bare config struct or a verbose builder. Functional options give a clean, extensible API with sane defaults and validation.

NG:
```go
// config struct: can't distinguish "unset" from zero; or a clunky builder
func NewClient(host string, cfg Config) {}
NewClient("db.local", Config{}) // Timeout 0? default? unclear
```

OK:
```go
// functional options: defaults + validation, easy to extend
type Option func(*options) error
func WithTimeout(d time.Duration) Option {
    return func(o *options) error {
        if d < 0 { return errors.New("timeout should be positive") }
        o.timeout = &d
        return nil
    }
}
func NewClient(host string, opts ...Option) (*Client, error) {
    var o options
    for _, opt := range opts {
        if err := opt(&o); err != nil {
            return nil, err
        }
    }
    return &Client{host: host, opts: o}, nil
}
NewClient("db.local", WithTimeout(3*time.Second))
```

## Creating generic utility packages
Avoid meaningless catch-all package names like `util`, `common`, or `base`. Name packages after what they provide so call sites read well.

NG:
```go
// vague: util.NewIDSet, util.Values -> no meaning at call site
package util
```

OK:
```go
// expressive: idset.New(...) reads clearly
package idset
type Set map[string]struct{}
func New(...string) Set { return Set{} }
func (s Set) Values() []string { return nil }
```
