---
name: go-safety-critical
description: Use when writing safety-critical Go code, building reliable infrastructure, or applying NASA/JPL Power of 10 rules to Go. Covers bounded loops, error discipline, memory predictability, and static analysis for high-reliability systems.
---

# Safety-Critical Go (NASA Power of 10 Adapted)

NASA/JPL's "Power of 10" rules for safety-critical code, adapted for Go.

## Rule 1: Simple Control Flow

**No goto, no deep recursion, no panic for control flow.**

```go
// ❌ BAD: goto
func process() {
    goto cleanup  // Never use
cleanup:
    // ...
}

// ❌ BAD: unbounded recursion
func traverse(n *Node) {
    if n == nil { return }
    traverse(n.Left)   // Stack overflow risk
    traverse(n.Right)
}

// ✅ GOOD: iterative with explicit stack
func traverse(root *Node) {
    stack := []*Node{root}
    for len(stack) > 0 {
        n := stack[len(stack)-1]
        stack = stack[:len(stack)-1]
        if n == nil { continue }
        stack = append(stack, n.Left, n.Right)
    }
}

// ❌ BAD: panic for control flow
func find(items []Item, id string) Item {
    for _, item := range items {
        if item.ID == id { return item }
    }
    panic("not found")  // Never for expected conditions
}

// ✅ GOOD: error return
func find(items []Item, id string) (Item, error) {
    for _, item := range items {
        if item.ID == id { return item, nil }
    }
    return Item{}, ErrNotFound
}
```

## Rule 2: Bounded Loops

**All loops must have provable upper bounds.**

```go
const MaxIterations = 10000

// ❌ BAD: unbounded
func poll(ctx context.Context) error {
    for {  // How long does this run?
        if ready() { return nil }
        time.Sleep(time.Second)
    }
}

// ✅ GOOD: bounded with timeout
func poll(ctx context.Context) error {
    ctx, cancel := context.WithTimeout(ctx, 30*time.Second)
    defer cancel()

    ticker := time.NewTicker(time.Second)
    defer ticker.Stop()

    for i := 0; i < MaxIterations; i++ {
        select {
        case <-ctx.Done():
            return ctx.Err()
        case <-ticker.C:
            if ready() { return nil }
        }
    }
    return ErrTimeout
}

// ✅ GOOD: bounded retry
func withRetry(fn func() error) error {
    const maxRetries = 3
    var err error
    for i := 0; i < maxRetries; i++ {
        if err = fn(); err == nil {
            return nil
        }
        time.Sleep(time.Duration(i+1) * time.Second)
    }
    return fmt.Errorf("failed after %d retries: %w", maxRetries, err)
}
```

## Rule 3: Predictable Memory

**Pre-allocate, avoid unbounded growth, pool reusable objects.**

```go
// ❌ BAD: unbounded growth
func collect() []Item {
    var items []Item  // Grows unpredictably
    for item := range stream {
        items = append(items, item)
    }
    return items
}

// ✅ GOOD: pre-allocated with limit
const MaxItems = 10000

func collect() ([]Item, error) {
    items := make([]Item, 0, 256)  // Pre-allocate
    for item := range stream {
        if len(items) >= MaxItems {
            return nil, ErrTooManyItems
        }
        items = append(items, item)
    }
    return items, nil
}

// ✅ GOOD: sync.Pool for reuse
var bufferPool = sync.Pool{
    New: func() any { return make([]byte, 4096) },
}

func process(data []byte) {
    buf := bufferPool.Get().([]byte)
    defer bufferPool.Put(buf)
    // Use buf...
}
```

## Rule 4: Short Functions

**60 lines max. Single responsibility.**

```go
// ❌ BAD: 200-line function doing everything

// ✅ GOOD: decomposed
func handleRequest(ctx context.Context, req *Request) (*Response, error) {
    if err := validateRequest(req); err != nil {
        return nil, err
    }

    data, err := fetchData(ctx, req.ID)
    if err != nil {
        return nil, err
    }

    result := transformData(data, req.Options)
    return buildResponse(result), nil
}

func validateRequest(req *Request) error { /* 10 lines */ }
func fetchData(ctx context.Context, id string) (*Data, error) { /* 20 lines */ }
func transformData(data *Data, opts Options) *Result { /* 15 lines */ }
func buildResponse(result *Result) *Response { /* 10 lines */ }
```

## Rule 5: Defensive Checks

**Check inputs. Check errors. Assert invariants.**

```go
// ✅ GOOD: validate all inputs
func processUser(user *User, limit int) error {
    // Preconditions
    if user == nil {
        return errors.New("user cannot be nil")
    }
    if limit <= 0 || limit > MaxLimit {
        return fmt.Errorf("limit must be 1-%d, got %d", MaxLimit, limit)
    }
    if user.ID == "" {
        return errors.New("user ID required")
    }

    // Invariant check (use sparingly for "impossible" states)
    if user.CreatedAt.After(time.Now()) {
        // This should never happen - indicates data corruption
        return fmt.Errorf("invariant violation: user %s created in future", user.ID)
    }

    return doProcess(user, limit)
}

// ✅ GOOD: check EVERY error
data, err := fetchData(ctx, id)
if err != nil {
    return nil, fmt.Errorf("fetching data for %s: %w", id, err)
}

n, err := file.Write(data)
if err != nil {
    return fmt.Errorf("writing file: %w", err)
}
if n != len(data) {
    return fmt.Errorf("short write: %d/%d bytes", n, len(data))
}
```

## Rule 6: Minimal Scope

**Declare variables at point of use.**

```go
// ❌ BAD: declared far from use
func process() error {
    var user *User
    var data []byte
    var err error

    // 50 lines later...
    user, err = getUser()
}

// ✅ GOOD: minimal scope
func process() error {
    user, err := getUser()
    if err != nil {
        return err
    }

    data, err := fetchData(user.ID)
    if err != nil {
        return err
    }
    // data only visible from here
}

// ✅ GOOD: if-scoped variables
if conn, err := db.Connect(ctx); err != nil {
    return err
} else {
    defer conn.Close()
    // conn only visible in else block
}
```

## Rule 7: Validate Everything

**Check all inputs. Check all returns.**

```go
// ✅ GOOD: validate caller input
func CreateUser(ctx context.Context, name, email string) (*User, error) {
    // Validate all parameters
    if ctx == nil {
        return nil, errors.New("context required")
    }
    if name == "" {
        return nil, errors.New("name required")
    }
    if !isValidEmail(email) {
        return nil, fmt.Errorf("invalid email: %s", email)
    }

    user, err := db.Insert(ctx, name, email)
    if err != nil {
        return nil, fmt.Errorf("inserting user: %w", err)
    }

    // Validate return value
    if user.ID == "" {
        return nil, errors.New("database returned user without ID")
    }

    return user, nil
}
```

## Rule 8: No Magic (Build Tags Only)

**No code generation magic. Explicit over implicit.**

```go
// ❌ BAD: magic comments affecting behavior
//go:generate complex-code-generator

// ✅ ACCEPTABLE: simple build constraints
//go:build linux

// ❌ BAD: reflection-heavy code
func process(v any) {
    reflect.ValueOf(v).MethodByName("Do").Call(nil)
}

// ✅ GOOD: explicit interfaces
type Processor interface {
    Do() error
}

func process(p Processor) error {
    return p.Do()
}
```

## Rule 9: Simple Pointers

**Limit indirection. Prefer value semantics.**

```go
// ❌ BAD: pointer to pointer
func update(pp **Config) { ... }

// ❌ BAD: storing pointers in long-lived structures unnecessarily
type Cache struct {
    items map[string]*Item  // Pointer only if Item is large
}

// ✅ GOOD: value when small
type Cache struct {
    items map[string]Item  // Copy is fine for small structs
}

// ✅ GOOD: clear ownership
type Server struct {
    config Config      // Owned, copied at construction
    db     *Database   // Shared, documented lifecycle
}
```

## Rule 10: Static Analysis

**All warnings addressed. Multiple analyzers.**

```yaml
# .golangci.yml for safety-critical code
linters:
  enable:
    - errcheck      # All errors handled
    - govet         # Vet checks
    - staticcheck   # Advanced static analysis
    - gosec         # Security issues
    - bodyclose     # HTTP body closed
    - nilerr        # Nil error returns
    - exhaustive    # Switch exhaustiveness
    - unparam       # Unused parameters
    - prealloc      # Slice pre-allocation

linters-settings:
  govet:
    enable-all: true
  errcheck:
    check-blank: true
    check-type-assertions: true
  gosec:
    severity: medium
```

```bash
# CI pipeline
go vet ./...
golangci-lint run --config=.golangci-strict.yml
go test -race -coverprofile=coverage.out ./...
```

## Quick Checklist

- [ ] No goto, no unbounded recursion
- [ ] All loops have explicit bounds or timeouts
- [ ] Memory pre-allocated, growth bounded
- [ ] Functions under 60 lines
- [ ] All inputs validated, all errors checked
- [ ] Variables declared at smallest scope
- [ ] No reflection, minimal code generation
- [ ] Pointer indirection minimized
- [ ] golangci-lint passes with strict config
- [ ] Tests run with `-race` flag

## When to Apply

These rules are **mandatory** for:
- Infrastructure controlling physical systems
- Financial transaction processing
- Healthcare/medical systems
- Security-critical authentication

These rules are **recommended** for:
- Production APIs with SLAs
- Data pipelines with correctness requirements
- Long-running services

Sources: [NASA's Power of 10 Rules](https://www.perforce.com/blog/kw/NASA-rules-for-developing-safety-critical-code) | [Original Paper](https://spinroot.com/gerard/pdf/P10.pdf)
