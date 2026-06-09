# GRASP 原则

> 出处：Craig Larman，《Applying UML and Patterns》（1997）。GRASP = General Responsibility Assignment Software Patterns（通用职责分配软件模式）。

GRASP 是一套**分配职责**给类和对象的框架。与 SOLID（告诉你*什么*不该做）不同，GRASP 告诉你*谁*应该做什么。

## 1. 信息专家 (Information Expert)

> **把职责分配给拥有所需信息的类。**

### 意图

拥有数据的类应该做相关的工作。这保持了行为与数据的统一，是面向对象设计的核心。

### 如何识别违规

- 类 A 的方法从类 B 拉取数据来做关于 B 数据的决策
- 数据和行为分散在不同的类中

### 示例

**坏：** `CartService` 从 `Cart` 拉取数据来计算总额：
```python
class CartService:
    def calculate_total(self, cart):
        total = 0
        for item in cart.items:
            total += item.price * item.quantity
        return total
```

**好：** `Cart` 知道自己的数据，所以它来计算：
```python
class Cart:
    @property
    def total(self):
        return sum(item.price * item.quantity for item in self.items)
```

### 踩坑
- 要与高内聚平衡——不要因为"有信息"就把所有东西塞进一个类

---

## 2. 创建者 (Creator)

> **如果类 B 包含、聚合、记录或紧密使用类 A，则把创建 A 实例的职责分配给 B。**

### 意图

对象应该创建它们需要管理的对象。这减少了耦合——创建者与创建的对象之间已有关系。

### 示例

```python
class Order:
    def add_item(self, product, quantity):
        item = OrderItem(product, quantity)  # Order 创建 OrderItem
        self.items.append(item)
```

### 踩坑
- 创建者不是说"谁调 new 谁创建"——要考虑所有权语义。如果类 A 只是短暂使用类 B，不要让 A 创建 B

---

## 3. 控制器 (Controller)

> **系统事件由控制器接收，然后委派给其他对象。**

### 意图

控制器接收外部事件（用户操作、API 调用），然后委派给合适的领域对象。它不应该自己做工作。

### 示例

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

### 踩坑
- 控制器不应变成"上帝类"——它委派，它不自己做所有事。如果你的控制器有 50 个方法且包含复杂逻辑，说明它做得太多了

---

## 4. 低耦合 (Low Coupling)

> **最小化类之间的依赖。**

### 意图

低耦合减少了变更的连锁反应。当类 A 依赖类 B 时，B 的变更可能需要 A 也变更。

### 如何衡量

- 数一个类依赖了多少其他类
- 数有多少其他类依赖这个类
- 一个类有 10+ 个依赖是危险信号

### 降低耦合的技巧

- 依赖接口而非具体类
- 使用依赖注入
- 引入中间对象（间接原则）
- 用事件/通知替代直接调用

### 踩坑
- 零耦合不可能也不可取——目标是**可管理的**耦合，不是无耦合

---

## 5. 高内聚 (High Cohesion)

> **把相关职责放在同一个类里。**

### 意图

高内聚的类，其方法都服务于同一个明确定义的目的。这让类更容易理解、测试和维护。

### 如何识别低内聚

- 类的方法操作的是不同的字段子集
- 你不能用一句话描述这个类而不说"和"
- 类的方法来自不相关的领域

### 示例

**低内聚：**
```python
class ReportGenerator:
    def fetch_data(self): ...
    def format_report(self): ...
    def send_email(self): ...
    def archive_report(self): ...
```

**高内聚：**
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

### 踩坑
- 高内聚和低耦合是一枚硬币的两面——改善一个通常会改善另一个

---

## 6. 多态 (Polymorphism)

> **用多态处理基于类型的行为变化。**

### 意图

当行为因类型而异时，用多态方法替代条件逻辑。这是 GRASP 对 OCP 的另一种表述。

### 示例

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

### 踩坑
- 不要对所有东西都用多态——如果变化很简单（一行代码），条件判断可能更清晰

---

## 7. 纯虚构 (Pure Fabrication)

> **当领域类导致高耦合时，创建非领域类来解耦。**

### 意图

有时把职责分配给领域类会违反低耦合或高内聚。这时，创建一个不代表领域概念——一个"纯虚构"类。

### 示例

```python
# 领域类不应该知道持久化
class Order:
    def __init__(self, items):
        self.items = items

# 纯虚构：不是领域概念，但为了解耦而需要
class OrderRepository:
    def save(self, order): ...
    def find_by_id(self, id): ...
```

### 踩坑
- 纯虚构不是说"随便编类"——是在领域类导致耦合时才用的最后手段
- Repository、Service、Factory、Adapter——这些都是纯虚构的例子

---

## 8. 间接 (Indirection)

> **用中间对象来协调组件间的关系。**

### 意图

两个组件之间的直接耦合会造成紧密依赖。引入中间人来解耦它们。

### 示例

```python
# 无间接：OrderService 直接依赖 WeChatNotifier
class OrderService:
    def confirm(self, order):
        WeChatNotifier.send(order)  # 紧耦合

# 有间接：两者都依赖接口
class OrderService:
    def __init__(self, notifier: Notifier):
        self.notifier = notifier

    def confirm(self, order):
        self.notifier.send(order)
```

### 踩坑
- 过多的间接会产生"中间人"反模式——如果中间人除了转发什么都不做，就删掉它

---

## 9. 保护变化 (Protected Variations)

> **识别变化点，用稳定接口保护它们。**

### 意图

找到系统中可能变化的部分（数据库、支付网关、文件格式），把它们包在稳定接口后面。这是 GRASP 对 OCP 和 DIP 的另一种表述。

### 示例

```python
# 变化点：不同的支付网关
class PaymentGateway(ABC):
    @abstractmethod
    def pay(self, amount): ...

class StripeGateway(PaymentGateway):
    def pay(self, amount): ...

class WeChatPayGateway(PaymentGateway):
    def pay(self, amount): ...
```

### 踩坑
- 保护变化是 GRASP 对 OCP 和 DIP 的统合——它们是同一想法的不同视角
- 不要为还不存在的变化做保护（YAGNI）
