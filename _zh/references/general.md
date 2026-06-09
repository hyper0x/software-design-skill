# 通用设计原则

贯穿软件设计所有层面的原则，从单个函数到整个系统都适用。

---

## DRY — 不要重复自己 (Don't Repeat Yourself)

> 出处：Andrew Hunt & David Thomas，《程序员修炼之道》（1999）

**每条知识在系统中必须有一个单一、明确、权威的表达。**

### 意图

当知识被重复时，变更必须在多个地方进行。总会有遗漏的，导致不一致。DRY 减少了 bug 的滋生面，让系统更容易变更。

### 如何识别违规

- 同一条业务规则出现在多个文件中
- 配置值在多个地方硬编码
- 前端和后端都有相同的校验逻辑
- API 契约在客户端和服务器端手动重复

### 常见踩坑

- DRY 说的是**知识**，不是代码。两段相同但服务于不同业务规则的代码不是重复
- 四种重复类型：强加的（环境迫使）、无意的（没意识到）、不耐烦的（懒得抽象）、团队间的（多人重复）
- 复述代码的注释违反 DRY——注释应解释*为什么*，不是*是什么*
- DRY 的反面是 WET："Write Everything Twice"（写两遍）或"We Enjoy Typing"（我们喜欢打字）

---

## KISS — 保持简单 (Keep It Simple, Stupid)

> 出处：Kelly Johnson（洛克希德·马丁臭鼬工厂），1960年代

**简单的系统比复杂的更好。**

### 意图

复杂性是可靠性的敌人。简单的代码更容易理解、测试、调试和修改。大多数"聪明"的解决方案制造的问题比解决的还多。

### 如何识别违规

- 一个 5 行的解决方案被替换成 50 行的"灵活"框架
- 为还不存在的问题做过度的抽象
- 深继承层次，一个简单函数就能搞定
- 为简单行为设计复杂的配置系统

### 常见踩坑

- KISS 不是说"做简单点就行"——是"不要引入偶然复杂度"
- 简单 != 容易。有时最简单的解决方案需要更多的前期思考
- 如果你需要画图才能向另一个开发者解释你的代码，那它可能不够简单

---

## YAGNI — 你不需要它 (You Ain't Gonna Need It)

> 出处：Ron Jeffries，极限编程（1999）

**不需要的功能就别做。**

### 意图

每一行代码都是负债——它需要被理解、测试和维护。添加"面向未来"但从未使用的功能是浪费精力，同时也让代码库更复杂。

### 如何识别违规

- "我们以后可能需要"的抽象
- 为还不存在的功能准备的配置开关
- 只有一个实现的通用接口
- 在性能问题被测量之前就加了缓存层

### 常见踩坑

- YAGNI 不是说"不用规划"——是"今天不需要的今天不建"
- YAGNI 不适用于难以逆转的架构决策（如选择数据库）。那些需要提前考虑
- 平衡点：构建今天需要的，但设计上不阻止明天的变更

---

## 组合优于继承 (Composition over Inheritance)

> 出处：GoF（四人帮），《设计模式》（1994）

**优先用组合来组装行为，而不是继承。**

### 意图

继承在父类和子类之间创建了紧耦合。父类的变更可能破坏所有子类。组合——对象持有对其他对象的引用并委派给它们——更灵活、更不易碎。

### 如何识别违规

- 深继承层次（3 层以上）
- 子类覆盖了父类的大部分方法
- "菱形问题"（多重继承冲突）
- 类继承了它不需要的行为

### 示例

**继承（脆弱）：**
```python
class Bird:
    def fly(self): ...

class Ostrich(Bird):
    def fly(self): raise NotImplementedError  # 问题！
```

**组合（灵活）：**
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

### 常见踩坑

- 组合优于继承是**偏好**，不是铁律。继承有合理的用途（如模板方法模式、框架基类）
- 判断标准："子类真的是父类的子类型吗？"如果是，继承可能合适。如果只是"想复用一些方法"，用组合

---

## Tell, Don't Ask（告诉，不要问）

> 出处：Andrew Hunt & David Thomas，《程序员修炼之道》（1999）

**告诉对象做什么，别问它要数据自己做。**

### 意图

过程式代码问数据然后做决策。面向对象代码告诉对象做什么，让它自己决定怎么做。这保持了行为与数据的统一。

### 如何识别违规

```python
# 违规：问数据，然后自己做决策
if employee.type == "manager":
    bonus = salary * 0.2
else:
    bonus = salary * 0.1

# 更好：告诉对象
bonus = employee.calculate_bonus(salary)
```

### 常见踩坑

- Tell Don't Ask 不是说"永远别调 getter"——是"别替能做决定的对象做决定"
- 数据对象（DTO、值对象）的界限是模糊的——它们的存在就是为了持有数据，问它们要数据没问题

---

## Fail Fast（快速失败）

> 出处：Andrew Hunt & David Thomas，《程序员修炼之道》（1999）

**让错误在发生点立即暴露。**

### 意图

一个死掉的程序比一个半死不活的程序造成的破坏小得多。静默失败会破坏数据、掩盖 bug、让调试变成噩梦。尽早失败，大声失败。

### 如何识别违规

- `except: pass`——静默吞掉异常
- 返回 `None` 或 `-1` 而不是抛出错误
- 检测到无效状态后继续执行
- 记录错误后假装什么都没发生

### 常见踩坑

- Fail Fast 说的是**开发期**安全，不是生产环境的错误处理。在生产中你可能需要优雅降级——但错误仍然应该是可见的
- 平衡点：在边界（API 入口）校验输入，而不是在每个内部函数里
- 用断言处理"绝不应该发生"的条件，用错误处理处理"可能发生"的条件

---

## 关注点分离 (Separation of Concerns)

> 出处：Edsger Dijkstra，《On the role of scientific thought》（1974）

**不同的关注点放在不同的模块。**

### 意图

每个模块应处理一个明确定义的关注点。混合关注点（如业务逻辑混在数据库代码中）会创建紧耦合，让变更变得危险。

### 如何识别违规

- 一个函数同时处理 HTTP 解析、业务逻辑和数据库访问
- 修改 UI 需要修改业务逻辑
- 业务规则散落在控制器、视图和辅助函数中

### 常见踩坑

- 关注点分离是 SRP、分层架构和包设计共同衍生的父概念
- 艺术在于知道在哪里画边界——太细会碎片化，太粗会耦合

---

## 信息隐藏 (Information Hiding)

> 出处：David Parnas，《On the Criteria to Be Used in Decomposing Systems into Modules》（1972）

**把实现细节藏在稳定接口后面。**

### 意图

模块的内部实现应对其使用者不可见。这允许实现变更而不影响调用方。信息隐藏是封装的基础。

### 如何识别违规

- 应该是私有的字段被公开
- 内部数据结构通过 getter 暴露
- 调用方依赖返回数据的内部格式
- 注释解释"怎么做"而不是"做什么"

### 常见踩坑

- 信息隐藏不等于"getter 和 setter"——返回内部状态的 getter 仍然是在暴露
- 要问的问题："如果我改变内部表示，有多少调用方会受影响？"答案应该是零

---

## 单一抽象层 (Single Level of Abstraction)

> 出处：Robert C. Martin，《代码整洁之道》（2008）

**函数内的代码应在同一抽象层级。**

### 意图

函数应该像故事一样读——高层步骤在顶部，每个步骤委派给低层辅助函数。混合抽象层级（如一个高层业务步骤旁边跟着一个低层字符串操作）让代码难以阅读。

### 示例

**坏：**
```python
def process_order(order):
    validate_order(order)  # 高层
    total = sum(item.price * item.quantity for item in order.items)  # 低层细节
    apply_discount(order, total)  # 高层
    order.status = "processed"  # 低层细节
```

**好：**
```python
def process_order(order):
    validate_order(order)
    total = calculate_total(order)
    apply_discount(order, total)
    mark_as_processed(order)
```

### 常见踩坑

- 这是**可读性**原则，不是正确性原则——两种写法都能工作，但"好"的版本更容易扫读
- 判断标准：你能在不读辅助函数的情况下理解函数体的流程吗？

---

## 童子军规则 (Boy Scout Rule)

> 出处：Andrew Hunt & David Thomas，《程序员修炼之道》（1999）

**离开时让代码比来时更干净。**

### 意图

持续的小改进防止代码腐烂。如果每个人每次接触文件时都修复一个小问题，代码库会越来越好，而不是越来越糟。

### 如何应用

- 遇到命名不好的变量就重命名
- 看到注释解释代码块就提取成方法
- 注意到死代码就删除
- 在编辑的文件中修复格式不一致

### 常见踩坑

- 童子军规则说的是**小改进**，不是大规模重构。如果变更很大，单独建 ticket
- 不要把清理和功能开发混在同一个 commit 里——不同的关注点用不同的 commit
- 这条规则适用于你已经接触的代码，不是"去找东西来清理"

---

## 扩展优于配置 (Extension over Configuration)

> 优先用新代码路径，而不是配置开关，来实现新行为。

### 意图

配置开关（`if feature_enabled`）创建了难以测试、推理和删除的隐式代码路径。为新行为添加新类或新函数更干净、更明确。

### 如何识别违规

```python
# 违规：配置开关创建了隐藏的代码路径
def process_payment(order):
    if config.new_payment_flow:
        # 新流程
    else:
        # 旧流程

# 更好：分离的代码路径
class PaymentProcessor(ABC):
    @abstractmethod
    def process(self, order): ...

class OldPaymentProcessor(PaymentProcessor): ...
class NewPaymentProcessor(PaymentProcessor): ...
```

### 常见踩坑

- 扩展优于配置不是说"永远别用配置"——是"别让配置替代好的设计"
- 配置适合：环境差异（dev/staging/prod）、用户偏好、真正可变的参数（超时、限制）
- 配置不适合：行为变更、持续数月的功能开关、创建两条并行代码路径的分支逻辑
