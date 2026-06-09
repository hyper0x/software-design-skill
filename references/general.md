# General Design Principles

Cross-cutting principles that apply at every level of software design, from a single function to the entire system.

---

## DRY — Don't Repeat Yourself

> Origin: Andrew Hunt & David Thomas, *The Pragmatic Programmer* (1999)

**Every piece of knowledge must have a single, unambiguous, authoritative representation within a system.**

### Intent

When knowledge is duplicated, changes must be made in multiple places. Eventually one gets missed, introducing inconsistency. DRY reduces the surface area for bugs and makes systems easier to change.

### How to recognize violations

- The same business rule appears in multiple files
- Configuration values are hard-coded in multiple places
- Validation logic exists in both frontend and backend
- API contracts are manually duplicated in client and server code

### Gotchas

- DRY is about **knowledge**, not code. Two identical code blocks serving different business rules are NOT duplication
- There are four types of duplication: imposed (environment forces it), inadvertent (developers don't realize), impatient (too lazy to abstract), inter-developer (multiple people duplicate)
- Code comments that restate the code violate DRY — comments should explain *why*, not *what*
- The opposite of DRY is WET: "Write Everything Twice" or "We Enjoy Typing"

---

## KISS — Keep It Simple, Stupid

> Origin: Kelly Johnson (Lockheed Skunk Works), 1960s

**Simple systems work better than complex ones.**

### Intent

Complexity is the enemy of reliability. Simple code is easier to understand, test, debug, and change. Most "clever" solutions create more problems than they solve.

### How to recognize violations

- A 5-line solution replaced with a 50-line "flexible" framework
- Over-engineered abstractions for problems that don't exist yet
- Deep inheritance hierarchies where a simple function would do
- Complex configuration systems for simple behavior

### Gotchas

- KISS is not "dumb it down" — it's "don't add accidental complexity"
- Simple != easy. Sometimes the simplest solution requires more upfront thinking
- If you need a diagram to explain your code to another developer, it's probably not simple enough

---

## YAGNI — You Ain't Gonna Need It

> Origin: Ron Jeffries, Extreme Programming (1999)

**Don't add functionality until it's necessary.**

### Intent

Every line of code is a liability — it must be understood, tested, and maintained. Adding "future-proof" features that never get used is wasted effort that also complicates the codebase.

### How to recognize violations

- "We might need this later" abstractions
- Configuration flags for features that don't exist yet
- Generic interfaces with only one implementation
- Caching layers before performance problems are measured

### Gotchas

- YAGNI is not "don't plan" — it's "don't build what you don't need today"
- YAGNI doesn't apply to architectural decisions that are hard to reverse (e.g., choosing a database). Those need upfront consideration
- The balance: build what you need today, but design in a way that doesn't prevent tomorrow's changes

---

## Composition over Inheritance

> Origin: Gang of Four, *Design Patterns* (1994)

**Prefer composing behaviors over inheriting them.**

### Intent

Inheritance creates tight coupling between parent and child classes. A change to the parent can break all children. Composition, where objects hold references to other objects and delegate to them, is more flexible and less fragile.

### How to recognize violations

- Deep inheritance hierarchies (3+ levels)
- A subclass overrides most methods of its parent
- The "diamond problem" (multiple inheritance conflicts)
- A class inherits behavior it doesn't need

### Example

**Inheritance (fragile):**
```python
class Bird:
    def fly(self): ...

class Ostrich(Bird):
    def fly(self): raise NotImplementedError  # problem!
```

**Composition (flexible):**
```python
class Bird:
    def __init__(self, movement: MovementStrategy):
        self.movement = movement

    def move(self):
        self.movement.move()

class FlyingMovement:
    def move(self): ...

class WalkingMovement:
    def move(self): ...
```

### Gotchas

- Composition over Inheritance is a **preference**, not a rule. There are valid uses of inheritance (e.g., template method pattern, framework base classes)
- The test: "Is the subclass truly a subtype of the parent?" If yes, inheritance may be appropriate. If "I just want to reuse some methods," use composition

---

## Tell, Don't Ask

> Origin: Andrew Hunt & David Thomas, *The Pragmatic Programmer* (1999)

**Tell objects what to do, don't ask for their data and do it yourself.**

### Intent

Procedural code asks for data and makes decisions. Object-oriented code tells objects what to do and lets them decide how. This keeps behavior and data together.

### How to recognize violations

```python
# Violation: asking for data, then deciding
if employee.type == "manager":
    bonus = salary * 0.2
else:
    bonus = salary * 0.1

# Better: tell the object
bonus = employee.calculate_bonus(salary)
```

### Gotchas

- Tell Don't Ask doesn't mean "never call getters" — it means "don't make decisions for objects that can decide for themselves"
- The line is fuzzy with data objects (DTOs, value objects) — those exist to hold data, and asking them for it is fine

---

## Fail Fast

> Origin: Andrew Hunt & David Thomas, *The Pragmatic Programmer* (1999)

**Let errors surface immediately at the point of failure.**

### Intent

A dead program does far less damage than a crippled one. Silent failures corrupt data, mask bugs, and make debugging a nightmare. Fail early, fail loudly.

### How to recognize violations

- `except: pass` — silently swallowing exceptions
- Returning `None` or `-1` instead of raising an error
- Continuing execution after detecting an invalid state
- Logging an error and pretending nothing happened

### Gotchas

- Fail Fast is about **development-time** safety, not production error handling. In production, you may want graceful degradation — but the error should still be visible
- The balance: validate inputs at boundaries (API entry points), not in every internal function
- Use assertions for "should never happen" conditions, error handling for "might happen" conditions

---

## Separation of Concerns

> Origin: Edsger Dijkstra, *On the role of scientific thought* (1974)

**Different concerns belong in different modules.**

### Intent

Each module should address a single, well-defined concern. Mixing concerns (e.g., business logic mixed with database code) creates tight coupling and makes changes risky.

### How to recognize violations

- A single function handles HTTP parsing, business logic, and database access
- Changing the UI requires changing business logic
- Business rules are scattered across controllers, views, and helpers

### Gotchas

- SoC is the parent concept that SRP, Layered Architecture, and Package Design all derive from
- The art is knowing where to draw the boundaries — too fine and you get fragmentation, too coarse and you get coupling

---

## Information Hiding

> Origin: David Parnas, *On the Criteria to Be Used in Decomposing Systems into Modules* (1972)

**Hide implementation details behind stable interfaces.**

### Intent

The internals of a module should be invisible to its consumers. This allows the implementation to change without affecting callers. Information hiding is the foundation of encapsulation.

### How to recognize violations

- Public fields that should be private
- Internal data structures exposed through getters
- Callers depend on the internal format of returned data
- Comments that explain "how" instead of "what"

### Gotchas

- Information Hiding is not the same as "getters and setters" — a getter that returns internal state is still exposing it
- The question to ask: "If I change the internal representation, how many callers break?" The answer should be zero

---

## Single Level of Abstraction

> Origin: Robert C. Martin, *Clean Code* (2008)

**Code within a function should be at the same level of abstraction.**

### Intent

A function should read like a story — high-level steps at the top, each delegating to a lower-level helper. Mixing abstraction levels (e.g., a high-level business step next to a low-level string manipulation) makes code hard to read.

### Example

**Bad:**
```python
def process_order(order):
    validate_order(order)  # high-level
    total = sum(item.price * item.quantity for item in order.items)  # low-level detail
    apply_discount(order, total)  # high-level
    order.status = "processed"  # low-level detail
```

**Good:**
```python
def process_order(order):
    validate_order(order)
    total = calculate_total(order)
    apply_discount(order, total)
    mark_as_processed(order)
```

### Gotchas

- This is a **readability** principle, not a correctness principle — the code works either way, but the "good" version is easier to scan
- The test: can you read the function body and understand the flow without reading the helper functions?

---

## Boy Scout Rule

> Origin: Andrew Hunt & David Thomas, *The Pragmatic Programmer* (1999)

**Leave the codebase cleaner than you found it.**

### Intent

Small, continuous improvements prevent code rot. If everyone fixes one small thing each time they touch a file, the codebase gets better over time instead of worse.

### How to apply

- Rename a poorly named variable when you encounter it
- Extract a method when you see a comment explaining a block
- Remove dead code when you notice it
- Fix formatting inconsistencies in files you're editing

### Gotchas

- The Boy Scout Rule is about **small improvements**, not major refactoring. If the change is large, create a separate ticket
- Don't mix cleanup with feature work in the same commit — separate commits for separate concerns
- The rule applies to code you're already touching, not "go find things to clean"

---

## Extension over Configuration

> Prefer new code paths over configuration flags for new behavior.

### Intent

Configuration flags (`if feature_enabled`) create implicit code paths that are hard to test, reason about, and remove. Adding a new class or function for new behavior is cleaner and more explicit.

### How to recognize violations

```python
# Violation: configuration flag creates hidden code paths
def process_payment(order):
    if config.new_payment_flow:
        # new flow
    else:
        # old flow

# Better: separate code paths
class PaymentProcessor(ABC):
    @abstractmethod
    def process(self, order): ...

class OldPaymentProcessor(PaymentProcessor): ...
class NewPaymentProcessor(PaymentProcessor): ...
```

### Gotchas

- Extension over Configuration doesn't mean "never use config" — it means "don't let config replace good design"
- Configuration is appropriate for: environment differences (dev/staging/prod), user preferences, and truly variable parameters (timeouts, limits)
- Configuration is inappropriate for: behavioral changes, feature toggles that persist for months, and branching logic that creates two parallel code paths
