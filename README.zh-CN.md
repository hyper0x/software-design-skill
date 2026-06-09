# 软件设计原则技能

一套经过实战检验的软件设计原则集合，聚焦单体应用。

## 目录结构

```
SKILL.md                    ← 导航页（渐进式暴露）
├── references/             ← 深度参考：每组原则
│   ├── solid.md            — SOLID：SRP、OCP、LSP、ISP、DIP
│   ├── grasp.md            — GRASP：9 条职责分配模式
│   ├── general.md          — DRY、KISS、YAGNI、组合优于继承等
│   ├── interface-design.md — 得墨忒耳定律、CQS、Postel、最小惊讶
│   ├── package-design.md   — ADP、CCP、SDP
│   └── architecture.md     — 分层、依赖规则、端口与适配器、DDD 轻量替代
├── examples/               — 重构前后代码示例（即将推出）
├── scripts/                — 自动化脚本（即将推出）
├── _zh/                    — 中文版（镜像根目录结构）
│   ├── SKILL.md
│   └── references/
├── README.md
├── README.zh-CN.md
└── CHANGELOG.md
```

## 涵盖的原则

| 分组 | 原则 |
|------|------|
| **SOLID** | 单一职责、开闭、里氏替换、接口隔离、依赖反转 |
| **GRASP** | 信息专家、创建者、控制器、低耦合、高内聚、多态、纯虚构、间接、保护变化 |
| **通用** | DRY、KISS、YAGNI、组合优于继承、Tell Don't Ask、Fail Fast、关注点分离、信息隐藏、单一抽象层、童子军规则、扩展优于配置 |
| **接口** | 得墨忒耳定律、命令-查询分离、鲁棒性原则、最小惊讶原则 |
| **包/模块** | 无环依赖、共同闭包、稳定依赖 |
| **架构** | 分层架构、依赖规则、端口与适配器、DDD 轻量替代（统一语言、包隔离、入口类、数据访问层） |

## 设计理念

本技能采用**渐进式暴露**模式：

1. **SKILL.md** — 导航页，快速索引 + 踩坑提醒
2. **references/** — 深度参考，含出处、意图、识别方法、重构前后示例、踩坑
3. **examples/** — 可运行代码示例（即将推出）
4. **scripts/** — 自动化审计脚本（即将推出）

每条原则包含：
- **出处** — 谁提出的，什么时候
- **意图** — 解决什么问题
- **识别** — 如何发现违规
- **重构前后** — 具体代码示例
- **踩坑** — 老手都会犯的错

## 相关项目

- [skill-composer-se](https://github.com/hyper0x/skill-composer-se) — 软件工程工作流（TDD、重构、设计评审）

## 许可证

MIT
