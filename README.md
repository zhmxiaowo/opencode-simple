# OpenCode Simple — CDD & Spec-First 通用编程框架

> 基于 [Everything Claude Code](https://github.com/affaan-m/everything-claude-code) 的精简 CDD 改造版，适配 [OpenCode](https://opencode.ai) 规范

## 🎯 核心哲学

| 原则 | 说明 |
|------|------|
| 🚫 **绝对禁止 TDD** | 不写任何测试代码，面向 Unity/UE5/Cocos 等无 Headless Test 环境 |
| 📋 **Spec-First** | 编码前必须查阅文档并生成 SPEC.md |
| ⚙️ **编译驱动 (CDD)** | 唯一验证标准：`Exit Code 0` + `LSP 0 Error` |
| 🧠 **Think Before Fix** | 编译错误必须调用 sequential-thinking 推理，禁止盲目试错 |

## 📁 框架结构

```
opencode-simple/
├── AGENTS.md                        # OpenCode 主规则文件（项目级指令）
├── CLAUDE.md                        # Claude Code 兼容（保留）
├── opencode.jsonc                   # OpenCode 配置（MCP、instructions）
├── .opencode/                       # OpenCode 标准目录
│   ├── agents/                      # 8 个精简 Agent（子代理）
│   │   ├── architect.md             # 架构设计（只读）
│   │   ├── build-error-resolver.md  # CDD 编译驱动解决引擎 ⭐
│   │   ├── code-reviewer.md         # 代码审查（只读，无测试要求）
│   │   ├── doc-updater.md           # 文档更新
│   │   ├── harness-optimizer.md     # Harness 优化
│   │   ├── loop-operator.md         # 自主循环操作
│   │   ├── planner.md              # Spec-First 研究 + 实施规划
│   │   └── refactor-cleaner.md      # 重构清理
│   ├── commands/                    # 32 个用户命令
│   │   ├── spec.md                  # /spec — Spec-First 规范生成 ⭐
│   │   ├── plan.md                  # /plan — 实施规划 → planner agent
│   │   ├── cdd.md                   # /cdd — 快速编译验证 ⭐
│   │   ├── build-fix.md             # /build-fix → build-error-resolver agent
│   │   ├── verify.md                # /verify — CDD 全面验证
│   │   ├── code-review.md           # /code-review → code-reviewer agent
│   │   ├── learn.md                 # /learn — 提取模式
│   │   ├── evolve.md                # /evolve — 进化 instincts
│   │   └── ...                      # 会话管理、instinct 管理等
│   ├── skills/                      # 16 个通用 Skill
│   │   ├── continuous-learning-v2/  # V2 本能学习系统 ⭐
│   │   ├── continuous-learning/     # V1 学习系统
│   │   ├── verification-loop/       # CDD 验证循环（已改造）
│   │   ├── blueprint/               # 蓝图规划
│   │   ├── search-first/            # 搜索优先
│   │   ├── eval-harness/            # 评估框架
│   │   ├── coding-standards/        # 编码标准
│   │   └── ...                      # 其他通用 skills
│   ├── plugins/                     # OpenCode 插件（对应 Claude Code Hooks）
│   └── tools/                       # 自定义工具（可选）
├── rules/common/                    # 永远遵守的准则（via opencode.jsonc instructions）
│   ├── cdd-core.md                  # CDD 三大铁律 ⭐
│   ├── development-workflow.md      # CDD 化开发流程
│   ├── agents.md                    # Agent 编排
│   ├── patterns.md                  # 通用模式（含 CDD/Spec 模式）
│   ├── coding-style.md              # 编码风格
│   ├── security.md                  # 安全准则
│   ├── git-workflow.md              # Git 工作流
│   ├── hooks.md                     # Hooks 使用
│   └── performance.md               # 性能优化
├── hooks/                           # Claude Code 异步 Hooks（CC 兼容保留）
├── scripts/                         # 工具脚本（Hook 实现、CI 验证等）
├── contexts/                        # 上下文模板
├── mcp-configs/                     # MCP 配置参考文件
└── schemas/                         # JSON Schema（校验用）
```

## 🔌 核心 MCP 矩阵（仅 5 个）

| MCP | 用途 | CDD 角色 |
|-----|------|---------|
| `exa-web-search` | 全网搜索代码和报错 | Spec-First 研究 |
| `context7` | 查阅最新官方文档 | Spec-First 防幻觉 |
| `mcp-language-server` | 本地 LSP 诊断 | **CDD 核心验证** |
| `memory` | 跨会话记忆 | 本能学习支持 |
| `sequential-thinking` | 链式思考 | **编译错误推理** |

## 🔄 工作流

```
/spec [功能]  →  SPEC.md  →  用户确认  →  编码  →  /cdd  →  /verify  →  /code-review  →  ✅ 完成
```

## 🏗️ 从 ECC 保留的核心机制

1. **V2 本能学习** — 自动从会话中提取编译错误模式为 instincts
2. **异步 Hooks** — 会话持久化、上下文防腐烂、质量门控（Claude Code 兼容）
3. **Memory MCP** — 跨会话项目记忆
4. **Sequential Thinking** — 编译错误的强制链式推理

## 🚀 快速开始

1. 将整个 `opencode-simple/` 目录内容复制到你的项目根目录
2. 确保根目录包含 `opencode.jsonc`、`AGENTS.md`、`.opencode/` 目录
3. 设置环境变量 `EXA_API_KEY`
4. 开始使用：`/spec [你要实现的功能]`

### 配置说明

- **`opencode.jsonc`** — OpenCode 主配置文件。遵循 [OpenCode Config Schema](https://opencode.ai/docs/config)
  - `instructions` 字段引入 `rules/common/*.md` 作为全局准则
  - `mcp` 字段定义 MCP 服务器（使用 `type` + `command` 数组格式）
- **`AGENTS.md`** — OpenCode 主规则文件（等同于 Claude Code 的 `CLAUDE.md`）
- **`.opencode/agents/`** — 子代理定义（frontmatter 含 `description`、`mode`、`tools`）
- **`.opencode/commands/`** — 用户命令（frontmatter 含 `description`、可选 `agent`）
- **`.opencode/skills/`** — 可复用工作流知识（`SKILL.md` frontmatter 含 `name`、`description`）

## ⚠️ 兼容性说明

本框架同时保留了 Claude Code 和 OpenCode 的兼容结构：

| 特性 | OpenCode | Claude Code |
|------|----------|-------------|
| 规则文件 | `AGENTS.md` | `CLAUDE.md` |
| Agent/Command/Skill 目录 | `.opencode/` | `.claude/` |
| Hook 系统 | `.opencode/plugins/` | `hooks/hooks.json` |
| 配置文件 | `opencode.jsonc` | `CLAUDE.md` 内嵌 |

> `hooks/` 和 `scripts/hooks/` 目录使用 Claude Code 的 Hook 格式。如需在 OpenCode 中实现同等功能，需迁移到 `.opencode/plugins/` 插件系统。

## 📜 从 ECC 删除的内容

| 类别 | 删除项 |
|------|--------|
| Agents | tdd-guide, e2e-runner, go-reviewer, go-build-resolver, python-reviewer, database-reviewer, security-reviewer, kotlin-reviewer, chief-of-staff |
| Commands | tdd, e2e, go-*, python-review, test-coverage, multi-*, pm2, gradle-build |
| Skills | 所有语言特定 (golang-*, cpp-*, django-*, python-*, springboot-*, java-*, jpa-*, swift-*, perl-*, kotlin-*)、业务特定 (investor-*, carrier-*, customs-*, energy-*, logistics-* 等)、TDD/E2E 相关 |
| Rules | testing.md、所有语言目录 (typescript, python, golang, swift, php, perl, kotlin) |
