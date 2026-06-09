# Package / Module Design Principles

> Origin: Robert C. Martin, *Agile Software Development, Principles, Patterns, and Practices* (2002)

Principles for organizing classes into packages (or modules/directories) and managing dependencies between them.

For monolith applications, three principles matter most:

---

## ADP — Acyclic Dependencies Principle

> **The dependency graph of packages must have no cycles.**

### Intent

When package A depends on package B, and package B depends on package A (directly or transitively), you have a cycle. Cycles make it impossible to:
- Understand the dependency direction
- Test packages in isolation
- Reason about change impact

### How to recognize violations

- Circular imports that cause runtime errors
- Compilation order issues (in compiled languages)
- "I can't extract this code because it depends on everything"
- A change in one package requires rebuilding everything

### Breaking cycles

| Technique | How |
|-----------|-----|
| **Dependency Inversion** | Make both packages depend on an interface in a third package |
| **Extract** | Move the shared dependency into its own package |
| **Merge** | If the cycle is tight, the packages should be one |

### Gotchas

- ADP violations cause cascading rebuilds — one change forces recompilation of everything
- In Go, cycles are caught at compile time (good). In Python, they cause runtime import errors. In both cases, they're a design smell
- A cycle between two packages is almost always a sign that they should be one package, or that shared code needs to be extracted

---

## CCP — Common Closure Principle

> **Classes that change together, belong together.**

### Intent

If two classes are always modified for the same reason, they should be in the same package. This minimizes the number of packages affected by a change.

### How to apply

- Group classes by **change reason**, not by technical category
- Don't put all "controllers" in one package and all "services" in another — group by feature/domain instead

### Example

**Bad (grouped by technical layer):**
```
payment/
├── controllers/
│   ├── OrderController.py
│   └── RefundController.py
├── services/
│   ├── OrderService.py
│   └── RefundService.py
├── models/
│   ├── Order.py
│   └── Refund.py
```

**Good (grouped by change reason):**
```
payment/
├── order/
│   ├── OrderController.py
│   ├── OrderService.py
│   └── Order.py
└── refund/
    ├── RefundController.py
    ├── RefundService.py
    └── Refund.py
```

### Gotchas

- CCP is the package-level equivalent of SRP — a package should have one reason to change
- CCP and CRP (Common Reuse Principle) are in tension — CCP says "put things that change together together," CRP says "don't force clients to depend on things they don't use." In monoliths, CCP is usually more important

---

## SDP — Stable Dependencies Principle

> **Depend in the direction of stability.**

### Intent

Stable packages (things that rarely change) should be depended upon. Unstable packages (things that change frequently) should depend on stable ones, not the other way around.

### What makes a package "stable"

A package is stable if it has many incoming dependencies (many things depend on it) and few outgoing dependencies (it depends on little). Changing a stable package is expensive because everything that depends on it may need to change too.

### How to apply

```
Business Logic (unstable, changes often)
    ↓ depends on
Utility Library (stable, rarely changes)
```

Not:
```
Utility Library (stable)
    ↓ depends on
Business Logic (unstable)  ← BAD: stable depends on unstable
```

### Gotchas

- SDP doesn't mean "stable packages are better" — it means "unstable packages should depend on stable ones"
- Abstract interfaces are the most stable thing — they have no implementation to change. That's why DIP works: both depend on abstractions
- In practice: utility packages (`strings`, `dates`, `logging`) are stable. Business logic packages are unstable. Business logic should depend on utilities, not vice versa
