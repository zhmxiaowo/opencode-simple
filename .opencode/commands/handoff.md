---
description: 生成结构化会话交接文档，让新 session 无缝接手当前工作。比 /save-session 更完整。
---

# /handoff — 会话交接

生成一份结构化的交接文档，包含当前工作的完整上下文，使另一个 session（或未来的你）可以立即继续工作。

## 输出格式

```markdown
# Handoff: [当前任务简述]

**Date:** YYYY-MM-DD
**Branch:** [当前分支]

## 1. 当前状态
- [ ] 已完成的步骤
- [x] 正在进行的步骤
- [ ] 待完成的步骤

## 2. SPEC 状态
- SPEC 位置: [文件路径，如果有]
- SPEC 状态: DRAFT / CONFIRMED / IMPLEMENTING / DONE
- 关键设计决策: [列出]

## 3. CDD 状态
- 编译命令: [具体命令]
- 上次编译结果: PASS / FAIL
- 未解决的编译错误: [如果有，列出]
- sequential-thinking 分析结论: [如果有]

## 4. 文件变更清单
[git diff --name-only HEAD 的输出]

## 5. 关键决策记录
- [决策 1]: [原因]
- [决策 2]: [原因]

## 6. 已知风险 / 阻塞项
- [风险/阻塞 1]

## 7. 下一步行动
1. [具体的下一步]
2. [具体的下一步]
```

## 工作流程

1. **收集状态** — 读取 git status、当前分支、SPEC 文件、最近的编译结果
2. **总结决策** — 回顾本次会话做出的关键技术决策和原因
3. **输出文档** — 生成到 `docs/handoff-YYYY-MM-DD.md` 或直接输出到对话
4. **存入 memory** — 调用 `memory` MCP 持久化交接摘要

## 与 /save-session 的区别

| | `/save-session` | `/handoff` |
|---|---|---|
| 粒度 | 自动快照 | 结构化、人类可读 |
| CDD 状态 | ❌ | ✅ 编译命令 + 结果 |
| SPEC 状态 | ❌ | ✅ 位置 + 进度 |
| 决策记录 | ❌ | ✅ 关键决策 + 原因 |
| 下一步行动 | ❌ | ✅ 具体可执行 |

## 参数

- `/handoff` — 完整交接文档
- `/handoff brief` — 简要版（只输出状态 + 下一步）
