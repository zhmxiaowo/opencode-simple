# AGENTS.md — OpenCode Simple (CDD & Spec-First)

本文件为 OpenCode AI 编程助手提供项目指导。

## 项目概述

这是一个基于 ECC (Everything Claude Code) 架构的**纯粹 CDD（编译驱动开发）& Spec-First** 通用编程框架模板。复制到任何项目根目录后即可使用 OpenCode 进行 CDD 驱动的开发。

**核心哲学**：
- 🚫 **绝对禁止 TDD** — 不写任何测试代码
- 📋 **Spec 先行** — 无 SPEC.md 不编码
- ⚙️ **编译驱动** — Exit Code 0 + LSP 0 Error = 任务完成
- 🧠 **Think Before Fix** — 编译错误必须用 sequential-thinking 分析

## 架构

### 核心组件

- `.opencode/agents/` — 精简的 Agent 团队（planner, architect, build-error-resolver, code-reviewer 等）
- `.opencode/skills/` — 通用工作流知识（CDD 验证、蓝图规划、持续学习 V2 等）
- `.opencode/commands/` — 用户命令（/spec, /plan, /build-fix, /verify, /learn 等）
- `rules/common/` — 永远遵守的准则（CDD 核心、编码风格、安全），通过 opencode.jsonc 的 instructions 字段引入
- `hooks/` — Claude Code 兼容的异步 Hooks（会话持久化、上下文防腐烂、持续学习观测）
- `scripts/` — 跨平台 Node.js 工具脚本

### 核心 MCP 矩阵（仅 5 个）

| MCP | 用途 |
|-----|------|
| `exa-web-search` | 全网搜索代码和报错 |
| `context7` | 查阅最新官方文档防幻觉 |
| `mcp-language-server` | 本地 LSP 诊断（CDD 核心） |
| `memory` | 跨会话记忆 |
| `sequential-thinking` | 编译错误链式推理 |

## 关键命令

| 命令 | 说明 |
|------|------|
| `/plan` | **Spec-First + 实施计划**: MCP 研究 → SPEC → 步骤分解（编码前必须执行） |
| `/spec` | `/plan` 的轻量版，只输出 SPEC 不输出步骤 |
| `/build-fix` | CDD 修复编译错误 |
| `/cdd` | 快速 CDD 编译验证 |
| `/verify` | CDD 全面验证（编译+LSP+AI slop 检查，无测试） |
| `/code-review` | 代码审查（无测试覆盖率要求） |
| `/handoff` | 结构化会话交接（CDD 状态、SPEC 进度、决策记录） |
| `/learn` | 从会话中提取可复用模式 |
| `/evolve` | 将 instincts 进化为 skills/commands |

## 工作流

```
意图分类 → /plan [功能] → SPEC + 步骤 → 用户确认 → 编码 → /build-fix → /verify → /code-review → /handoff
```

## 保留的 ECC 核心机制

1. **V2 本能学习 (Continuous Learning)** — 自动从会话中提取编译错误模式
2. **Memory MCP** — 跨会话记忆，记录项目上下文
3. **Sequential Thinking** — 编译错误时的强制链式推理

## 禁止事项

- ❌ 不要编写任何测试文件
- ❌ 不要引用 Jest/Vitest/pytest 等测试框架
- ❌ 不要以"缺少测试"为由阻塞代码审查
- ❌ 不要在没有 SPEC.md 的情况下开始编码
- ❌ 不要在编译错误时盲目试错（必须先 think）

## 外部规则引用

CRITICAL: 当你遇到文件引用（如 @rules/common/cdd-core.md），按需使用 Read 工具加载它。这些规则文件与当前任务相关。

核心规则文件：
- CDD 核心准则: @rules/common/cdd-core.md
- 编码风格: @rules/common/coding-style.md
- 开发工作流: @rules/common/development-workflow.md
- Agent 编排: @rules/common/agents.md
- 通用模式: @rules/common/patterns.md
- 安全准则: @rules/common/security.md
- Git 工作流: @rules/common/git-workflow.md
- 性能优化: @rules/common/performance.md
- Hooks 使用: @rules/common/hooks.md

当需要查阅文档时，使用 `context7` MCP 工具。
当需要搜索代码示例时，使用 `exa-web-search` MCP 工具。
