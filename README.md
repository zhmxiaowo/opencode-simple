# OpenCode Simple — CDD & Spec-First 通用编程框架

> 基于 [Everything Claude Code](https://github.com/affaan-m/everything-claude-code) 的精简 CDD 改造版

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
├── CLAUDE.md                    # AI 助手指导文件
├── opencode.jsonc               # OpenCode MCP 配置（5 个核心 MCP）
├── agents/                      # 8 个精简 Agent
│   ├── architect.md             # 架构设计
│   ├── build-error-resolver.md  # CDD 编译驱动解决引擎 ⭐
│   ├── code-reviewer.md         # 代码审查（无测试要求）
│   ├── doc-updater.md           # 文档更新
│   ├── harness-optimizer.md     # Harness 优化
│   ├── loop-operator.md         # 自主循环操作
│   ├── planner.md               # 实施规划
│   └── refactor-cleaner.md      # 重构清理
├── commands/                    # 用户命令
│   ├── spec.md                  # /spec — Spec-First 规范生成 ⭐ (新增)
│   ├── cdd.md                   # /cdd — 快速编译验证 ⭐ (新增)
│   ├── build-fix.md             # /build-fix — CDD 修复编译错误
│   ├── verify.md                # /verify — CDD 全面验证
│   ├── plan.md                  # /plan — 实施规划
│   ├── code-review.md           # /code-review — 代码审查
│   ├── learn.md                 # /learn — 提取模式
│   ├── evolve.md                # /evolve — 进化 instincts
│   └── ...                      # 会话管理、instinct 管理等
├── rules/common/                # 永远遵守的准则
│   ├── cdd-core.md              # CDD 三大铁律 ⭐ (新增)
│   ├── development-workflow.md  # CDD 化开发流程
│   ├── agents.md                # Agent 编排（已更新）
│   ├── patterns.md              # 通用模式（含 CDD/Spec 模式）
│   ├── coding-style.md          # 编码风格
│   ├── security.md              # 安全准则
│   ├── git-workflow.md          # Git 工作流
│   ├── hooks.md                 # Hooks 使用
│   └── performance.md           # 性能优化
├── skills/                      # 16 个通用 Skill
│   ├── continuous-learning-v2/  # V2 本能学习系统 ⭐ (保留)
│   ├── continuous-learning/     # V1 学习系统
│   ├── verification-loop/       # CDD 验证循环（已改造）
│   ├── blueprint/               # 蓝图规划
│   ├── search-first/            # 搜索优先
│   ├── eval-harness/            # 评估框架
│   ├── coding-standards/        # 编码标准
│   └── ...                      # 其他通用 skills
├── hooks/                       # 异步 Hooks（完整保留）
│   ├── hooks.json               # Hook 配置
│   └── README.md
├── scripts/                     # 工具脚本（完整保留）
│   ├── hooks/                   # Hook 实现脚本
│   ├── lib/                     # 共享库
│   └── ...
├── contexts/                    # 上下文模式
├── mcp-configs/                 # MCP 配置参考
└── schemas/                     # JSON Schema
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
2. **异步 Hooks** — 会话持久化、上下文防腐烂、质量门控
3. **Memory MCP** — 跨会话项目记忆
4. **Sequential Thinking** — 编译错误的强制链式推理

## 🚀 快速开始

1. 将 `opencode.jsonc` 复制到你的项目根目录或 `~/.config/opencode/`
2. 设置环境变量 `EXA_API_KEY`
3. 开始使用：`/spec [你要实现的功能]`

## 📜 从 ECC 删除的内容

| 类别 | 删除项 |
|------|--------|
| Agents | tdd-guide, e2e-runner, go-reviewer, go-build-resolver, python-reviewer, database-reviewer, security-reviewer, kotlin-reviewer, chief-of-staff |
| Commands | tdd, e2e, go-*, python-review, test-coverage, multi-*, pm2, gradle-build |
| Skills | 所有语言特定 (golang-*, cpp-*, django-*, python-*, springboot-*, java-*, jpa-*, swift-*, perl-*, kotlin-*)、业务特定 (investor-*, carrier-*, customs-*, energy-*, logistics-* 等)、TDD/E2E 相关 |
| Rules | testing.md、所有语言目录 (typescript, python, golang, swift, php, perl, kotlin) |
