# SOLID Principles

> Origin: Robert C. Martin (Uncle Bob), 1990s-2000s. First introduced in his articles and later consolidated in *Agile Software Development, Principles, Patterns, and Practices* (2002).

## S — Single Responsibility Principle

> **A class should have one, and only one, reason to change.**

### Intent

When a class has multiple responsibilities, a change to one responsibility may break the others. SRP keeps each class focused, making it easier to understand, test, and maintain.

### How to recognize violations

- A class has more than one "axis of change" — different stakeholders request changes for different reasons
- A single class imports from unrelated domains (e.g., a `User` class that also handles email formatting and database persistence)
- You find yourself changing the same class for different reasons in different PRs

### Before/After

**Bad:**
```python
class Order:
    def calculate_total(self): ...
    def save_to_database(self): ...
    def render_html(self): ...
    def send_confirmation_email(self): ...
```

**Good:**
```python
class Order:
    def calculate_total(self): ...

class OrderRepository:
    def save(self, order): ...

class OrderRenderer:
    def to_html(self, order): ...

class OrderNotificationService:
    def send_confirmation(self, order): ...
```

### Gotchas

- SRP is about **reasons to change**, not about "doing one thing." A class can have multiple methods if they all serve the same stakeholder's concern
- "Reason to change" = a person or group who would request a change. If the accounting team and the shipping team both want changes to the same class, it has two responsibilities
- Don't over-split — a class with 3 methods is fine. A class with 3 methods that each belong to different domains is not

---

## O — Open/Closed Principle

> **Software entities should be open for extension, but closed for modification.**

### Intent

You should be able to add new behavior without changing existing code. This prevents introducing bugs in working code when adding features.

### How to recognize violations

- Adding a new feature requires modifying an existing class with a switch/if-else chain
- You're adding `case` statements to an existing switch
- You're adding `if type == NEW_TYPE` checks

### Before/After

**Bad:**
```python
class DiscountCalculator:
    def calculate(self, order, customer_type):
        if customer_type == "regular":
            return order.total * 0.05
        elif customer_type == "vip":
            return order.total * 0.10
        elif customer_type == "employee":
            return order.total * 0.20
```

**Good:**
```python
from abc import ABC, abstractmethod

class DiscountStrategy(ABC):
    @abstractmethod
    def calculate(self, order): ...

class RegularDiscount(DiscountStrategy):
    def calculate(self, order): return order.total * 0.05

class VipDiscount(DiscountStrategy):
    def calculate(self, order): return order.total * 0.10

class EmployeeDiscount(DiscountStrategy):
    def calculate(self, order): return order.total * 0.20
```

### Gotchas

- OCP doesn't mean "no changes ever" — it means changes should come via new code, not modifying existing code
- Premature abstraction violates YAGNI — don't make everything "pluggable" until you have concrete evidence you'll need it
- The sweet spot: abstract at the **second occurrence** of a pattern, not the first

---

## L — Liskov Substitution Principle

> **Subtypes must be substitutable for their base types.**

### Intent

If a program uses a base class, it should work correctly with any of its subclasses without knowing which subclass it is. Violations break polymorphism and require `if type(x) == ...` checks.

### How to recognize violations

- A subclass throws exceptions for methods that work in the base class
- A subclass returns different types than the base class
- A subclass has empty method bodies ("not supported")
- Callers check `isinstance()` before using an object

### Before/After

**Bad:**
```python
class Rectangle:
    def set_width(self, w): self.width = w
    def set_height(self, h): self.height = h

class Square(Rectangle):
    def set_width(self, w):
        self.width = w
        self.height = w  # violates: changes height when only width was set
```

**Good:**
```python
class Shape(ABC):
    @abstractmethod
    def area(self): ...

class Rectangle(Shape):
    def __init__(self, w, h): self.width, self.height = w, h
    def area(self): return self.width * self.height

class Square(Shape):
    def __init__(self, side): self.side = side
    def area(self): return self.side ** 2
```

### Gotchas

- LSP violations often look like "this subclass doesn't need that method" — that's a design smell
- The classic "is-a" test: if a subclass changes behavior inherited from the parent, it's not truly a subtype
- In practice: if you have `if isinstance(x, SpecialCase)` anywhere, you likely have an LSP violation

---

## I — Interface Segregation Principle

> **Many client-specific interfaces are better than one general-purpose interface.**

### Intent

No client should be forced to depend on methods it does not use. Fat interfaces lead to coupling — changing one method forces recompilation of all clients, even those that don't use it.

### How to recognize violations

- A class implements an interface but leaves some methods empty or throwing `NotImplementedError`
- A class receives dependencies it doesn't use (constructor injection of unused services)
- You see "optional" methods in an interface that some implementations don't need

### Before/After

**Bad:**
```python
class Worker(ABC):
    @abstractmethod
    def work(self): ...
    @abstractmethod
    def eat(self): ...

class Human(Worker):
    def work(self): ...
    def eat(self): ...

class Robot(Worker):
    def work(self): ...
    def eat(self): raise NotImplementedError  # forced dependency
```

**Good:**
```python
class Workable(ABC):
    @abstractmethod
    def work(self): ...

class Eatable(ABC):
    @abstractmethod
    def eat(self): ...

class Human(Workable, Eatable):
    def work(self): ...
    def eat(self): ...

class Robot(Workable):
    def work(self): ...
```

### Gotchas

- ISP is violated when you see empty method bodies in implementations
- ISP applies to **function parameters** too — don't pass a whole object when you only need one field
- In dynamically typed languages, ISP is about convention (duck typing) rather than interface declarations

---

## D — Dependency Inversion Principle

> **Depend on abstractions, not on concretions.**

### Intent

High-level modules should not depend on low-level modules. Both should depend on abstractions. This decouples business logic from infrastructure details (databases, APIs, file systems).

### How to recognize violations

- A business logic class imports a database driver directly
- Changing a database library requires changes in business logic
- Unit tests are slow because they need a real database
- You can't swap implementations without changing callers

### Before/After

**Bad:**
```python
class OrderService:
    def __init__(self):
        self.db = MySQLConnection("localhost")  # depends on concrete

    def save(self, order):
        self.db.execute("INSERT INTO orders ...")
```

**Good:**
```python
class OrderRepository(ABC):
    @abstractmethod
    def save(self, order): ...

class MySQLOrderRepository(OrderRepository):
    def save(self, order):
        # MySQL-specific implementation

class OrderService:
    def __init__(self, repo: OrderRepository):  # depends on abstraction
        self.repo = repo

    def save(self, order):
        self.repo.save(order)
```

### Gotchas

- DIP is often confused with Dependency Injection — DI is a technique (passing dependencies via constructor), DIP is the principle (depending on abstractions)
- DIP doesn't mean "interface for everything" — only for **volatile dependencies** (things that change: databases, APIs, file systems). Stable things like `std::string` or Python's `datetime` don't need abstraction
- The abstraction should be **owned by the high-level module**, not the low-level one — the business logic defines what it needs, not what the infrastructure provides
