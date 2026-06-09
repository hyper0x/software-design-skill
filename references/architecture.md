# Architecture Principles (Monolith)

Architecture-level principles for structuring monolith applications. These principles help manage complexity without the overhead of distributed systems patterns.

---

## Layered Architecture

> Origin: Common pattern, formalized in patterns of enterprise application architecture

**Organize code into horizontal layers with clear responsibilities.**

### The classic three layers

```
Presentation Layer (UI / API controllers)
    ↓
Business Logic Layer (domain services, business rules)
    ↓
Data Access Layer (repositories, database access)
```

### Rules

- Each layer only depends on the layer below it
- No layer should skip to a non-adjacent layer (e.g., UI directly accessing the database)
- Each layer has a well-defined responsibility

### Gotchas

- Layered Architecture is not "three layers always" — the number of layers depends on complexity. A simple CRUD app might only need two layers
- The danger: "sinkhole anti-pattern" — layers that do nothing but forward calls. If a layer adds no value, remove it
- In monoliths, strict layering can be relaxed for cross-cutting concerns (logging, auth) — those can touch all layers

---

## Dependency Rule

> Origin: Robert C. Martin, *Clean Architecture* (2017)

**Source code dependencies must point inward, toward higher-level policies.**

### The concentric circles

```
Frameworks / Drivers     (outermost — details)
    ↓
Interface Adapters       (controllers, presenters)
    ↓
Application Business Rules (use cases)
    ↓
Enterprise Business Rules (entities — innermost, highest-level)
```

### What this means in practice

- Business logic (inner circles) must **never** import anything from frameworks or databases (outer circles)
- Business logic should be pure — no framework annotations, no database imports, no HTTP concerns
- All dependencies cross the boundary **inward**

### Before/After

**Violation:**
```python
from django.db import models  # framework dependency in business logic

class Order(models.Model):
    def calculate_total(self):
        ...
```

**Clean:**
```python
# business/order.py — pure Python, no framework imports
class Order:
    def __init__(self, items):
        self.items = items

    def calculate_total(self):
        return sum(item.price for item in self.items)
```

### Gotchas

- Dependency Rule is violated the moment a domain model references a framework type
- The rule applies to **source code dependencies**, not runtime flow. At runtime, control flows from outer to inner and back
- In monoliths, you don't need the full Clean Architecture machinery — just the core insight: business logic should not depend on frameworks

---

## Ports & Adapters (Hexagonal Architecture)

> Origin: Alistair Cockburn, 2005

**Core business logic communicates with the outside world through interfaces (ports), with concrete implementations (adapters) plugged in.**

### Structure

```
                    ┌─────────────┐
                    │  Adapter    │  ← e.g., MySQLRepository
                    │  (impl)     │
                    └──────┬──────┘
                           │ implements
                    ┌──────┴──────┐
   HTTP Request ───▶│   Port      │  ← interface (e.g., OrderRepository)
                    │  (interface)│
                    └──────┬──────┘
                           │ called by
                    ┌──────┴──────┐
                    │  Core       │  ← business logic, pure
                    │  (domain)   │
                    └─────────────┘
```

### What to abstract

| Abstract (create a port) | Don't abstract |
|--------------------------|----------------|
| Database | Standard library types |
| External APIs | Value objects (Money, Date) |
| File system | Language built-ins |
| Message queues | Third-party SDKs you own |

### Gotchas

- Ports & Adapters doesn't mean "interface for everything" — only for **external dependencies** that could change
- The port (interface) should be **defined by the core**, not by the adapter. The core says "I need a repository that can save orders" — the adapter implements it
- In a monolith, you don't need a separate "infrastructure" project. Keep adapters in a subdirectory

---

## DDD Lightweight

A simplified take on Domain-Driven Design for monoliths — keeping the valuable parts without the full framework.

### 1. Ubiquitous Language

> Origin: Eric Evans, *Domain-Driven Design* (2003)

**Use the same terms in code as the business uses in conversation.**

#### How to apply

- If the business says "order status is 'shipped'", the code should use `order.status == "shipped"`, not `order.state == 3`
- If the business says "cancel an order", the method should be `order.cancel()`, not `order.updateStatus(4)`
- Keep a glossary of business terms and enforce it in code reviews

#### Gotchas

- Ubiquitous Language is violated when code uses `status` but the business says `state`
- The language must be consistent across the team — if the business changes a term, update the code
- Don't mix business terms with technical terms in the same context (e.g., `OrderDTO` — "Order" is business, "DTO" is technical, but together they're fine as a boundary object)

---

### 2. Package Isolation

> Simplified from Bounded Context

**Code in one package should not directly reference internals of another.**

#### How to apply

- Each package has a clear public interface (exported types/functions)
- Internal types are package-private
- Cross-package communication happens through the public interface

```python
# order/__init__.py — public interface
from .service import OrderService
from .models import Order, OrderItem

# order/_internal.py — private, not exported
class OrderValidator:  # internal detail
    ...
```

#### Gotchas

- Package Isolation doesn't mean "no imports" — it means "import interfaces, not internals"
- In Python, use `_` prefix for internal modules. In Go, use lowercase identifiers. In both, the convention is the enforcement

---

### 3. Entry Class

> Simplified from Aggregate Root

**Control access to an object graph through a single entry point.**

#### How to apply

- If `Order` contains `OrderItem`s, all modifications to items go through `Order`
- No direct manipulation of `order.items` from outside

```python
class Order:
    def __init__(self):
        self._items = []

    def add_item(self, product, quantity):
        self._items.append(OrderItem(product, quantity))

    @property
    def items(self):
        return list(self._items)  # return a copy, not the original
```

#### Gotchas

- Entry Class is not "one class to rule them all" — each aggregate has its own entry
- The entry class is responsible for **invariants** — rules that must always be true (e.g., "order total must be positive")
- Don't create entry classes for simple data that has no behavior — a plain list is fine for a shopping cart that's just a list of IDs

---

### 4. Data Access Layer

> Simplified from Repository Pattern

**Encapsulate all data access behind a dedicated layer.**

#### How to apply

- Business logic never writes SQL directly
- All queries go through a data access class
- The data access class returns domain objects, not database rows

```python
class OrderRepository:
    def find_by_id(self, order_id: int) -> Order: ...
    def find_by_customer(self, customer_id: int) -> list[Order]: ...
    def save(self, order: Order): ...
```

#### Gotchas

- In a monolith, a simple DAO (Data Access Object) is sufficient — you don't need the full Repository pattern with Unit of Work
- The data access layer should return domain objects, not raw database rows. The mapping happens inside the layer
- Don't put business logic in the data access layer — it's for data access only
