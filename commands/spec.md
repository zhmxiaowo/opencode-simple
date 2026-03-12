---
description: 快捷方式：只输出 SPEC 规范，不输出完整实施计划。等同于 /plan --spec-only。
---

# /spec — Spec-First 快捷方式

> `/plan` 的轻量版本 — 只输出 SPEC 部分，跳过实施步骤分解。

## 说明

`/spec` 与 `/plan` 共享同一个 **planner** agent 和同一套 Spec-First 流程（MCP 文档研究 → SPEC 输出 → 等待确认），区别仅在于：

| 命令 | 输出内容 |
|------|---------|
| `/plan` | SPEC + 实施步骤 + 风险评估（完整） |
| `/spec` | 仅 SPEC（API 设计、数据模型、依赖、编译计划） |

## 工作流程

1. **需求确认** — 提取功能名称、技术栈、核心需求
2. **MCP 文档研究** — 并行调用 `context7`（官方文档）+ `exa-web-search`（社区实践）
3. **输出 SPEC** — 生成包含以下章节的规范文档：
   - API / 接口设计（精确到可直接复制到代码）
   - 数据模型
   - 依赖项及版本
   - 编译验证计划（具体命令）
   - 已知限制 & 兼容性
   - 参考文档链接
4. **等待确认** — 必须等用户明确确认后方可编码

## 参数

- `/spec Unity PlayerController`
- `/spec Cocos 2D骨骼动画`
- `/spec UE5 Gameplay Ability System`
- `/spec React Dashboard`

## 何时用 `/spec`，何时用 `/plan`

- **小功能 / 单文件改动** → `/spec` 足够
- **多文件 / 跨模块 / 需要分阶段实施** → 用 `/plan`（包含 SPEC + 完整步骤）
