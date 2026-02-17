# Python Code Generation Rules
## Clean Code + Architectural Patterns

A structured guide for generating clean, maintainable, production-grade Python backend code.
Optimized for testability, domain modeling, and long-term scalability.

---

# 1. Core Philosophy

- Explicit > implicit
- Simple > clever
- Composition > inheritance
- Exceptions > sentinel values
- Dataclasses > nested dictionaries
- Abstractions over implementation details
- Domain logic independent from infrastructure
- Architecture should slow the growth of complexity

---

# 2. Readability & Clarity

## General Principles

- Move complex expressions into helper functions.
- Avoid repeating logic — extract reusable abstractions.
- Prefer clarity over cleverness.
- Avoid `for/else` blocks.
- Reduce visual noise (avoid redundant slice bounds).
- Avoid negative strides and complex slicing expressions.
- Avoid unpacking into 4+ variables.

## Looping & Iteration

Prefer:
- `enumerate()` over `range(len(...))`
- `zip()` for parallel iteration
- `itertools.zip_longest()` for uneven iterables
- Catch-all unpacking:

```python
head, *rest = items
```

Avoid:
- Manual indexing when unpacking is clearer.
- Slicing with start + end + stride together.

---

# 3. Functions & API Design

## Return Values

- Never return `None` to indicate failure.
- Raise explicit domain exceptions.
- Use tuples only for small, obvious multiple returns.
- For structured returns, use `@dataclass`.

## Parameter Design

- Prefer keyword arguments for clarity.
- Use keyword-only args (`*`) to prevent ambiguity.
- Use positional-only args (`/`) to reduce API coupling.
- Never use mutable default values.
- Use `None` for dynamic defaults and document behavior.

---

# 4. Domain-Driven Design (DDD) Rules

## Domain Model First

- Model the business rules explicitly.
- Domain layer must contain:
  - Entities
  - Value Objects
  - Aggregates
  - Domain Exceptions
  - Domain Events

The domain model:
- Must not depend on Flask, SQLAlchemy, ORMs, or infrastructure.
- Must be pure Python.
- Must have fast unit tests.
- Must enforce invariants internally.

## Entities

- Have identity.
- Encapsulate business rules.
- Mutate through explicit methods.

## Value Objects

- Immutable.
- Equality based on value.
- Use `@dataclass(frozen=True)`.

## Aggregates

- Define consistency boundaries.
- Only modify data through aggregate roots.
- One aggregate = one repository.

---

# 5. Layered Architecture Rules

Follow strict layering:

```
Entrypoints (API / CLI)
        ↓
Service Layer
        ↓
Domain Model
        ↓
Repositories (Abstraction)
        ↓
Infrastructure (ORM / DB / External APIs)
```

Rules:

- Dependencies must only point inward.
- Domain layer cannot import infrastructure.
- Service layer orchestrates, but does not contain business rules.
- Infrastructure implements interfaces defined by inner layers.

Avoid "big ball of mud" architectures.

---

# 6. Repository Pattern

Purpose:
- Abstract persistent storage.
- Allow domain logic to be tested without database.

Rules:

- Define repository interface near the domain.
- Concrete repository implementations live in infrastructure.
- One repository per aggregate.
- Repository should return domain objects, not ORM models.

Avoid:
- Domain model depending on ORM.
- Exposing ORM objects outside infrastructure.

---

# 7. Unit of Work Pattern

Purpose:
- Manage transactional consistency.
- Define atomic operations.

Rules:

- A Unit of Work represents a single transaction boundary.
- It exposes repositories.
- It tracks new domain events.
- It commits or rolls back atomically.

Avoid:
- Ad-hoc commits scattered through code.
- Read-modify-write cycles without concurrency control.

---

# 8. Service Layer Pattern

Purpose:
- Define use cases.
- Orchestrate domain + repository interactions.

Rules:

- Service layer methods correspond to application use cases.
- Service layer coordinates:
  - Repositories
  - Unit of Work
  - Domain methods
- Service layer contains orchestration logic only.

Avoid:
- Business rules in API routes.
- Direct repository usage from entrypoints.

Entrypoints should be thin:

```python
@app.post("/allocate")
def allocate_endpoint():
    cmd = AllocateCommand(...)
    messagebus.handle(cmd)
```

---

# 9. Commands & Events

## Commands

- Represent intent ("Do this").
- Handled by a single handler.
- Trigger state changes.

## Events

- Represent something that happened.
- Can have multiple handlers.
- Trigger side effects.

Rules:

- Domain emits events.
- Handlers respond.
- Avoid implicit side effects.
- Keep handlers single-responsibility.

---

# 10. Message Bus Pattern

Purpose:
- Route commands and events to handlers.

Rules:

- System becomes message-driven.
- Handlers are granular and single-purpose.
- Handlers may publish new events.

Tradeoffs:

- Increased architectural complexity.
- Event schema duplication risk.
- Eventual consistency considerations.

---

# 11. CQRS (Command Query Responsibility Segregation)

Principle:
- Reads and writes have different responsibilities.

Rules:

- Domain model optimized for writes.
- Query layer may bypass domain for optimized reads.
- Do not force complex domain logic into read paths.

---

# 12. Dependency Injection

Rules:

- Inject repositories and UoW into services.
- Avoid global state.
- Avoid hard-coded infrastructure dependencies.
- Use manual DI (bootstrap script) instead of complex frameworks unless needed.

Example:

```python
def bootstrap():
    repo = SqlAlchemyRepository(...)
    uow = SqlAlchemyUnitOfWork(...)
    return MessageBus(uow=uow)
```

---

# 13. Testing Strategy (TDD-Oriented)

Follow the test pyramid:

1. Fast domain unit tests.
2. Service-layer tests with fake repositories.
3. Minimal integration tests.
4. Very few end-to-end tests.

Rules:

- Domain tests must not touch database.
- Use fakes instead of mocks where possible.
- Use mocks only at boundaries.
- Avoid patching internals.

Test at highest possible abstraction.

---

# 14. Concurrency & Data Integrity

Rules:

- Define consistency boundaries via aggregates.
- Use optimistic concurrency (version numbers) when possible.
- Use pessimistic locking only when necessary.
- Understand transaction isolation levels.

Avoid:
- Read-modify-write anti-patterns without safeguards.

---

# 15. Event-Driven Architecture Considerations

Be aware of:

- Event schema evolution.
- Message reliability (at-least-once vs at-most-once).
- Eventual consistency.
- Outbox pattern for reliability.

Document events clearly.
Version event schemas.

---

# 16. Performance Guidelines

- Profile before optimizing (`cProfile`).
- Avoid SELECT N+1 problems.
- Use `deque` for FIFO.
- Use `heapq` for priority queues.
- Use `bisect` for sorted lists.
- Use `Decimal(str_value)` for financial values.
- Avoid `list.pop(0)`.

---

# 17. Infrastructure & Adapters

Use Ports & Adapters (Hexagonal Architecture):

- Define ports (interfaces) in inner layers.
- Implement adapters in outer layers.
- Infrastructure is replaceable.

Adapters include:
- Web frameworks
- Databases
- Message brokers
- Email systems

They translate external input → commands.

---

# 18. Refactoring Strategy

When evolving an existing system:

- Introduce repository abstraction first.
- Extract service layer next.
- Isolate domain logic.
- Gradually introduce message bus.
- Avoid rewriting entire system at once.

Prefer incremental strangler-pattern refactoring.

---

# 19. When NOT to Use These Patterns

Avoid over-architecting:

- Small scripts
- Simple CRUD apps
- Throwaway prototypes

Introduce patterns only when complexity justifies them.

---

# 20. Summary Mental Model

Think in terms of:

- Use cases (Service Layer)
- Business rules (Domain Model)
- Consistency boundaries (Aggregates + UoW)
- Persistence abstraction (Repository)
- Intent vs fact (Commands vs Events)
- Dependency direction (Inward only)
- Replaceable infrastructure (Adapters)

Goal:

> Allow system complexity to grow more slowly than system size.
