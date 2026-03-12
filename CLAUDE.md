# CLAUDE.md — OpenCode Simple (CDD & Spec-First)

本文件为 AI 编程助手提供项目指导。

## 项目概述

这是一个基于 ECC (Everything Claude Code) 架构的**纯粹 CDD（编译驱动开发）& Spec-First** 通用编程框架。

**核心哲学**：
- 🚫 **绝对禁止 TDD** — 不写任何测试代码
- 📋 **Spec 先行** — 无 SPEC.md 不编码
- ⚙️ **编译驱动** — Exit Code 0 + LSP 0 Error = 任务完成
- 🧠 **Think Before Fix** — 编译错误必须用 sequential-thinking 分析

## 架构

### 核心组件

- **agents/** — 精简的 Agent 团队（planner, architect, build-error-resolver, code-reviewer 等）
- **skills/** — 通用工作流知识（CDD 验证、蓝图规划、持续学习 V2 等）
- **commands/** — 用户命令（/spec, /plan, /build-fix, /verify, /learn 等）
- **rules/** — 永远遵守的准则（CDD 核心、编码风格、安全）
- **hooks/** — 异步 Hooks（会话持久化、上下文防腐烂、持续学习观测）
- **scripts/** — 跨平台 Node.js 工具脚本（hooks 和 setup 用）
- **mcp-configs/** — 5 个核心 MCP 配置

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
| `/verify` | CDD 验证（编译+LSP+AI slop 检查，无测试） |
| `/code-review` | 代码审查（无测试覆盖率要求） |
| `/handoff` | 结构化会话交接（CDD 状态、SPEC 进度、决策记录） |
| `/learn` | 从会话中提取可复用模式 |
| `/evolve` | 将 instincts 进化为 skills/commands |

## 工作流

```
意图分类 → /plan [功能] → SPEC + 步骤 → 用户确认 → 编码 → /build-fix → /verify → /code-review → /handoff
```

## 保留的 ECC 神级机制

1. **V2 本能学习 (Continuous Learning)** — 自动从会话中提取编译错误模式
2. **异步 Hooks** — 防止上下文腐烂，自动保存会话状态
3. **Memory MCP** — 跨会话记忆，记录项目上下文
4. **Sequential Thinking** — 编译错误时的强制链式推理

## 禁止事项

- ❌ 不要编写任何测试文件
- ❌ 不要引用 Jest/Vitest/pytest 等测试框架
- ❌ 不要以"缺少测试"为由阻塞代码审查
- ❌ 不要在没有 SPEC.md 的情况下开始编码
- ❌ 不要在编译错误时盲目试错（必须先 think）
