# SOLID 原则

> 出处：Robert C. Martin（Uncle Bob），1990s-2000s。首次出现在他的文章中，后整理成书《Agile Software Development, Principles, Patterns, and Practices》（2002）。

## S — 单一职责原则 (Single Responsibility Principle)

> **一个类应该只有一个变更理由。**

### 意图

当一个类有多个职责时，一个职责的变更可能破坏其他职责。SRP 让每个类保持专注，使其更容易理解、测试和维护。

### 如何识别违规

- 一个类有多个"变更轴心"——不同的人因为不同的理由要求修改它
- 一个类 import 了不相关的领域（如 `User` 类同时处理 email 格式化和数据库持久化）
- 你在不同的 PR 中因为不同的理由修改同一个类

### 重构前后

**坏：**
```python
class Order:
    def calculate_total(self): ...
    def save_to_database(self): ...
    def render_html(self): ...
    def send_confirmation_email(self): ...
```

**好：**
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

### 常见踩坑

- SRP 说的是**变更理由**，不是"只做一件事"。一个类可以有多个方法，只要它们都服务于同一个利益相关者的关注点
- "变更理由" = 会要求修改这个类的人或角色。如果财务团队和物流团队都要修改同一个类，说明它有两个职责
- 不要过度拆分——一个 3 个方法的类没问题。但 3 个方法分属不同领域就有问题

---

## O — 开闭原则 (Open/Closed Principle)

> **软件实体应对扩展开放，对修改关闭。**

### 意图

你应该能够通过新增代码来添加新行为，而不需要修改已有代码。这防止了在添加功能时破坏已经能工作的代码。

### 如何识别违规

- 添加新功能需要修改已有的 switch/if-else 链
- 你在给已有的 switch 加 `case`
- 你在加 `if type == NEW_TYPE` 检查

### 重构前后

**坏：**
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

**好：**
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

### 常见踩坑

- OCP 不是说"永远不改代码"——而是说变更应该通过新增代码实现，不是修改已有代码
- 过早抽象违反 YAGNI——不要等到有确凿证据就做"可插拔"设计
- 最佳时机：在**第二次**出现相同模式时抽象，不是第一次

---

## L — 里氏替换原则 (Liskov Substitution Principle)

> **子类型必须可替换其基类型。**

### 意图

如果程序使用基类，它应该能正确地与任何子类一起工作，而不需要知道用的是哪个子类。违规会破坏多态，迫使代码中出现 `if type(x) == ...` 检查。

### 如何识别违规

- 子类抛出基类方法不抛的异常
- 子类返回与基类不同的类型
- 子类有空方法体（"不支持"）
- 调用方在使用对象前检查 `isinstance()`

### 重构前后

**坏：**
```python
class Rectangle:
    def set_width(self, w): self.width = w
    def set_height(self, h): self.height = h

class Square(Rectangle):
    def set_width(self, w):
        self.width = w
        self.height = w  # 违规：只设置了宽度，却改了高度
```

**好：**
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

### 常见踩坑

- LSP 违规的典型表现："这个子类不需要那个方法"——这是设计坏味道
- 经典的"is-a"测试：如果子类改变了父类的行为，它就不是真正的子类型
- 实践中：如果你在任何地方写了 `if isinstance(x, SpecialCase)`，很可能有 LSP 违规

---

## I — 接口隔离原则 (Interface Segregation Principle)

> **多个客户端专用接口优于一个通用接口。**

### 意图

没有客户端应该被迫依赖它不使用的方法。胖接口导致耦合——修改一个方法会强制所有客户端重新编译，即使它们不用这个方法。

### 如何识别违规

- 一个类实现了接口但有些方法为空或抛出 `NotImplementedError`
- 一个类接收了它不用的依赖（构造函数注入了未使用的服务）
- 接口中有"可选"方法，部分实现不需要

### 重构前后

**坏：**
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
    def eat(self): raise NotImplementedError  # 被迫依赖
```

**好：**
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

### 常见踩坑

- ISP 违规的标志：实现类里有空方法体
- ISP 也适用于**函数参数**——不要只为了用一个字段就传整个对象
- 在动态类型语言中，ISP 更多是约定（鸭子类型）而不是接口声明

---

## D — 依赖反转原则 (Dependency Inversion Principle)

> **依赖抽象，不依赖具体实现。**

### 意图

高层模块不应依赖低层模块。两者都应依赖抽象。这解耦了业务逻辑与基础设施细节（数据库、API、文件系统）。

### 如何识别违规

- 业务逻辑类直接 import 数据库驱动
- 更换数据库库需要修改业务逻辑
- 单元测试很慢，因为需要真实数据库
- 无法在不修改调用方的情况下替换实现

### 重构前后

**坏：**
```python
class OrderService:
    def __init__(self):
        self.db = MySQLConnection("localhost")  # 依赖具体实现

    def save(self, order):
        self.db.execute("INSERT INTO orders ...")
```

**好：**
```python
class OrderRepository(ABC):
    @abstractmethod
    def save(self, order): ...

class MySQLOrderRepository(OrderRepository):
    def save(self, order):
        # MySQL 具体实现

class OrderService:
    def __init__(self, repo: OrderRepository):  # 依赖抽象
        self.repo = repo

    def save(self, order):
        self.repo.save(order)
```

### 常见踩坑

- DIP 常被混淆为依赖注入——DI 是技术手段（通过构造函数传依赖），DIP 是原则（依赖抽象）
- DIP 不是说"所有东西都抽象"——只抽象**易变依赖**（会变化的东西：数据库、API、文件系统）。稳定的东西如 `str`、`datetime` 不需要抽象
- 抽象应该**由高层模块定义**，而不是低层模块——业务逻辑定义它需要什么，而不是基础设施提供什么
