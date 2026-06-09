---
name: software-design
description: "Comprehensive software design principles for monolith applications. Covers SOLID, GRASP, DRY, KISS, YAGNI, Law of Demeter, CQS, Layered Architecture, Dependency Rule, and more. Use when the user asks about software design, code architecture, design principles, refactoring guidance, module organization, or when writing code that needs structural guidance."
version: 1.0.0
released: 2026-06-09
authors:
  - hyper0x
license: MIT
---

# Software Design Principles

A curated collection of battle-tested software design principles, organized by scope. Each principle includes its origin, intent, and — most importantly — the **gotchas** that trip up even experienced developers.

## How to use this skill

This skill uses **progressive disclosure**. Start here for the overview, then dive into the reference files for depth.

```
SKILL.md               ← Navigation hub (this file)
├── references/        ← Deep dives: each principle group
│   ├── solid.md
│   ├── grasp.md
│   ├── general.md
│   ├── interface-design.md
│   ├── package-design.md
│   └── architecture.md
├── examples/          ← Before/after code examples
│   ├── solid/
│   ├── grasp/
│   └── architecture/
└── scripts/           ← Automation (audit, lint, etc.)
```

Chinese version is available in `_zh/` (mirrors the root structure).

---

## Quick reference

| Group | Principles | Focus |
|-------|-----------|-------|
| [SOLID](references/solid.md) | SRP, OCP, LSP, ISP, DIP | Class-level design |
| [GRASP](references/grasp.md) | 9 responsibility assignment patterns | Object collaboration |
| [General](references/general.md) | DRY, KISS, YAGNI, Composition > Inheritance, Tell Don't Ask, Fail Fast, SoC, Information Hiding, SLA, Boy Scout, Extension > Config | Cross-cutting |
| [Interface Design](references/interface-design.md) | Law of Demeter, CQS, Postel's Law, Least Surprise | API & method contracts |
| [Package Design](references/package-design.md) | ADP, CCP, SDP | Module boundaries |
| [Architecture](references/architecture.md) | Layered, Dependency Rule, Ports & Adapters, DDD lightweight | System structure |

---

## SOLID

| # | Principle | Origin | One-liner |
|---|-----------|--------|-----------|
| S | Single Responsibility | Robert C. Martin | A class should have one reason to change |
| O | Open/Closed | Bertrand Meyer | Open for extension, closed for modification |
| L | Liskov Substitution | Barbara Liskov | Subtypes must be substitutable for their base types |
| I | Interface Segregation | Robert C. Martin | Many client-specific interfaces are better than one general-purpose interface |
| D | Dependency Inversion | Robert C. Martin | Depend on abstractions, not concretions |

**Gotchas:**
- SRP is about **reasons to change**, not about "doing one thing" — a class can do many things if they all change for the same reason
- OCP doesn't mean "no changes ever" — it means changes should come via new code, not modifying existing code
- LSP violations often look like "this subclass doesn't need that method" — that's a design smell
- ISP is violated when you see empty method bodies in implementations
- DIP is often confused with Dependency Injection — DI is a technique, DIP is the principle

→ [Full SOLID reference](references/solid.md)

---

## GRASP

| # | Principle | Origin | One-liner |
|---|-----------|--------|-----------|
| 1 | Information Expert | Craig Larman | Assign responsibility to the class that has the information needed to fulfill it |
| 2 | Creator | Craig Larman | Let a class create instances of classes it contains, aggregates, or closely uses |
| 3 | Controller | Craig Larman | Assign system events to a controller class that delegates to other objects |
| 4 | Low Coupling | Craig Larman | Minimize dependencies between classes |
| 5 | High Cohesion | Craig Larman | Keep related responsibilities together within a class |
| 6 | Polymorphism | Craig Larman | Use polymorphism to handle variations in type-based behavior |
| 7 | Pure Fabrication | Craig Larman | Create non-domain classes to achieve low coupling when domain classes won't do |
| 8 | Indirection | Craig Larman | Use an intermediate object to mediate between components |
| 9 | Protected Variations | Craig Larman | Identify points of variation and protect them with stable interfaces |

**Gotchas:**
- Information Expert doesn't mean "put everything in one class" — balance with High Cohesion
- Creator is not "whoever calls `new`" — it's about ownership semantics
- Controller should not become a "god class" — it delegates, it doesn't do everything
- Pure Fabrication is not "make up classes for fun" — it's a last resort when domain classes create too much coupling
- Protected Variations is the GRASP name for what OCP and DIP also address — they're complementary views

→ [Full GRASP reference](references/grasp.md)

---

## General Principles

| Principle | Origin | One-liner |
|-----------|--------|-----------|
| DRY | Hunt & Thomas | Every piece of knowledge must have a single, unambiguous representation |
| KISS | Kelly Johnson | Simple systems work better than complex ones |
| YAGNI | Ron Jeffries / XP | Don't add functionality until it's necessary |
| Composition over Inheritance | GoF | Prefer composing behaviors over inheriting them |
| Tell, Don't Ask | Hunt & Thomas | Tell objects what to do, don't ask for their data and do it yourself |
| Fail Fast | Hunt & Thomas | Let errors surface immediately at the point of failure |
| Separation of Concerns | Edsger Dijkstra | Different concerns belong in different modules |
| Information Hiding | David Parnas | Hide implementation details behind stable interfaces |
| Single Level of Abstraction | Robert C. Martin | Code within a function should be at the same level of abstraction |
| Boy Scout Rule | Hunt & Thomas | Leave the codebase cleaner than you found it |
| Extension over Configuration | — | Prefer new code paths over configuration flags for new behavior |

**Gotchas:**
- DRY is about **knowledge**, not code — two identical blocks serving different business rules are NOT duplication
- KISS is not "dumb it down" — it's "don't add accidental complexity"
- YAGNI is not "don't plan" — it's "don't build what you don't need today"
- Composition over Inheritance is a **preference**, not a rule — there are valid uses of inheritance
- Tell Don't Ask doesn't mean "never call getters" — it means "don't make decisions for objects that can decide for themselves"
- Fail Fast is about **development-time** safety, not production error handling
- Extension over Configuration doesn't mean "never use config" — it means "don't let config replace good design"

→ [Full General reference](references/general.md)

---

## Interface Design

| Principle | Origin | One-liner |
|-----------|--------|-----------|
| Law of Demeter | Northeastern Univ. | Only talk to your immediate friends |
| Command-Query Separation | Bertrand Meyer | A method should either change state or return data, not both |
| Postel's Law | Jon Postel / TCP | Be conservative in what you send, liberal in what you accept |
| Principle of Least Surprise | — | A function's behavior should match what its name and signature imply |

**Gotchas:**
- LoD is not "never chain calls" — it's "don't reach through objects to get at their internals"
- CQS doesn't mean "never return a value from a mutating method" — it means "don't hide mutations in query methods"
- Postel's Law is about **input tolerance**, not "accept anything" — validate strictly at boundaries
- Least Surprise violations often come from side effects — `save()` should not send emails

→ [Full Interface Design reference](references/interface-design.md)

---

## Package Design

| Principle | Origin | One-liner |
|-----------|--------|-----------|
| ADP — Acyclic Dependencies | Robert C. Martin | The dependency graph of packages must have no cycles |
| CCP — Common Closure | Robert C. Martin | Classes that change together belong together |
| SDP — Stable Dependencies | Robert C. Martin | Depend in the direction of stability |

**Gotchas:**
- ADP violations cause cascading rebuilds — one change forces recompilation of everything
- CCP is the package-level equivalent of SRP — a package should have one reason to change
- SDP doesn't mean "stable packages are better" — it means "unstable packages should depend on stable ones"

→ [Full Package Design reference](references/package-design.md)

---

## Architecture (Monolith)

| Principle | Origin | One-liner |
|-----------|--------|-----------|
| Layered Architecture | — | Organize code into horizontal layers with clear responsibilities |
| Dependency Rule | Robert C. Martin | Source code dependencies must point inward, toward higher-level policies |
| Ports & Adapters | Alistair Cockburn | Core business logic communicates with the outside world through interfaces |
| Ubiquitous Language | Eric Evans | Use the same terms in code as the business uses in conversation |
| Package Isolation | — | Code in one package should not directly reference internals of another |
| Entry Class | — | Control access to an object graph through a single entry point |
| Data Access Layer | — | Encapsulate all data access behind a dedicated layer |

**Gotchas:**
- Layered Architecture is not "three layers always" — the number of layers depends on complexity
- Dependency Rule is violated the moment a domain model references a framework type
- Ports & Adapters doesn't mean "interface for everything" — only for external dependencies
- Ubiquitous Language is violated when code uses `status` but the business says `state`
- Package Isolation doesn't mean "no imports" — it means "import interfaces, not internals"
- Entry Class is not "one class to rule them all" — each aggregate has its own entry

→ [Full Architecture reference](references/architecture.md)

---


