# Software Design Skill

A curated collection of battle-tested software design principles for monolith applications — for both AI agents and humans.

## What's inside

```
SKILL.md                    ← Navigation hub (progressive disclosure)
├── references/             ← Deep dives for each principle group
│   ├── solid.md            — SOLID: SRP, OCP, LSP, ISP, DIP
│   ├── grasp.md            — GRASP: 9 responsibility assignment patterns
│   ├── general.md          — DRY, KISS, YAGNI, Composition > Inheritance, etc.
│   ├── interface-design.md — Law of Demeter, CQS, Postel, Least Surprise
│   ├── package-design.md   — ADP, CCP, SDP
│   └── architecture.md     — Layered, Dependency Rule, Ports & Adapters, DDD lightweight
├── examples/               — Before/after code examples (coming soon)
├── scripts/                — Automation scripts (coming soon)
├── _zh/                    — Chinese version (SKILL.md + references/ only)
│   ├── SKILL.md
│   └── references/
├── README.md
├── README.zh-CN.md
└── CHANGELOG.md
```

## Principles covered

| Group | Principles |
|-------|-----------|
| **SOLID** | Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, Dependency Inversion |
| **GRASP** | Information Expert, Creator, Controller, Low Coupling, High Cohesion, Polymorphism, Pure Fabrication, Indirection, Protected Variations |
| **General** | DRY, KISS, YAGNI, Composition over Inheritance, Tell Don't Ask, Fail Fast, Separation of Concerns, Information Hiding, Single Level of Abstraction, Boy Scout Rule, Extension over Configuration |
| **Interface** | Law of Demeter, Command-Query Separation, Postel's Law, Principle of Least Surprise |
| **Package** | Acyclic Dependencies, Common Closure, Stable Dependencies |
| **Architecture** | Layered Architecture, Dependency Rule, Ports & Adapters, DDD Lightweight (Ubiquitous Language, Package Isolation, Entry Class, Data Access Layer) |

## Design philosophy

This skill follows the **progressive disclosure** pattern:

1. **SKILL.md** — Navigation hub with quick reference and gotchas
2. **references/** — Deep dives with origin, intent, recognition patterns, before/after examples, and gotchas
3. **examples/** — Runnable code examples (coming soon)
4. **scripts/** — Automation for audit and enforcement (coming soon)

Each principle includes:
- **Origin** — Who proposed it and when
- **Intent** — What problem it solves
- **Recognition** — How to spot violations
- **Before/After** — Concrete code examples
- **Gotchas** — Common mistakes even experienced developers make

## Related

- [skill-composer-se](https://github.com/hyper0x/skill-composer-se) — Software engineering workflow (TDD, refactoring, design review)

## License

MIT

## 中文版

中文版位于 [`_zh/`](_zh/) 目录（含 SKILL.md + references/），从 [`_zh/SKILL.md`](_zh/SKILL.md) 开始阅读。
