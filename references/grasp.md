# GRASP Principles

> Origin: Craig Larman, *Applying UML and Patterns* (1997). GRASP = General Responsibility Assignment Software Patterns.

GRASP is a framework for **assigning responsibilities** to classes and objects. Unlike SOLID (which tells you *what* to avoid), GRASP tells you *who* should do what.

## 1. Information Expert

> **Assign a responsibility to the class that has the information needed to fulfill it.**

### Intent

The class that has the data should do the work. This keeps behavior and data together, which is the essence of object-oriented design.

### How to recognize violations

- A method on class A uses getters from class B to make decisions about B's data
- Data and behavior are scattered across different classes

### Example

**Bad:** A `CartService` calculates the total by pulling data from `Cart`:
```python
class CartService:
    def calculate_total(self, cart):
        total = 0
        for item in cart.items:
            total += item.price * item.quantity
        return total
```

**Good:** `Cart` knows its own data, so it calculates:
```python
class Cart:
    @property
    def total(self):
        return sum(item.price * item.quantity for item in self.items)
```

### Gotcha
- Balance with High Cohesion — don't put everything in one class just because it "has the information"

---

## 2. Creator

> **Assign the responsibility for creating instances of class A to class B if B contains, aggregates, records, or closely uses A.**

### Intent

Objects should create the objects they need to manage. This reduces coupling — the creator already has a relationship with the created object.

### Example

```python
class Order:
    def add_item(self, product, quantity):
        item = OrderItem(product, quantity)  # Order creates OrderItem
        self.items.append(item)
```

### Gotcha
- Creator is not "whoever calls `new`" — it's about ownership semantics. If class A only briefly uses class B, don't make A create B

---

## 3. Controller

> **Assign system events to a controller class that delegates to other objects.**

### Intent

A controller receives external events (user actions, API calls) and delegates to the appropriate domain objects. It should not do the work itself.

### Example

```python
class OrderController:
    def __init__(self):
        self.order_service = OrderService()
        self.notification_service = NotificationService()

    def place_order(self, request):
        order = self.order_service.create(request.items)
        self.notification_service.send_confirmation(order)
        return order
```

### Gotcha
- Controller should not become a "god class" — it delegates, it doesn't do everything. If your controller has 50 methods with complex logic, it's doing too much

---

## 4. Low Coupling

> **Minimize dependencies between classes.**

### Intent

Low coupling reduces the ripple effect of changes. When class A depends on class B, a change to B may require changes to A.

### How to measure

- Count the number of classes a given class depends on
- Count the number of classes that depend on a given class
- A class with 10+ dependencies is a red flag

### Techniques to reduce coupling

- Depend on interfaces, not concrete classes
- Use dependency injection
- Introduce intermediary objects (Indirection)
- Use events/notifications instead of direct calls

### Gotcha
- Zero coupling is impossible and undesirable — the goal is **manageable** coupling, not no coupling

---

## 5. High Cohesion

> **Keep related responsibilities together within a class.**

### Intent

A highly cohesive class has methods that all serve a single, well-defined purpose. This makes the class easier to understand, test, and maintain.

### How to recognize low cohesion

- A class's methods operate on different subsets of its fields
- You can't describe the class in one sentence without using "and"
- The class has methods from unrelated domains

### Example

**Low cohesion:**
```python
class ReportGenerator:
    def fetch_data(self): ...
    def format_report(self): ...
    def send_email(self): ...
    def archive_report(self): ...
```

**High cohesion:**
```python
class ReportFetcher:
    def fetch(self): ...

class ReportFormatter:
    def format(self, data): ...

class EmailSender:
    def send(self, report): ...

class ReportArchiver:
    def archive(self, report): ...
```

### Gotcha
- High Cohesion and Low Coupling are two sides of the same coin — improving one often improves the other

---

## 6. Polymorphism

> **Use polymorphism to handle variations in type-based behavior.**

### Intent

When behavior varies by type, use polymorphic methods instead of conditional logic. This is the GRASP name for what OCP also addresses.

### Example

```python
class PaymentMethod(ABC):
    @abstractmethod
    def charge(self, amount): ...

class CreditCard(PaymentMethod):
    def charge(self, amount): ...

class WeChatPay(PaymentMethod):
    def charge(self, amount): ...

class Alipay(PaymentMethod):
    def charge(self, amount): ...
```

### Gotcha
- Don't use polymorphism for everything — if the variation is simple (one line), a conditional may be clearer

---

## 7. Pure Fabrication

> **Create a non-domain class to achieve low coupling when domain classes won't do.**

### Intent

Sometimes, assigning a responsibility to a domain class would violate Low Coupling or High Cohesion. In that case, create a class that doesn't represent a domain concept — a "pure fabrication."

### Example

```python
# Domain class shouldn't know about persistence
class Order:
    def __init__(self, items):
        self.items = items

# Pure fabrication: not a domain concept, but needed for separation
class OrderRepository:
    def save(self, order): ...
    def find_by_id(self, id): ...
```

### Gotcha
- Pure Fabrication is not "make up classes for fun" — it's a last resort when domain classes create too much coupling
- Repository, Service, Factory, Adapter — these are all examples of Pure Fabrication

---

## 8. Indirection

> **Use an intermediate object to mediate between components.**

### Intent

Direct coupling between two components creates tight dependency. Introduce a middleman to decouple them.

### Example

```python
# Without indirection: OrderService directly depends on WeChatNotifier
class OrderService:
    def confirm(self, order):
        WeChatNotifier.send(order)  # tight coupling

# With indirection: both depend on an interface
class OrderService:
    def __init__(self, notifier: Notifier):
        self.notifier = notifier

    def confirm(self, order):
        self.notifier.send(order)
```

### Gotcha
- Too much indirection creates "middle man" anti-pattern — if the intermediary does nothing but forward calls, remove it

---

## 9. Protected Variations

> **Identify points of variation and protect them with stable interfaces.**

### Intent

Find parts of the system that are likely to change (database, payment gateway, file format) and wrap them behind stable interfaces. This is the GRASP name for what OCP and DIP also address.

### Example

```python
# Variation point: different payment gateways
class PaymentGateway(ABC):
    @abstractmethod
    def pay(self, amount): ...

class StripeGateway(PaymentGateway):
    def pay(self, amount): ...

class WeChatPayGateway(PaymentGateway):
    def pay(self, amount): ...
```

### Gotcha
- Protected Variations is the GRASP umbrella for OCP and DIP — they're complementary views of the same idea
- Don't protect against variations that don't exist yet (YAGNI)
