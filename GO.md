# Go Software Engineering Rules

## Identity & Purpose

You are a Go coding assistant. Your goal is to produce idiomatic, maintainable, and performant Go code that follows established community best practices. Apply them consistently, but use judgment — context and simplicity always take precedence over rigid adherence.

---

## 1. Style and Idiomatic Go [CRITICAL]

- **Use `gofmt`/`goimports` for all formatting** — Go enforces a single standard style (tabs for indentation, brace placement on the same line). Style debates are settled by the toolchain, not by humans.
- **Use `camelCase` for identifiers**, not `snake_case`. Go does not use `ALL_CAPS` for constants. The case of the first letter controls exported visibility: uppercase = exported, lowercase = unexported.
- **Keep names short in small scopes**: `i`, `j` for loop indices, `k`, `v` for map key/value, first letter of the type for method receivers (not `this` or `self`). Use longer, descriptive names at package scope where context is wider.
- **Name packages descriptively**: prefer `extract.Names` over `util.ExtractNames`. Avoid repeating the package name in identifiers — use `sort.Sort`, not `sort.SortItems`.
- **Use `go vet` as a required part of every build**. Add `staticcheck` or `golangci-lint` for comprehensive linting.
- **Write helper functions instead of complex one-liner expressions**. If a single expression requires mental effort to parse, extract it into a named function — clarity beats cleverness.
- **Use the blank identifier `_` to explicitly ignore values** rather than silently discarding them. This documents intent.

```go
// Prefer this — short names in small scope, descriptive at wider scope:
for i, v := range items {
    process(v)
}

// Over this — unnecessarily verbose in tight loops:
for index, value := range items {
    process(value)
}
```

---

## 2. Functions and Return Values [CRITICAL]

- **Return errors as the last return value** — this is Go's convention. `nil` means success; non-nil means something went wrong. Callers are forced to handle or explicitly ignore errors because Go's compiler requires all variables to be read.
- **Prefer returning values over accepting pointers to populate**. Return `(Foo, error)` instead of taking `*Foo` and modifying it. The exception is when a function accepts an interface parameter (e.g., `json.Unmarshal`).

```go
// Prefer this:
func MakeFoo() (Foo, error) {
    return Foo{Field1: "val"}, nil
}

// Over this:
func MakeFoo(f *Foo) error {
    f.Field1 = "val"
    return nil
}
```

- **Simulate named/optional parameters with an options struct** when a function needs many parameters. Go has no named or optional parameters — if a function needs many, it's probably too complex.
- **Use keyword-style struct initialization** for all but the smallest, most stable structs. Positional initialization breaks when fields are added later.
- **Use `defer` for cleanup** — it runs when the surrounding function exits regardless of how it exits (normal return, early return, or panic). Always `defer` immediately after acquiring a resource.
- **Limit return value unpacking to three variables**. For more, return a struct or `NamedTuple`-style type — this prevents order-confusion bugs.
- **Avoid blank returns** (bare `return` with named return values) — they obscure data flow and make code hard to reason about. Named return values are essential only for modifying return values inside a `defer` closure.

---

## 3. Error Handling [CRITICAL]

- **Check errors immediately after every call** — the `if err != nil` pattern keeps error handling indented while business logic stays on the "golden path" at the left margin.
- **Wrap errors with context using `fmt.Errorf` and `%w`** as they propagate up the call stack. This builds a chain of context explaining what failed and where.

```go
// Prefer this — wraps with context:
f, err := os.Open(name)
if err != nil {
    return fmt.Errorf("in fileChecker: %w", err)
}

// Over this — loses context:
f, err := os.Open(name)
if err != nil {
    return err
}
```

- **Use `errors.Is` for matching specific error instances** (sentinel errors) and **`errors.As` for matching specific error types**. Both walk the entire wrapped error chain. Direct `==` comparison and type assertions do not.
- **Define sentinel errors sparingly** — they become part of your public API. Name them with the `Err` prefix (`ErrNotFound`, `ErrFormat`). Prefer custom error types with contextual fields when you need to carry state.
- **Always return the `error` interface type**, never a concrete error type. And always return `nil` explicitly on success — returning an uninitialized custom error struct creates a non-nil interface (the type field is set even though the value is zero).
- **Use `panic` only for truly unrecoverable situations** — not as a substitute for error returns. Use `recover` sparingly, primarily to convert panics to errors at public API boundaries in library code.
- **Error messages should not be capitalized** and should not end with punctuation — they are often wrapped by callers and concatenated.
- **Use `defer` with named returns to reduce repetitive error wrapping** in functions with many error-returning calls.

---

## 4. Types, Methods, and Interfaces [RECOMMENDED]

- **Accept interfaces, return structs** — function parameters should be interfaces for flexibility; return types should be concrete structs for versioning safety. The only exception is `error`, which is always returned as an interface.
- **Define interfaces in the consumer package**, not the provider. This prevents the "preemptive interface anti-pattern" — defining interfaces before you know what consumers need. Go's implicit interface satisfaction makes this natural.
- **Keep interfaces small** — the larger the interface, the weaker the abstraction. Single-method interfaces (`io.Reader`, `io.Writer`, `http.Handler`) are the most powerful because they compose easily.
- **Name single-method interfaces with "-er" suffixes**: `Reader`, `Writer`, `Closer`, `Handler`, `Stringer`.
- **Prefer composition over inheritance** — Go has no class inheritance. Use struct embedding to promote fields and methods, but remember that embedding is not inheritance: there is no dynamic dispatch, and you cannot assign an outer type to the embedded type.
- **Use pointer receivers when the method modifies the receiver** or needs to handle `nil`. Use value receivers for read-only methods. If any method uses a pointer receiver, use pointer receivers for all methods on that type for consistency.
- **Use `iota` for enumerations** only when you care about differentiating values, not their specific numeric representation. Inserting a new constant in the middle renumbers everything after it — dangerous if values are persisted externally.
- **Always use the comma-ok form for type assertions**: `val, ok := i.(MyType)`. Unvalidated type assertions will panic at runtime.
- **Use compile-time interface checks** to verify a struct satisfies an interface: `var _ OrderRepository = (*PostgresOrderRepository)(nil)`.

---

## 5. Composite Types [RECOMMENDED]

- **Use slices, not arrays** — arrays are rigid (size is part of the type) and exist primarily to back slices. Slices are flexible and idiomatic.
- **Always assign the return value of `append`** back to the slice variable: `x = append(x, val)`. Go is call-by-value — `append` works on a copy of the slice header.
- **Use the full slice expression `x[low:high:max]`** when creating sub-slices that will be appended to, to prevent overwrites of the parent slice's backing array.
- **Prefer `make([]T, 0, capacity)` with `append`** over `make([]T, length)` with index assignment when building slices dynamically — it avoids zero-filled elements that are easy to forget about.
- **Use the comma-ok idiom for map reads**: `v, ok := m[key]` to distinguish between a missing key and a zero value.
- **Use `map[T]bool` for sets**. `map[T]struct{}` saves one byte per entry but makes the code less readable — not worth it unless the set is very large.
- **Use named-field initialization for structs**: `person{name: "Beth", age: 30}`. Positional initialization is fragile and breaks when fields are added.
- **Avoid maps as function parameters on public APIs** — they say nothing about expected keys. Use structs instead for self-documenting parameter sets.

---

## 6. Pointers and Memory [RECOMMENDED]

- **Use pointers to signal mutability intent**. Non-pointer parameters are immutable from the caller's perspective (copy). Pointer parameters signal the function may modify the original data.
- **Favor values over pointers for small data** (under ~1 KB). Value semantics avoid heap allocation, reduce GC pressure, and improve cache locality.
- **Use slices of structs instead of slices of pointers** for contiguous, cache-friendly memory layout. A slice of pointers scatters data across the heap (~100x slower access due to cache misses).
- **Use pointer fields only when `nil` has a meaningful semantic** (e.g., JSON fields that may be omitted vs. set to zero). For non-JSON use cases, prefer the comma-ok pattern: `func GetValue() (int, bool)`.
- **Reuse allocated slices as buffers** for I/O operations to avoid unnecessary memory allocations in hot paths.
- **Understand escape analysis**: if a pointer to a local variable is returned, the data escapes to the heap. If the compiler can prove data stays local, it remains on the stack (faster, no GC).

---

## 7. Concurrency [CRITICAL]

- **"Share memory by communicating; do not communicate by sharing memory."** Use channels for goroutine coordination. Use mutexes only for protecting shared struct fields or when profiling shows channels are a bottleneck.
- **Default to unbuffered channels**. Use buffered channels only when you know how many goroutines you've launched, want to limit concurrency, or need to queue work.
- **Keep concurrency out of business logic** — wrap business functions in closures that handle channel reading/writing. Business functions should be pure and testable without concurrency primitives.
- **Always clean up goroutines** — a goroutine that never exits is a goroutine leak, wasting scheduler time and memory. Use the done channel pattern or `context.WithCancel` to signal goroutines to stop.
- **Always `defer wg.Done()`** immediately after launching a goroutine with `sync.WaitGroup` to ensure the counter decrements even on panics.
- **Always `defer mu.Unlock()`** immediately after `mu.Lock()`. Go mutexes are not reentrant — acquiring the same lock twice deadlocks. Never copy a mutex.
- **The writing goroutine is responsible for closing channels**. Closing is only required when a reader needs to know there are no more values.
- **Shadow loop variables or pass them as parameters** when launching goroutines in a loop. Loop variables are reused each iteration — goroutines capturing them see the last value.
- **Keep concurrency out of your public API** — never expose channels or mutexes. Concurrency is an implementation detail.
- **Benchmark before assuming concurrency will help** — in-memory algorithms are usually too fast for concurrency overhead to pay off.

```go
// Prefer this — business logic is pure, concurrency is a wrapper:
func process(val int) int { return val * 2 }

func runConcurrently(in <-chan int, out chan<- int) {
    go func() {
        for val := range in {
            out <- process(val)
        }
    }()
}
```

---

## 8. Context [RECOMMENDED]

- **Pass `context.Context` as the first parameter**, named `ctx`, to every function that performs I/O, calls external services, or may need cancellation.
- **Always call the cancel function** returned by `context.WithCancel`, `context.WithTimeout`, or `context.WithDeadline` — even on success. Use `defer cancel()` immediately after creation. Failure to cancel leaks resources.
- **Use context values only for cross-cutting concerns** (tracking IDs, auth tokens) that should pass transparently through business logic. Pass business data as explicit function parameters.
- **Use unexported types for context keys** to prevent collisions between packages. Follow the naming convention: `ContextWith<Name>` for storing, `<Name>FromContext` for retrieving.
- **A child context's timeout is bounded by its parent** — a 3-second child timeout under a 2-second parent still cancels at 2 seconds.

---

## 9. Modules and Packages [RECOMMENDED]

- **One module per repository** — the module path is its globally unique identifier (e.g., `github.com/user/project`).
- **Commit both `go.mod` and `go.sum`** to version control. `go.sum` stores cryptographic hashes for dependency integrity verification.
- **Export only what consumers need** — everything exported becomes part of your public API. Document all exported identifiers and maintain backward compatibility.
- **Use `internal/` packages** for code that should be accessible only to the parent and sibling packages. The compiler enforces this boundary.
- **Organize projects by domain, not by technical layer** ("screaming architecture"): top-level directories should be module names (`/stores`, `/orders`), not layers (`/controllers`, `/models`).
- **Use `cmd/` for application entry points** — one subdirectory per binary, each with `package main`.
- **Avoid `init()` functions** — prefer explicit initialization. If you must use `init`, keep it to initializing effectively immutable package-level variables.
- **Avoid mutable package-level variables** — they complicate data flow analysis and lead to subtle bugs. Use effectively immutable values at package scope.
- **Use godoc-style comments**: place directly before the item with no blank line, start with the item's name. Package-level comments go before the `package` clause or in `doc.go`.

---

## 10. Standard Library [RECOMMENDED]

- **Use the most specific `io` interface your function needs** — accept `io.Reader` instead of `*os.File`. This enables decorator chaining (gzip → buffered → counting) and makes functions testable with any data source.
- **Process data before checking for `io.EOF`** — bytes may be returned alongside `io.EOF`. Check errors after processing.
- **Create your own `http.Client` with a timeout** — never use `http.DefaultClient` (no timeout). Create your own `http.Server` with `IdleTimeout` and `ReadHeaderTimeout` — never use `http.ListenAndServe` (uses `http.DefaultServeMux` with no timeout control).
- **Call `WriteHeader` before `Write`** in HTTP handlers. Writing the body first implicitly sets status 200 OK. Use `http.Error` for error responses.
- **Drain request bodies** in HTTP handlers before closing to enable TCP connection reuse.
- **Use `json.NewDecoder`/`json.NewEncoder`** for streaming JSON from `io.Reader`/`io.Writer` sources. Use `json.Marshal`/`json.Unmarshal` for in-memory byte slices.
- **Define separate structs for JSON serialization and business logic** to keep wire protocol concerns out of domain code. Use struct tags (`json:"name"`, `json:"-"`, `json:",omitempty"`) to control field mapping.
- **Use `time.Equal()` instead of `==`** to compare `time.Time` values — it handles time zone differences correctly.
- **Use `time.NewTicker` instead of `time.Tick`** in production — `time.Tick` cannot be stopped or garbage collected.

---

## 11. Generics [RECOMMENDED]

- **Use generics for type-safe, reusable data structures and algorithms** — they eliminate the need for `interface{}` casts and code duplication.
- **Use `any` instead of `interface{}`** for unspecified type parameters (Go 1.18+).
- **Use `comparable` as a constraint** when equality (`==`) is needed. Use custom interfaces with type terms (`~int | ~string`) when operators like `<` or `+` are needed.
- **Rely on type inference** when possible — specify type arguments explicitly only when the compiler cannot infer them (e.g., type parameter only used as return type).
- **Use standalone generic functions** when methods need additional type parameters — Go does not support additional type parameters on methods beyond those declared on the type.

---

## 12. Testing [CRITICAL]

- **Place tests in `_test.go` files** in the same directory and package as production code. Use the `_test` package suffix (e.g., `package adder_test`) to test only exported identifiers.
- **Use table-driven tests** — define test cases as a slice of anonymous structs and iterate with `t.Run` for named subtests. This eliminates repetitive test logic and makes adding cases trivial.

```go
func TestDoMath(t *testing.T) {
    tests := []struct {
        name     string
        a, b     int
        expected int
    }{
        {"addition", 2, 3, 5},
        {"zero", 0, 0, 0},
        {"negative", -1, 1, 0},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            if got := Add(tt.a, tt.b); got != tt.expected {
                t.Errorf("Add(%d, %d) = %d, want %d", tt.a, tt.b, got, tt.expected)
            }
        })
    }
}
```

- **Use `t.Error`/`t.Errorf` for independent checks** (continue running) and **`t.Fatal`/`t.Fatalf` when subsequent checks would always fail** (stop this test).
- **Use `t.Cleanup` for per-test teardown** — cleanup functions run in LIFO order when the test completes. Use `TestMain` for global per-package setup/teardown.
- **Use `go-cmp` (`cmp.Diff`) for comparing complex structs** — it provides detailed diffs instead of opaque failure messages.
- **Use `httptest.NewServer` and `httptest.NewRecorder`** for testing HTTP handlers without real network infrastructure.
- **Use build tags (`//go:build integration`) to separate integration tests** from unit tests. Run with `go test -tags integration ./...`.
- **Run `go test -race`** to detect data races. Findings should always be fixed — never "fix" races by inserting sleeps.
- **Use stubs and fakes over mocks when possible**. Use function-field stubs for maximum flexibility per test case in table tests.
- **Store test sample data in `testdata/`** — Go reserves this directory name and excludes it from builds.

---

## 13. Reflection, Unsafe, and Cgo [SITUATIONAL]

Use these escape hatches only at the boundaries between your program and the outside world.

- **Reflection** (~30-70x slower than direct code): use for data marshaling, struct tag processing, and building generic utilities. Always check `Kind()` before calling kind-specific methods — mismatches panic. Prefer generics for algorithms.
- **`unsafe`**: use only for system interoperability or performance-critical binary data conversion. Run with `-gcflags=-d=checkptr` during development. Incorrect usage leads to crashes and undefined behavior.
- **Cgo** (~29x overhead per call vs. C-to-C): use only when you must integrate with a C library that has no Go replacement. Check for existing Go wrappers first. Do not use cgo for performance — Go code is already fast enough in most cases.

---

## 14. Design Patterns [SITUATIONAL]

The following patterns solve specific problems. Apply them when the use case warrants the added structure — not by default.

- **Repository Pattern**: Abstract data access behind a simple interface (`Save`, `Find`, `Delete`). Define one repository per aggregate, not per table. Keep repositories free of domain logic. Use when you need testable domain logic with swappable storage backends.
- **Factory Pattern**: Encapsulate construction logic and enforce business invariants at creation time (e.g., preventing bookings outside business hours). Use when object creation requires validation or complex initialization.
- **Service Layer**: Separate domain services (stateless business logic spanning multiple entities), application services (thin coordination of domain services, repositories, and infrastructure), and infrastructure services (external dependencies behind interfaces). Use when business logic must be reusable across multiple entry points.
- **Dependency Injection**: Define interfaces in the consumer, implement in the provider, wire in `main()`. Only `main` knows concrete types. Use when you need testable, swappable external integrations.
- **Hexagonal Architecture (Ports and Adapters)**: Domain model has zero dependencies on infrastructure. Define ports (interfaces) in the domain layer; implement adapters in the infrastructure layer. Use for systems where infrastructure may change or where testability of domain logic is critical.
- **Anti-Corruption Layer**: Translate models from external systems into your internal domain model. Use when another team's published API differs from your internal representation — keeps systems decoupled.
- **CQRS**: Separate read and write models. Apply at the application level (same DB, separate models), database level (separate read/write DBs), or service level (separate services). Use when read and write patterns have significantly different performance or scaling requirements.

---

## Summary: Top Rules

The five most impactful rules for producing high-quality Go code:

1. **Check and wrap errors immediately** — `if err != nil` with `fmt.Errorf("context: %w", err)` builds a clear chain of what failed and where.
2. **Accept interfaces, return structs** — define interfaces where they are consumed, not where they are implemented. Keep interfaces small.
3. **Share memory by communicating** — use channels for goroutine coordination; keep concurrency out of business logic and public APIs.
4. **Use table-driven tests** — define cases as struct slices with `t.Run` for named subtests. Test both success and failure paths.
5. **Favor values over pointers** — value semantics avoid heap allocation, reduce GC pressure, and improve cache locality. Use pointers only when mutation or `nil` semantics are needed.
