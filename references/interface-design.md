# Interface Design Principles

Principles for designing method signatures, API boundaries, and contracts between components.

---

## Law of Demeter (LoD)

> Origin: Northeastern University, 1987. Formally: "Each unit should have only limited knowledge about other units."

**Only talk to your immediate friends.**

### Intent

A method should only call methods on:
1. Itself
2. Its own parameters
3. Objects it creates
4. Its own direct component objects

It should **not** reach through one object to call methods on a second object's internals.

### How to recognize violations

```python
# Violation: reaching through objects
order.getCustomer().getAddress().getCity()

# Better: tell the order what you need
order.getCustomerCity()
```

### Gotchas

- LoD is not "never chain calls" — it's "don't reach through objects to get at their internals"
- LoD applies to **internal structure**, not to fluent APIs or builders (`.setX().setY().build()`) — those are designed for chaining
- The real cost of LoD violations: they expose internal structure, making it impossible to change without breaking callers

---

## Command-Query Separation (CQS)

> Origin: Bertrand Meyer, *Object-Oriented Software Construction* (1988)

**A method should either change state or return data, but not both.**

### Intent

- **Commands**: methods that change state but return nothing (or return a void/unit)
- **Queries**: methods that return data but have no side effects

When a method does both, callers can't tell if calling it will change something or just look something up.

### How to recognize violations

```python
# Violation: both modifies state and returns data
def pop(self) -> Item:
    self.count -= 1
    return self.items[self.count]

# Better: separate command and query
def remove_last(self):
    self.count -= 1

def peek(self) -> Item:
    return self.items[self.count - 1]
```

### Gotchas

- CQS doesn't mean "never return a value from a mutating method" — it means "don't hide mutations in query methods"
- In practice, some operations are inherently both (e.g., `pop()`). The principle is a guideline, not an absolute rule
- The most dangerous violation: a query method that caches results as a side effect — callers don't expect `getX()` to modify anything

---

## Postel's Law (Robustness Principle)

> Origin: Jon Postel, RFC 761 (TCP specification), 1980

**Be conservative in what you send, be liberal in what you accept.**

### Intent

Your code should produce well-formed, strict output. But when receiving input, be tolerant of minor variations. This makes systems more robust.

### How to apply

- **Sending**: always produce correct, well-formed output. Don't rely on the receiver being tolerant
- **Receiving**: accept reasonable variations (whitespace, casing, extra fields) but reject truly invalid input

### Gotchas

- Postel's Law is about **input tolerance**, not "accept anything" — validate strictly at system boundaries
- In security-critical contexts, be strict in what you accept too — tolerance can be exploited
- The principle has been criticized in modern protocol design — strict validation at boundaries is often preferred

---

## Principle of Least Surprise

> Origin: Common UX principle, applied to software design

**A function's behavior should match what its name and signature imply.**

### Intent

Code should be predictable. When a developer reads `user.save()`, they should be able to predict what it does without reading the implementation. Surprises (side effects, unexpected behavior) erode trust.

### How to recognize violations

```python
# Violation: surprising side effect
def save(self):
    self.db.persist(self)
    self.send_confirmation_email()  # surprise! caller didn't expect this

# Better: explicit naming
def save(self):
    self.db.persist(self)

def save_and_notify(self):
    self.save()
    self.send_confirmation_email()
```

### Gotchas

- Least Surprise violations often come from **side effects** — `save()` should not send emails
- The principle applies to **error behavior** too — if a function can fail, the caller should be able to predict how
- When you can't avoid surprise, document it clearly in the function name or docstring
