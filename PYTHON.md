# Python Code Generation Rules

A structured guide for generating clean, maintainable, backend-oriented Python code.  
Inspired by the Zen of Python, PEP 8, and modern backend engineering practices.

---

# 1. Core Philosophy

- Readability counts.
- Explicit > implicit.
- Simple > clever.
- Composition > inheritance.
- Exceptions > sentinel values.
- Clarity > micro-optimization.
- Design for maintainability and evolution.

Code is read far more often than it is written.

---

# 2. Style & Formatting (PEP 8)

## Naming

- `snake_case` → variables & functions
- `PascalCase` → classes
- `UPPER_CASE` → constants
- Prefix internal helpers with `_`
- Avoid single-letter names outside small scopes

## Formatting

- 4 spaces indentation
- ≤ 88 character line length
- One statement per line
- Logical spacing with blank lines
- Use auto-formatters (do not manually align)

## Imports Order

1. Standard library
2. Third-party packages
3. Local modules

Avoid:

- `from module import *`
- Circular imports

---

# 3. Readability & Structure

- Move complex expressions into helper functions.
- Avoid repeating logic — extract abstractions.
- Avoid deeply nested code.
- Avoid overly clever slicing (especially negative strides).
- Avoid `for/else` unless necessary.
- Reduce visual noise (no redundant slice bounds like `[:len(x)]`).

---

# 4. Looping & Iteration

Prefer:

- `enumerate()` over `range(len(...))`
- `zip()` for parallel iteration
- `itertools.zip_longest()` for uneven iterables
- Unpacking over indexing
- Catch-all unpacking:

```python
head, *rest = items
```

Avoid:

- Unpacking into 4+ variables
- Manual index tracking
- Assignment expressions (`:=`) outside simple conditions

---

# 5. Functions & Parameters

## Design Principles

- One responsibility per function.
- Prefer 5–20 line functions.
- Clear inputs → clear outputs.
- No hidden side effects.

## Return Values

- Do not return `None` to indicate failure.
- Raise explicit exceptions.
- Use tuples for small multi-returns.
- Use `dataclass` for structured returns.

## Parameter Design

- Prefer keyword arguments for clarity.
- Use keyword-only arguments (`*`) to prevent ambiguity.
- Use positional-only (`/`) to reduce API coupling.
- Never use mutable default arguments.
- Use `None` as sentinel for dynamic defaults.

Example:

```python
def create_user(name: str, *, metadata: dict | None = None):
    if metadata is None:
        metadata = {}
```

Avoid extending APIs via `*args`.

---

# 6. Dictionaries

Preferred order for missing keys:

1. `defaultdict` for accumulation
2. `dict.get()` for safe access
3. `setdefault()` only if default creation is cheap

Avoid:

- Expensive `setdefault()`
- Deeply nested dictionary structures
- Implicit reliance on insertion order

---

# 7. Comprehensions & Generators

## Comprehensions

- Prefer list comprehensions over `map()` and `filter()`.
- Avoid >2 control clauses.
- Avoid overly nested expressions.

## Generators

- Prefer generators for streaming/large data.
- Use `yield from` over manual loops.
- Be aware iterators exhaust.
- Avoid `send()` and `throw()` for control flow.

Generators > accumulating large lists.

---

# 8. Data Modeling

Use:

- `dataclass` for structured data.
- `Enum` for closed value sets.
- `namedtuple` for internal lightweight immutables.
- `__call__` classes instead of stateful closures.

Avoid:

- Nested dict/tuple structures for complex state.
- Overusing `@property`.
- Using `namedtuple` in public APIs that may evolve.

If internal state grows complex → refactor into multiple classes.

---

# 9. Object-Oriented Practices

- Prefer composition over inheritance.
- Avoid deep class hierarchies.
- Use `super()` (zero-argument form).
- Avoid multiple inheritance with instance state.
- Prefer public attributes over getters/setters.
- Use `@property` only for fast, unsurprising behavior.

Advanced features only when justified:

- Prefer class decorators over metaclasses.
- Prefer `__init_subclass__` over metaclasses.
- Avoid complex `__getattribute__`.

---

# 10. Concurrency & Parallelism

## General Principles

- Never allow unsynchronized shared mutation.
- Prefer message passing over shared state.
- Always add timeouts to external calls.

## Threads

- Use `threading.Lock` for shared state.
- Use `Queue` for pipelines/backpressure.
- Prefer fixed worker pools.

## AsyncIO

- Use `async/await` for I/O concurrency.
- Never block inside coroutines.
- Use `asyncio.run(debug=True)` during development.
- Use `run_in_executor()` for gradual migration.

## Multiprocessing

- Prefer `ProcessPoolExecutor`.
- Avoid advanced primitives unless required.

---

# 11. Performance & Scaling

- Profile before optimizing (`cProfile`).
- Use:
  - `deque` for FIFO
  - `heapq` for priority queues
  - `bisect` for sorted search
- Avoid `list.pop(0)`
- Use `memoryview` / `bytearray` for zero-copy I/O.
- Use `Decimal(str_value)` for financial precision.

Maintainability > micro-optimization.

---

# 12. Time & Serialization

- Store time in UTC.
- Convert to local time only at presentation.
- Use timezone-aware `datetime`.
- Avoid using `time` for timezone conversion.
- Use `pickle` only between trusted programs.

---

# 13. Subprocess Management

- Use `subprocess.run()` for simple cases.
- Use `Popen` for pipelines.
- Always specify timeouts for `communicate()`.

---

# 14. Testing

- Write unit tests for pure logic.
- Write integration tests for I/O.
- Prefer dependency injection for testability.
- Mock only at system boundaries.
- Use `subTest()` for parameterized testing.
- Implement `__repr__` for debugging clarity.

---

# 15. Error Handling & Observability

- Keep `try` blocks small.
- Catch specific exceptions.
- Use `else` on `try` to separate success path.
- Add context when re-raising.
- Never silently swallow exceptions.
- Log structured context (include IDs).
- Use `breakpoint()` for development.
- Use `tracemalloc` for memory diagnostics.

Errors should never pass silently.

---

# 16. Backend Architecture Principles

- Separate core logic from I/O.
- Keep orchestration thin.
- Inject dependencies explicitly.
- Avoid global state.
- Make operations idempotent when possible.
- Design APIs for clarity and evolution.
- Validate input at boundaries.

---

# Final Checklist

Before committing:

- Is this readable without explanation?
- Is there a simpler solution?
- Does it follow naming conventions?
- Are exceptions explicit and narrow?
- Is core logic separated from I/O?
- Is concurrency safe?
- Would another engineer understand this immediately?

---

Clarity > Cleverness  
Simplicity > Abstraction  
Explicit > Implicit  
Composition > Inheritance  
Maintainability > Micro-Optimization
