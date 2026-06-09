# 架构原则（单体聚焦）

面向单体应用的架构级原则。这些原则帮助管理复杂度，而不需要分布式系统模式的额外开销。

---

## 分层架构 (Layered Architecture)

> 出处：通用模式，在企业应用架构模式中形式化

**按职责将代码组织成水平层。**

### 经典三层

```
表现层（UI / API 控制器）
    ↓
业务逻辑层（领域服务、业务规则）
    ↓
数据访问层（仓库、数据库访问）
```

### 规则

- 每层只依赖它下面的层
- 没有层应该跳过相邻层（如 UI 直接访问数据库）
- 每层有明确定义的职责

### 常见踩坑

- 分层不是说"永远是三层"——层数取决于复杂度。简单的 CRUD 应用可能只需要两层
- 危险："漏斗反模式"——除了转发什么都不做的层。如果一层没有增加价值，就删掉它
- 在单体中，横切关注点（日志、认证）可以接触所有层——这是对严格分层的放松

---

## 依赖规则 (Dependency Rule)

> 出处：Robert C. Martin，《架构整洁之道》（2017）

**源码依赖必须指向内层（高层策略）。**

### 同心圆

```
框架/驱动       （最外层——细节）
    ↓
接口适配器       （控制器、展示器）
    ↓
应用业务规则     （用例）
    ↓
企业业务规则     （实体——最内层，最高层）
```

### 实践中意味着什么

- 业务逻辑（内圈）**绝不能** import 框架或数据库（外圈）的任何东西
- 业务逻辑应该是纯的——没有框架注解、没有数据库 import、没有 HTTP 关注点
- 所有依赖**向内**跨越边界

### 重构前后

**违规：**
```python
from django.db import models  # 业务逻辑中的框架依赖

class Order(models.Model):
    def calculate_total(self):
        ...
```

**干净：**
```python
# business/order.py — 纯 Python，无框架 import
class Order:
    def __init__(self, items):
        self.items = items

    def calculate_total(self):
        return sum(item.price for item in self.items)
```

### 常见踩坑

- 依赖规则在领域模型引用框架类型的那一刻就被违反了
- 这条规则适用于**源码依赖**，不是运行时流。运行时，控制流从外到内再返回
- 在单体中，你不需要完整的 Clean Architecture 机制——只要核心洞察：业务逻辑不应依赖框架

---

## 端口与适配器（六边形架构）(Ports & Adapters)

> 出处：Alistair Cockburn，2005年

**核心业务逻辑通过接口（端口）与外部世界通信，具体实现（适配器）插在接口上。**

### 结构

```
                    ┌─────────────┐
                    │  适配器      │  ← 如 MySQLRepository
                    │  (实现)      │
                    └──────┬──────┘
                           │ 实现
                    ┌──────┴──────┐
   HTTP 请求 ──────▶│  端口        │  ← 接口（如 OrderRepository）
                    │  (接口)      │
                    └──────┬──────┘
                           │ 被调用
                    ┌──────┴──────┐
                    │  核心        │  ← 业务逻辑，纯的
                    │  (领域)      │
                    └─────────────┘
```

### 什么需要抽象

| 需要抽象（创建端口） | 不需要抽象 |
|---------------------|-----------|
| 数据库 | 标准库类型 |
| 外部 API | 值对象（Money、Date） |
| 文件系统 | 语言内置类型 |
| 消息队列 | 你拥有的第三方 SDK |

### 常见踩坑

- 端口与适配器不是说"所有东西都抽象"——只抽象**可能变化的外部依赖**
- 端口（接口）应该由**核心定义**，而不是适配器。核心说"我需要一个能保存订单的仓库"——适配器实现它
- 在单体中，你不需要单独的"基础设施"项目。把适配器放在子目录中即可

---

## DDD 轻量替代

Domain-Driven Design 的简化版本——保留有价值的部分，去掉全套框架。

### 1. 统一语言 (Ubiquitous Language)

> 出处：Eric Evans，《领域驱动设计》（2003）

**代码术语与业务术语保持一致。**

#### 如何应用

- 如果业务说"订单状态是'已发货'"（shipped），代码应该用 `order.status == "shipped"`，而不是 `order.state == 3`
- 如果业务说"取消订单"，方法应该是 `order.cancel()`，而不是 `order.updateStatus(4)`
- 维护一个业务术语表，在代码审查中强制执行

#### 常见踩坑

- 统一语言在代码用 `status` 而业务说 `state` 时就被违反了
- 语言必须在团队中保持一致——如果业务改了术语，更新代码
- 不要在同一个上下文中混用业务术语和技术术语

---

### 2. 包隔离 (Package Isolation)

> 从限界上下文 (Bounded Context) 简化而来

**一个包的代码不应直接引用另一个包的内部实现。**

#### 如何应用

- 每个包有清晰的公共接口（导出的类型/函数）
- 内部类型是包私有的
- 跨包通信通过公共接口进行

```python
# order/__init__.py — 公共接口
from .service import OrderService
from .models import Order, OrderItem

# order/_internal.py — 私有，不导出
class OrderValidator:  # 内部细节
    ...
```

#### 常见踩坑

- 包隔离不是说"不能 import"——是"import 接口，不 import 内部实现"
- 在 Python 中，用 `_` 前缀表示内部模块。在 Go 中，用小写标识符。两种情况下，约定就是强制

---

### 3. 入口类 (Entry Class)

> 从聚合根 (Aggregate Root) 简化而来

**通过单一入口点控制对象图的访问。**

#### 如何应用

- 如果 `Order` 包含 `OrderItem`，所有对 items 的修改都通过 `Order`
- 不允许从外部直接操作 `order.items`

```python
class Order:
    def __init__(self):
        self._items = []

    def add_item(self, product, quantity):
        self._items.append(OrderItem(product, quantity))

    @property
    def items(self):
        return list(self._items)  # 返回副本，不是原对象
```

#### 常见踩坑

- 入口类不是说"一个类统治一切"——每个聚合有自己的入口
- 入口类负责**不变条件**——必须始终为真的规则（如"订单总额必须为正"）
- 不要为没有行为的简单数据创建入口类——一个纯列表对于只是 ID 列表的购物车来说就够了

---

### 4. 数据访问层 (Data Access Layer)

> 从仓库模式 (Repository Pattern) 简化而来

**将所有数据访问封装在专用层中。**

#### 如何应用

- 业务逻辑从不直接写 SQL
- 所有查询通过数据访问类进行
- 数据访问类返回领域对象，不是数据库行

```python
class OrderRepository:
    def find_by_id(self, order_id: int) -> Order: ...
    def find_by_customer(self, customer_id: int) -> list[Order]: ...
    def save(self, order: Order): ...
```

#### 常见踩坑

- 在单体中，简单的 DAO（数据访问对象）就足够了——你不需要完整的仓库模式加工作单元
- 数据访问层应返回领域对象，而不是原始数据库行。映射在层内部完成
- 不要把业务逻辑放在数据访问层——它只负责数据访问
