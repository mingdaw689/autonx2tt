# 领域文档

工程类 skill 在探索代码库时应如何消费本仓库的领域文档。

## 探索之前，先读这些

- **`CONTEXT.md`**（位于仓库根目录），或
- **`CONTEXT-MAP.md`**（位于仓库根目录，若存在）：它指向每个上下文各自的 `CONTEXT.md`。读取与当前主题相关的每一个。
- **`docs/adr/`**：读取涉及你即将着手领域的 ADR。多上下文仓库中，还应检查 `src/<context>/docs/adr/` 下的上下文级决策。

如果这些文件不存在，**静默继续**。不要提示它们缺失，也不要主动建议预先创建。`/domain-modeling` skill（可通过 `/grill-with-docs` 与 `/improve-codebase-architecture` 到达）会在术语或决策真正被敲定时按需创建它们。

## 文件结构

单上下文仓库（绝大多数仓库）：

```
/
├── CONTEXT.md
├── docs/adr/
│   ├── 0001-event-sourced-orders.md
│   └── 0002-postgres-for-write-model.md
└── src/
```

多上下文仓库（根目录存在 `CONTEXT-MAP.md`）：

```
/
├── CONTEXT-MAP.md
├── docs/adr/                          ← 系统级决策
└── src/
    ├── ordering/
    │   ├── CONTEXT.md
    │   └── docs/adr/                  ← 上下文级决策
    └── billing/
        ├── CONTEXT.md
        └── docs/adr/
```

## 使用 glossary 的词汇

当你的输出提到某个领域概念时（issue 标题、重构方案、假设、测试名称），使用 `CONTEXT.md` 中定义的术语。不要漂移到 glossary 明确回避的同义词。

如果你需要的概念尚未出现在 glossary 中，这是一个信号：要么你在发明项目并不使用的语言（请重新考虑），要么确实存在缺口（记录下来，交给 `/domain-modeling`）。

## 标记 ADR 冲突

如果你的输出与现有 ADR 相矛盾，请显式指出，而不是静默覆盖：

> _与 ADR-0007（event-sourced orders）相矛盾，但值得重新讨论，因为……_
