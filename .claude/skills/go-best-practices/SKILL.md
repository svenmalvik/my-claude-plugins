---
name: go-best-practices
description: Use when writing Go code, reviewing Go implementations, or making architectural decisions in Go projects. Covers idiomatic patterns, error handling, package design, and common pitfalls.
---

# Go Best Practices

## Core Principles

Go favors simplicity, readability, and explicit error handling over clever abstractions.

## Package Design

```
project/
  cmd/myapp/main.go     # Entry point only
  internal/             # Private packages
  pkg/                  # Public packages (if library)
```

**Package naming:**
- Lowercase, single word: `http`, `user`, `config`
- Avoid `util`, `common`, `helpers` - be specific
- Package name should not repeat in exported names: `http.Server` not `http.HTTPServer`

## Error Handling

```go
// Return errors, don't panic
func Load(path string) (*Config, error) {
    data, err := os.ReadFile(path)
    if err != nil {
        return nil, fmt.Errorf("reading config %s: %w", path, err)
    }
    // ...
}

// Wrap errors with context using %w
if err != nil {
    return fmt.Errorf("connecting to %s: %w", addr, err)
}

// Check specific errors
if errors.Is(err, os.ErrNotExist) {
    // handle missing file
}
```

**Anti-patterns:**
- `if err != nil { return err }` without context
- Ignoring errors with `_`
- Using `panic` for recoverable errors

## Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Exported | MixedCaps | `UserService` |
| Unexported | mixedCaps | `userCache` |
| Interfaces | -er suffix | `Reader`, `Stringer` |
| Acronyms | ALL CAPS | `HTTPServer`, `userID` |
| Getters | No Get prefix | `u.Name()` not `u.GetName()` |

## Interface Design

```go
// Small interfaces at point of use
type Reader interface {
    Read(p []byte) (n int, err error)
}

// Accept interfaces, return structs
func Process(r io.Reader) (*Result, error) { ... }

// Define interfaces where consumed, not implemented
```

## Struct Initialization

```go
// Use composite literals
cfg := &Config{
    Host:    "localhost",
    Port:    8080,
    Timeout: 30 * time.Second,
}

// Constructor for validation/defaults
func NewServer(addr string, opts ...Option) (*Server, error) {
    s := &Server{addr: addr}
    for _, opt := range opts {
        opt(s)
    }
    return s, s.validate()
}
```

## Concurrency

```go
// Use context for cancellation
func Fetch(ctx context.Context, url string) (*Response, error) {
    req, err := http.NewRequestWithContext(ctx, "GET", url, nil)
    // ...
}

// Channels for communication, mutexes for state
type Cache struct {
    mu    sync.RWMutex
    items map[string]Item
}

// Always handle goroutine lifecycle
go func() {
    defer wg.Done()
    // work
}()
```

## Testing

```go
// Table-driven tests
func TestParse(t *testing.T) {
    tests := []struct {
        name    string
        input   string
        want    int
        wantErr bool
    }{
        {"valid", "42", 42, false},
        {"invalid", "abc", 0, true},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := Parse(tt.input)
            if (err != nil) != tt.wantErr {
                t.Errorf("Parse() error = %v, wantErr %v", err, tt.wantErr)
                return
            }
            if got != tt.want {
                t.Errorf("Parse() = %v, want %v", got, tt.want)
            }
        })
    }
}
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Naked returns | Always name what you return |
| `interface{}` overuse | Use generics or specific types |
| Premature abstraction | Write concrete code first |
| Deep package nesting | Keep it flat, max 2-3 levels |
| Global state | Pass dependencies explicitly |
| Ignoring `go vet`/`staticcheck` | Run linters in CI |

## Generics (Go 1.18+)

```go
// Type constraints
func Map[T, U any](items []T, fn func(T) U) []U {
    result := make([]U, len(items))
    for i, item := range items {
        result[i] = fn(item)
    }
    return result
}

// Union constraints
type Number interface {
    int | int64 | float64
}

func Sum[T Number](values []T) T {
    var total T
    for _, v := range values {
        total += v
    }
    return total
}

// Generic data structures
type Stack[T any] struct {
    items []T
}

func (s *Stack[T]) Push(item T) { s.items = append(s.items, item) }
func (s *Stack[T]) Pop() (T, bool) {
    if len(s.items) == 0 {
        var zero T
        return zero, false
    }
    item := s.items[len(s.items)-1]
    s.items = s.items[:len(s.items)-1]
    return item, true
}
```

## Profiling and Benchmarks

```go
// Benchmark function
func BenchmarkProcess(b *testing.B) {
    data := setupTestData()
    b.ResetTimer()  // Exclude setup time
    for i := 0; i < b.N; i++ {
        Process(data)
    }
}

// Run with: go test -bench=. -benchmem
// Output: BenchmarkProcess-8  1000  1234567 ns/op  4096 B/op  10 allocs/op

// CPU profiling
// go test -cpuprofile=cpu.prof -bench=.
// go tool pprof cpu.prof

// Memory profiling
// go test -memprofile=mem.prof -bench=.
// go tool pprof mem.prof
```

## Linting (golangci-lint)

```yaml
# .golangci.yml
linters:
  enable:
    - errcheck
    - govet
    - staticcheck
    - unused
    - gosimple
    - ineffassign
    - misspell
    - gofmt
    - goimports

linters-settings:
  errcheck:
    check-blank: true
  govet:
    enable-all: true
```

Run: `golangci-lint run ./...`

## Race Detection and Fuzzing

```go
// Run tests with race detector
// go test -race ./...

// Fuzz testing (Go 1.18+)
func FuzzParse(f *testing.F) {
    // Seed corpus
    f.Add("valid input")
    f.Add("")
    f.Add("edge case")

    f.Fuzz(func(t *testing.T, input string) {
        result, err := Parse(input)
        if err == nil && result == nil {
            t.Error("nil result without error")
        }
    })
}

// Run: go test -fuzz=FuzzParse -fuzztime=30s
```

## Quick Checklist

- [ ] Errors wrapped with context (`%w`)
- [ ] Interfaces defined at consumer
- [ ] No exported fields unless necessary
- [ ] Context passed through call chain
- [ ] Resources cleaned up with `defer`
- [ ] Tests use `t.Run` subtests
- [ ] Tests run with `-race` flag
- [ ] golangci-lint passes
- [ ] Benchmarks for hot paths
