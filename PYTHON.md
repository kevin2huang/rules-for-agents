# Python Code Generation Rules

This document defines how to generate clean, maintainable, production-grade Python code.

These rules are prescriptive. Follow them unless explicitly instructed otherwise.

---

# 0. Core Philosophy

- Explicit > implicit
- Simple > clever
- Composition > inheritance
- Exceptions > sentinel values
- Dataclasses > nested dictionaries
- Domain logic independent from infrastructure
- Dependencies must point inward
- Optimize for testability first
- Profile before optimizing
- Code should be readable without comments

---

# 1. Required Project Structure (For Non-Trivial Services)

```
service_name/
    domain/
        models.py
        events.py
        exceptions.py
        repository.py
    service_layer/
        commands.py
        handlers.py
        unit_of_work.py
        messagebus.py (optional)
    infrastructure/
        repository.py
        orm.py
        uow.py
    entrypoints/
        api.py
    bootstrap.py
```

Do not collapse layers unless the system is trivial.

---

# 2. Dependency Direction (Strict)

Allowed:

```
entrypoints → service_layer → domain
infrastructure → domain
bootstrap → everything
```

Forbidden:

- domain importing infrastructure
- domain importing ORMs
- domain importing web frameworks
- service_layer importing FastAPI/Flask
- returning ORM models outside infrastructure

If violated, refactor.

---

# 3. Code Readability & Documentation Policy

## 3.1 Naming Is the First Documentation

- Use descriptive names.
- Avoid abbreviations unless domain-standard.
- Functions describe behavior.
- Classes represent domain concepts.
- Avoid vague names like `data`, `obj`, `tmp`.

Prefer:

```python
allocate_order_to_batch()
```

Avoid:

```python
alloc()
```

---

## 3.2 Comments Must Explain “Why”, Not “What”

Do NOT restate obvious code.

Bad:

```python
# increment counter
counter += 1
```

Good:

```python
# Optimistic concurrency check to prevent lost updates
if order.version != expected_version:
    raise ConcurrencyError(...)
```

Use comments to explain:

- Business invariants
- Non-obvious algorithm decisions
- Performance tradeoffs
- Architectural constraints
- External system workarounds

---

## 3.3 Docstring Rules

Add docstrings when:

- The function/class is part of a public API
- The behavior is non-obvious
- Side effects exist
- Invariants or preconditions matter

Example:

```python
def allocate(self, batch: Batch) -> None:
    """
    Allocate this order to a batch.

    Raises:
        InvalidAllocation if batch cannot accept order.
    """
```

Do NOT add docstrings to trivial private helpers.

---

## 3.4 Keep Functions Small

If a function requires heavy commenting to explain behavior, refactor instead.

- One level of abstraction per function.
- Keep happy path visually clear.
- Use guard clauses early.
- Avoid deeply nested conditionals.

---

## 3.5 Public Interfaces Must Be Documented

Service handlers, repository interfaces, and domain entities must include minimal docstrings describing:

- Purpose
- Invariants
- Side effects

---

# 4. Domain Layer Rules (Pure Core)

Must be:

- Pure Python
- Framework-independent
- Fast to unit test

Must contain:

- Entities
- Value objects
- Domain events
- Domain exceptions
- Abstract repository interfaces

Entity rules:

- Have identity
- Enforce invariants internally
- Mutate via explicit methods
- Emit domain events when state changes

Value objects:

- Immutable
- Use `@dataclass(frozen=True)`
- Equality based on value

---

# 5. Service Layer Rules

Purpose: Orchestrate use cases.

Must:

- Load aggregates via repository
- Call domain methods
- Commit through Unit of Work
- Dispatch domain events

Must NOT:

- Contain business logic
- Perform SQL
- Parse HTTP
- Access ORM directly

Example:

```python
def allocate(cmd: Allocate, uow: AbstractUnitOfWork):
    with uow:
        order = uow.orders.get(cmd.order_id)
        order.allocate(cmd.batch_ref)
        uow.commit()
```

---

# 6. Repository Pattern

Interface (domain/repository.py):

```python
class AbstractRepository(ABC):
    @abstractmethod
    def add(self, entity): ...
    
    @abstractmethod
    def get(self, ref): ...
```

Rules:

- Repository returns domain objects
- Never return ORM models
- One repository per aggregate
- Domain depends only on abstract interface

---

# 7. Unit of Work Pattern

- Defines transaction boundary
- Exposes repositories
- Controls commit/rollback
- All writes inside `with uow:`

Never:

- Commit outside UoW
- Scatter commits across codebase

---

# 8. Commands & Events

Commands:

- Represent intent
- Handled by one handler
- Cause state change

Events:

- Represent facts
- Can have multiple handlers
- Trigger side effects

Domain emits events.
Service layer dispatches them.

---

# 9. Message Bus (Optional)

Introduce when:

- Multiple side effects exist
- Event-driven coordination is needed

Handlers must:

- Be small
- Be single-responsibility
- Avoid hidden coupling

---

# 10. CQRS Guidance

- Writes go through domain model.
- Reads may bypass domain for performance.
- Never embed complex invariants in read paths.

---

# 11. Entrypoints Must Be Thin

Entrypoints:

- Parse input
- Convert to command
- Call service layer
- Return response

Nothing else.

---

# 12. Bootstrap & Dependency Injection

All wiring happens in `bootstrap.py`.

- Inject repositories and UoW
- Avoid globals
- Avoid hidden singletons
- Prefer explicit construction

---

# 13. Readability Rules

- Move complex expressions into helper functions.
- Avoid repeated logic.
- Avoid clever slicing (especially negative stride).
- Avoid `for/else` loops.
- Prefer unpacking over indexing.
- Avoid unpacking into 4+ variables.

Prefer:

- `enumerate()` over `range(len())`
- `zip()` over manual indexing
- `itertools.zip_longest()` when lengths differ

---

# 14. Function & API Design

- Never return `None` to signal failure.
- Raise explicit exceptions.
- Use tuples only for small multi-returns.
- Use `@dataclass` for structured returns.

Parameter rules:

- Prefer keyword arguments.
- Use keyword-only args (`*`) when clarity matters.
- Use positional-only args (`/`) to reduce API coupling.
- Never use mutable default arguments.
- Use `None` for dynamic defaults.

---

# 15. Dictionary Rules

Prefer:

- `defaultdict` for accumulation
- `get()` for external dictionaries

Avoid:

- `setdefault()` when default construction is expensive
- Deeply nested dictionaries
- Implicit reliance on insertion order

---

# 16. Comprehensions & Generators

- Prefer list comprehensions over `map()` and `filter()`.
- Avoid more than 2 control clauses in a comprehension.
- Prefer generators for large/streaming inputs.
- Avoid iterating over the same iterator twice.
- Prefer `yield from` over manual nested loops.
- Avoid `send()` and `throw()` for control flow.

---

# 17. Concurrency & Consistency

Aggregates define consistency boundaries.

- Modify one aggregate per transaction
- Use optimistic concurrency (version numbers)
- Avoid unsafe read-modify-write cycles

Threads:

- Protect shared state with `threading.Lock`
- Use `Queue` for pipelines

Async:

- Prefer async/await for I/O concurrency
- Avoid blocking calls in coroutines

---

# 18. Performance Rules

- Profile before optimizing (`cProfile`)
- Use:
  - `deque` for FIFO
  - `heapq` for priority queues
  - `bisect` for sorted lists
- Avoid `list.pop(0)`
- Use `Decimal(str_value)` for financial calculations
- Use `memoryview` + `bytearray` for zero-copy I/O

---

# 19. Error Handling

- Keep `try` blocks small
- Use `else` to separate success paths
- Use context managers for resource control
- Raise domain-specific exceptions

---

# 20. Testing Strategy

Follow test pyramid:

1. Domain unit tests (no DB)
2. Service tests with fake repositories
3. Integration tests with real DB
4. Minimal E2E tests

Rules:

- Domain tests must not import infrastructure
- Prefer fakes over mocks
- Mock only external systems
- Use `unittest.TestCase`
- Use `assertEqual()` over `assert`
- Use `subTest()` for data-driven tests

---

# 21. Strict Anti-Patterns

Forbidden:

- Fat controllers
- Business logic in API routes
- Domain importing ORM
- Returning ORM models outside infrastructure
- Scattered commits
- Global mutable state
- Deep nested data structures as domain model
- Mutable default arguments
- Returning `None` to signal failure

---

# 22. Agent Mental Checklist

When implementing a backend feature:

1. Identify the use case.
2. Define a command.
3. Model domain behavior explicitly.
4. Use repository abstraction.
5. Wrap changes in Unit of Work.
6. Keep entrypoint thin.
7. Inject dependencies via bootstrap.
8. Write domain tests first.
9. Ensure layering rules are respected.
10. Refactor if complexity leaks across layers.

Goal:

> Complexity should grow slower than the system.
> The domain should remain pure.
> The system should stay testable and evolvable.
