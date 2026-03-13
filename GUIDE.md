# OpenCode Simple 完全实战指南

> 以「**幻域传说 (Realm of Legends)**」大型游戏官网项目为例，全方位展示框架的 32 个命令、8 个 Agent、16 个 Skill、5 个 MCP、Hook 系统的完整用法与高级技巧。

---

## 目录

- [第一章：框架全景](#第一章框架全景)
- [第二章：项目背景 — 幻域传说官网](#第二章项目背景--幻域传说官网)
- [第三章：开发全流程实战](#第三章开发全流程实战)
  - [Phase 0: 项目初始化与框架安装](#phase-0-项目初始化与框架安装)
  - [Phase 1: Spec-First — 需求研究与规范生成](#phase-1-spec-first--需求研究与规范生成)
  - [Phase 2: 编码与 CDD 验证](#phase-2-编码与-cdd-验证)
  - [Phase 3: 代码审查与质量门控](#phase-3-代码审查与质量门控)
  - [Phase 4: 会话管理与上下文优化](#phase-4-会话管理与上下文优化)
  - [Phase 5: 自主学习与模式进化](#phase-5-自主学习与模式进化)
  - [Phase 6: 多 Agent 协作与自主循环](#phase-6-多-agent-协作与自主循环)
  - [Phase 7: 文档生成与交接](#phase-7-文档生成与交接)
- [第四章：32 个命令完整速查表](#第四章32-个命令完整速查表)
- [第五章：8 个 Agent 详解](#第五章8-个-agent-详解)
- [第六章：16 个 Skill 详解](#第六章16-个-skill-详解)
- [第七章：5 个 MCP 工具详解](#第七章5-个-mcp-工具详解)
- [第八章：Hook 系统与自动化](#第八章hook-系统与自动化)
- [第九章：高级技巧与精髓](#第九章高级技巧与精髓)
- [第十章：Intent Gate 意图分类决策树](#第十章intent-gate-意图分类决策树)
- [附录 A：命令-Agent-Skill-MCP 调用关系图](#附录-a命令-agent-skill-mcp-调用关系图)
- [附录 B：常见问题 FAQ](#附录-b常见问题-faq)

---

## 第一章：框架全景

### 1.1 核心哲学 — 四大铁律

| 铁律 | 含义 | 为什么 |
|------|------|--------|
| **绝对禁止 TDD** | 不写任何测试文件 | 面向 Unity/UE5/Cocos 等无 Headless Test 环境 |
| **Spec-First** | 无 SPEC 不编码 | 先研究、先规范，杜绝 AI 幻觉 |
| **CDD 编译驱动** | `Exit Code 0` + `LSP 0 Error` = 完成 | 编译器是唯一裁判 |
| **Think Before Fix** | 编译错误必须推理 | 禁止盲目试错，用 sequential-thinking 链式分析 |

### 1.2 架构一图流

```
opencode-simple/
├── AGENTS.md                    # 主规则文件（OpenCode 读取）
├── opencode.jsonc               # 配置文件（MCP + rules 引用）
├── .opencode/
│   ├── agents/    (8 个)        # 子代理：planner, architect, build-error-resolver...
│   ├── commands/  (32 个)       # 用户命令：/plan, /spec, /cdd, /verify...
│   ├── skills/    (16 个)       # 可复用知识：CDD 验证、蓝图、学习系统...
│   └── plugins/                 # OpenCode 插件（Hook 迁移目标）
├── rules/common/  (9 个)        # 永远遵守的准则（CDD、编码风格、安全...）
├── hooks/                       # Claude Code 兼容的异步 Hook
├── scripts/                     # 工具脚本（Hook 实现、CI 校验、REPL）
├── contexts/                    # 上下文模板（dev/research/review）
├── schemas/                     # JSON Schema 校验
└── mcp-configs/                 # MCP 配置参考
```

### 1.3 工作流总览

```
意图分类(Intent Gate)
       │
       ├── implement ──→ /spec → SPEC.md → 确认 → 编码 → /cdd → /verify → /code-review → 完成
       ├── fix ────────→ 直接 /build-fix → /cdd 验证
       ├── refactor ───→ /refactor-clean → /cdd 验证
       ├── research ───→ 只用 MCP 研究，不写代码
       └── quick-edit ─→ 直接修改 → /cdd 验证
```

---

## 第二章：项目背景 — 幻域传说官网

### 2.1 项目概述

**幻域传说 (Realm of Legends)** 是一款大型多人在线 RPG 游戏。我们要为它构建一个全功能官方网站，包含：

| 模块 | 功能 |
|------|------|
| **首页** | 游戏宣传视频、特色展示、新闻轮播 |
| **新闻中心** | 游戏公告、维护通知、版本更新、活动预告 |
| **游戏资料库** | 角色图鉴、装备数据库、技能树、副本攻略 |
| **社区论坛** | 玩家讨论、攻略分享、Bug 反馈 |
| **用户中心** | 注册/登录、角色绑定、充值记录、CDK 兑换 |
| **下载中心** | 多平台客户端下载、补丁更新 |
| **客服系统** | 在线工单、FAQ、AI 客服机器人 |
| **管理后台** | 内容管理、用户管理、数据分析 |

### 2.2 技术栈

```
前端: Next.js 15 (App Router) + TypeScript + Tailwind CSS + Framer Motion
后端: Next.js API Routes + Prisma ORM + PostgreSQL
认证: NextAuth.js v5
存储: Cloudflare R2 (图片/视频CDN)
部署: Vercel + Cloudflare
```

---

## 第三章：开发全流程实战

### Phase 0: 项目初始化与框架安装

#### 步骤 1：安装框架

```bash
# 创建项目
npx create-next-app@latest realm-of-legends --typescript --tailwind --app
cd realm-of-legends

# 复制 opencode-simple 框架到项目根目录
cp -r /path/to/opencode-simple/{.opencode,rules,hooks,scripts,schemas,contexts,mcp-configs,AGENTS.md,CLAUDE.md,opencode.jsonc} .

# 设置环境变量
export EXA_API_KEY="your-exa-api-key"
```

#### 步骤 2：配置包管理器

```
你输入: /setup-pm
```

**框架内部动作**：
- 运行 `node scripts/setup-package-manager.js`
- 检测 lock 文件（package-lock.json / pnpm-lock.yaml / yarn.lock / bun.lockb）
- 输出配置到 `.claude/package-manager.json`

**输出结果**：
```
Package Manager Detection:
  Lock file found: pnpm-lock.yaml
  Setting project package manager to: pnpm

  Saved to: .claude/package-manager.json
  { "packageManager": "pnpm", "setAt": "2026-03-13T10:00:00Z" }
```

#### 步骤 3：审计框架配置

```
你输入: /harness-audit
```

**框架内部动作**：
- 调用 `harness-optimizer` Agent (subagent 模式)
- 扫描 `.opencode/` 目录：agents (8), commands (32), skills (16)
- 检查 hooks.json 配置
- 检查 MCP 配置
- 生成评分卡

**输出结果（简要）**：
```
=== Harness Audit Report ===

Category Scores (out of 70):
  Tool Coverage:      9/10  ✅ 5 MCPs configured
  Context Efficiency:  8/10  ✅ Strategic compact skill loaded
  Quality Gates:       8/10  ✅ Post-edit hooks active
  Memory Persistence:  7/10  ⚠️ Memory MCP configured but no project memory yet
  Eval Coverage:       5/10  ⚠️ No eval definitions found
  Security Guardrails: 8/10  ✅ InsAIts wrapper available
  Cost Efficiency:     7/10  ✅ Model routing command available

  Overall: 52/70 (Good)

Top 3 Actions:
  1. Run `/plan` on first feature to seed project memory
  2. Create initial eval definitions with `/eval define`
  3. Initialize project instincts with `/skill-create`
```

#### 步骤 4：从 Git 历史提取编码模式（如果是已有项目）

```
你输入: /skill-create --commits 200
```

**框架内部动作**：
- 分析最近 200 条 git commit
- 检测 commit 命名约定（conventional commits 等）
- 检测文件共变模式（哪些文件总是一起修改）
- 检测架构模式（目录结构命名规律）
- 生成 SKILL.md

**输出结果（简要）**：
```
=== Skill Creation Report ===

Analyzed: 200 commits across 45 files

Detected Patterns:
  Commit Convention: conventional commits (feat/fix/refactor/docs)
  File Co-changes:
    - src/app/api/*.ts ↔ src/lib/prisma/*.ts (87% co-change rate)
    - src/components/*.tsx ↔ src/styles/*.css (72% co-change rate)
  Architecture: Feature-based directory organization

Generated: .claude/skills/learned/realm-of-legends-patterns.md
```

---

### Phase 1: Spec-First — 需求研究与规范生成

> **这是框架最核心的环节。记住：无 Spec 不编码。**

#### 场景 1：新功能开发 — 用户认证系统

##### 1.1 完整规划（/plan）

```
你输入: /plan 实现用户注册登录系统，支持邮箱注册、手机号注册、第三方OAuth登录（微信、QQ），
       包含JWT令牌刷新、密码重置、邮箱验证、防暴力破解
```

**框架内部动作（极其详细）**：

```
┌─────────────────────────────────────────────────────────┐
│  /plan 命令                                              │
│                                                          │
│  1. 触发 planner Agent (subagent 模式, 只读权限)          │
│     ├── tools: write=false, edit=false, bash=false       │
│     └── 只能读取代码 + 产出分析报告，不能修改任何文件       │
│                                                          │
│  2. planner Agent 执行 Spec-First 流程:                  │
│     ├── Step A: 并行调用 2 个 MCP 研究                    │
│     │   ├── context7 MCP → 查阅 NextAuth.js v5 官方文档   │
│     │   │   调用: context7_resolve-library-id("nextauth") │
│     │   │   调用: context7_query-docs("/nextauth/...",     │
│     │   │         "JWT refresh, OAuth providers setup")   │
│     │   │                                                │
│     │   └── exa-web-search MCP → 搜索最佳实践             │
│     │       调用: exa_web_search("NextAuth.js v5 WeChat   │
│     │             QQ OAuth provider JWT refresh 2024")    │
│     │                                                    │
│     ├── Step B: 读取现有代码结构                          │
│     │   ├── 读取 package.json 确认依赖                   │
│     │   ├── 读取 src/app/ 目录结构                       │
│     │   └── 读取 prisma/schema.prisma 数据模型           │
│     │                                                    │
│     ├── Step C: 生成 SPEC 规范                           │
│     │   包含: API 设计、数据模型、依赖、编译验证计划       │
│     │                                                    │
│     └── Step D: 生成实施步骤                             │
│         包含: 分阶段步骤 + depends-on 依赖图              │
│                                                          │
│  3. 输出 SPEC + 实施计划                                 │
│  4. ⏸️ 等待用户确认（绝不自动开始编码）                    │
└─────────────────────────────────────────────────────────┘
```

**输出结果（SPEC + Plan）**：

```markdown
# SPEC: 用户认证系统

## Tech Stack
- NextAuth.js v5 (Auth.js)
- Prisma ORM + PostgreSQL
- bcrypt (密码哈希)
- rate-limiter-flexible (防暴力破解)
- nodemailer (邮件验证)

## API 设计

### POST /api/auth/register
Request:
  { email: string, password: string, nickname: string }
Response:
  { success: true, data: { userId: string, verificationSent: boolean } }
  | { success: false, error: { code: "EMAIL_EXISTS", message: string } }

### POST /api/auth/login
Request:
  { email: string, password: string }
Response:
  { success: true, data: { accessToken: string, refreshToken: string, expiresIn: 3600 } }
  | { success: false, error: { code: "INVALID_CREDENTIALS" | "ACCOUNT_LOCKED", message: string } }

### POST /api/auth/refresh
Request:
  { refreshToken: string }
Response:
  { success: true, data: { accessToken: string, expiresIn: 3600 } }

### POST /api/auth/reset-password
### POST /api/auth/verify-email
### GET  /api/auth/oauth/[provider]  (wechat | qq | google)

## 数据模型

model User {
  id            String    @id @default(cuid())
  email         String?   @unique
  phone         String?   @unique
  passwordHash  String?
  nickname      String
  avatar        String?
  emailVerified DateTime?
  accounts      Account[]
  sessions      Session[]
  loginAttempts LoginAttempt[]
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
}

model LoginAttempt {
  id        String   @id @default(cuid())
  userId    String?
  ip        String
  success   Boolean
  createdAt DateTime @default(now())
}

## 依赖

| 包名 | 版本 | 用途 |
|------|------|------|
| next-auth | ^5.0.0 | 认证框架 |
| @prisma/client | ^6.0.0 | ORM |
| bcrypt | ^5.1.0 | 密码哈希 |
| rate-limiter-flexible | ^5.0.0 | 限流 |
| nodemailer | ^6.9.0 | 邮件发送 |

## 编译验证计划
  编译命令: npx tsc --noEmit
  LSP 检查: TypeScript Language Server
  预期 0 Error, 0 Warning

## 已知限制
- 微信 OAuth 需要企业认证（开发环境用测试号）
- QQ OAuth SDK 不支持 TypeScript 类型（需手动声明）
- Rate Limiter 单实例模式，多实例部署需改用 Redis

## 参考文档
- https://authjs.dev/getting-started/installation
- https://www.prisma.io/docs/concepts/components/prisma-schema
- https://open.weixin.qq.com/cgi-bin/showdocument?action=dir_list

---

## 实施计划

### Phase 1: 最小可用 (MVP)
Step 1: 数据库模型 [depends-on: none]
  - 文件: prisma/schema.prisma
  - 添加 User, Account, Session, LoginAttempt 模型
  - 运行 prisma migrate dev

Step 2: NextAuth 配置 [depends-on: Step 1]
  - 文件: src/lib/auth.ts, src/app/api/auth/[...nextauth]/route.ts
  - 配置 Credentials Provider (邮箱+密码)
  - 配置 JWT 策略和 token 刷新

Step 3: 注册 API [depends-on: Step 1]
  - 文件: src/app/api/auth/register/route.ts
  - 输入验证 (zod)
  - 密码哈希 (bcrypt)
  - 邮箱唯一性检查

### Phase 2: 核心体验
Step 4: 登录页面组件 [depends-on: Step 2]
  - 文件: src/app/(auth)/login/page.tsx, src/components/auth/LoginForm.tsx

Step 5: 注册页面组件 [depends-on: Step 3]
  - 文件: src/app/(auth)/register/page.tsx, src/components/auth/RegisterForm.tsx

Step 6: 防暴力破解 [depends-on: Step 2, Step 3]
  - 文件: src/lib/rate-limiter.ts, src/middleware.ts

### Phase 3: 扩展功能
Step 7: 邮箱验证 [depends-on: Step 3]
Step 8: 密码重置 [depends-on: Step 2]
Step 9: OAuth - 微信/QQ [depends-on: Step 2]

### Phase 4: 优化
Step 10: Token 自动刷新中间件 [depends-on: Step 2]
Step 11: 登录日志与异常检测 [depends-on: Step 6]

## 风险与缓解
| 风险 | 严重度 | 缓解措施 |
|------|--------|---------|
| 微信 SDK 无 TS 类型 | MEDIUM | 手写 .d.ts 声明文件 |
| JWT 密钥泄露 | HIGH | 使用环境变量，启用密钥轮转 |
| 并发注册竞态条件 | MEDIUM | 数据库唯一约束 + 应用层检查 |
```

> **关键时刻：AI 暂停，等你确认。**

```
你输入: 确认，开始编码。Phase 1 的 Step 1-3 可以并行做。
```

##### 1.2 轻量规范（/spec）— 适合小功能

```
你输入: /spec CDK 兑换码系统
```

**框架内部动作**：
```
┌────────────────────────────────────────────────────┐
│  /spec 命令 = /plan --spec-only                     │
│                                                     │
│  1. 同样触发 planner Agent (只读)                    │
│  2. 同样调用 context7 + exa-web-search MCP 研究      │
│  3. 只输出 SPEC 部分，不输出实施步骤                  │
│  4. 同样等待用户确认                                 │
└────────────────────────────────────────────────────┘
```

**输出**：只有 SPEC（API 设计 + 数据模型 + 依赖 + 编译计划），没有 Phase 1/2/3/4 步骤分解。

**什么时候用 /spec 而不是 /plan？**

| 场景 | 用 /spec | 用 /plan |
|------|---------|---------|
| 单个 API 端点 | ✅ | |
| 一个组件 | ✅ | |
| 跨多文件的功能 | | ✅ |
| 需要分阶段交付 | | ✅ |
| 架构级变更 | | ✅ |

---

### Phase 2: 编码与 CDD 验证

#### 场景 2：编码后立即验证

编码完成后，你需要验证代码是否通过编译。

##### 2.1 快速编译验证（/cdd）

```
你输入: /cdd
```

**框架内部动作**：
```
┌────────────────────────────────────────────────────┐
│  /cdd 命令                                          │
│                                                     │
│  1. 不调用 Agent，直接执行                           │
│  2. 自动检测项目类型:                                │
│     ├── 扫描 tsconfig.json → 发现 TypeScript         │
│     └── 确定编译命令: npx tsc --noEmit              │
│  3. 运行编译命令                                     │
│  4. 解析输出:                                        │
│     ├── PASS: Exit Code 0 → ✅ 报告通过              │
│     └── FAIL: Exit Code 1 → ❌ 建议运行 /build-fix   │
└────────────────────────────────────────────────────┘
```

**输出（通过时）**：
```
=== CDD Quick Check ===
Project Type: TypeScript (Next.js)
Command:      npx tsc --noEmit
Exit Code:    0
Errors:       0
Warnings:     2 (non-blocking)

Result: ✅ PASS
```

**输出（失败时）**：
```
=== CDD Quick Check ===
Project Type: TypeScript (Next.js)
Command:      npx tsc --noEmit
Exit Code:    1
Errors:       3

  src/lib/auth.ts(42,5): error TS2345: Argument of type 'string' is not
    assignable to parameter of type 'AuthConfig'.
  src/app/api/auth/register/route.ts(15,3): error TS2304: Cannot find name
    'hashPassword'.
  src/components/auth/LoginForm.tsx(28,11): error TS7006: Parameter 'e'
    implicitly has an 'any' type.

Result: ❌ FAIL
Suggestion: Run /build-fix to enter CDD repair loop
```

##### 2.2 编译错误修复（/build-fix）— CDD 核心

```
你输入: /build-fix
```

**框架内部动作（极其详细）**：
```
┌──────────────────────────────────────────────────────────┐
│  /build-fix 命令                                          │
│                                                           │
│  1. 触发 build-error-resolver Agent (subagent, 完整权限)   │
│     ├── tools: write=true, edit=true, bash=true           │
│     └── 这是唯一拥有"写+编辑+执行"全部权限的修复 Agent     │
│                                                           │
│  2. Agent 执行 CDD 修复循环:                               │
│                                                           │
│     Round 1:                                              │
│     ├── a. 运行 npx tsc --noEmit，捕获 3 个错误           │
│     ├── b. 按文件分组，按依赖排序                          │
│     ├── c. 取第一个错误: TS2345 in auth.ts:42              │
│     ├── d. 读取 auth.ts 完整文件                          │
│     ├── e. ⚡ 强制调用 sequential-thinking MCP ⚡          │
│     │   ┌──────────────────────────────────────────────┐  │
│     │   │ sequential-thinking 链式推理:                  │  │
│     │   │                                               │  │
│     │   │ Thought 1: 错误信息分析                       │  │
│     │   │   TS2345 说 string 不能赋给 AuthConfig        │  │
│     │   │   位置在 auth.ts 第 42 行                     │  │
│     │   │                                               │  │
│     │   │ Thought 2: 根因定位                           │  │
│     │   │   查看第 42 行: NextAuth(process.env.SECRET)  │  │
│     │   │   NextAuth() 期望 AuthConfig 对象，不是字符串  │  │
│     │   │   应该传入配置对象，密钥放在 secret 字段       │  │
│     │   │                                               │  │
│     │   │ Thought 3: 修复方案                           │  │
│     │   │   将 NextAuth(secret) 改为                    │  │
│     │   │   NextAuth({ secret, providers: [...] })      │  │
│     │   │   影响范围: 仅此一行，不影响其他文件           │  │
│     │   └──────────────────────────────────────────────┘  │
│     ├── f. 应用最小修复 (修改 < 5% 文件)                  │
│     └── g. 重新编译验证                                   │
│                                                           │
│     Round 2: (如果还有错误)                                │
│     ├── 处理 TS2304: Cannot find name 'hashPassword'     │
│     ├── sequential-thinking 分析:                         │
│     │   → 函数未导入，需要从 src/lib/crypto.ts 导入       │
│     ├── 添加 import { hashPassword } from '@/lib/crypto' │
│     └── 重新编译                                          │
│                                                           │
│     Round 3:                                              │
│     ├── 处理 TS7006: 'e' implicitly has 'any'            │
│     ├── sequential-thinking 分析:                         │
│     │   → 事件处理器参数缺少类型注解                      │
│     ├── 改为 (e: React.FormEvent<HTMLFormElement>)         │
│     └── 重新编译 → Exit Code 0 ✅                         │
│                                                           │
│  3. 输出修复报告                                          │
│                                                           │
│  ⚠️ 安全护栏:                                             │
│  - 如果修复引入了更多错误 → 回滚并报告                     │
│  - 如果同一错误 3 轮后仍存在 → 停止并报告                  │
│  - 如果需要架构变更 → 停止，建议用 /plan                   │
└──────────────────────────────────────────────────────────┘
```

**输出结果**：
```
=== CDD Fix Report ===

Rounds: 3
Errors Fixed: 3/3
New Errors Introduced: 0

Fixed:
  ✅ TS2345 auth.ts:42 — AuthConfig type mismatch (config object structure)
  ✅ TS2304 register/route.ts:15 — Missing import for hashPassword
  ✅ TS7006 LoginForm.tsx:28 — Missing type annotation for event handler

Final Compile: ✅ Exit Code 0 | 0 Errors | 0 Warnings
```

##### 2.3 全面验证（/verify）

```
你输入: /verify
```

**框架内部动作**：
```
┌───────────────────────────────────────────────────────┐
│  /verify 命令                                          │
│                                                        │
│  1. 不直接调用 Agent，但编译失败时自动触发              │
│     build-error-resolver                               │
│                                                        │
│  2. 按严格顺序执行 6 项检查:                            │
│                                                        │
│     Check 1: Build/Compile ─────────────────────────── │
│       运行 npx tsc --noEmit                            │
│       ├── PASS → 继续                                  │
│       └── FAIL → 调用 sequential-thinking 分析          │
│              → 调用 build-error-resolver Agent 修复     │
│              → 修复后重新验证                           │
│                                                        │
│     Check 2: Type/LSP ──────────────────────────────── │
│       使用 mcp-language-server 检查 LSP 诊断            │
│       确保 0 个 Error 级别诊断                          │
│                                                        │
│     Check 3: Lint ──────────────────────────────────── │
│       运行 next lint (如果配置了)                       │
│                                                        │
│     Check 4: Console.log/Debug Audit ───────────────── │
│       搜索 src/ 目录下的 console.log 语句              │
│       标记为需要清理（非阻塞）                          │
│                                                        │
│     Check 5: AI Slop Audit ─────────────────────────── │
│       搜索: "// TODO: implement", "placeholder",       │
│       空函数体, 不可达代码, 未使用变量                  │
│                                                        │
│     Check 6: Git Status ────────────────────────────── │
│       显示未提交的更改                                  │
└───────────────────────────────────────────────────────┘
```

**输出结果**：
```
=== CDD Verification Report ===

  Build/Compile:      ✅ PASS (Exit Code 0)
  Type/LSP:           ✅ PASS (0 Errors, 0 Warnings)
  Lint:               ✅ PASS (0 errors, 3 warnings)
  Debug Audit:        ⚠️ WARN (2 console.log found)
    - src/lib/auth.ts:55 — console.log("token refreshed")
    - src/app/api/auth/register/route.ts:32 — console.log(user)
  AI Slop:            ✅ PASS
  Git Status:         5 files modified, 3 files added

  Verdict: ✅ Ready for Code Review (clean up console.logs before PR)
```

##### 2.4 验证变体

```
/verify quick       # 只检查编译 + 类型（最快）
/verify full        # 全部 6 项检查（默认）
/verify pre-commit  # 提交前检查（编译 + lint + debug audit）
/verify pre-pr      # PR 前检查（全部 + 安全扫描）

/cdd               # = /verify quick 的快捷方式
/cdd full           # = /verify full
```

---

### Phase 3: 代码审查与质量门控

#### 场景 3：代码审查

```
你输入: /code-review
```

**框架内部动作**：
```
┌───────────────────────────────────────────────────────────┐
│  /code-review 命令                                         │
│                                                            │
│  1. 触发 code-reviewer Agent (subagent, 只读权限)           │
│     ├── tools: write=false, edit=false                     │
│     └── 可以运行 bash (用于 git diff/log)                  │
│                                                            │
│  2. Agent 执行审查流程:                                     │
│                                                            │
│     a. 收集变更:                                           │
│        git diff --name-only HEAD                           │
│        → 8 files changed                                   │
│                                                            │
│     b. 逐文件审查 (置信度 > 80% 才报告):                    │
│                                                            │
│        ┌─ CRITICAL (安全) ──────────────────────────┐      │
│        │ • 硬编码凭证                                │      │
│        │ • SQL 注入                                  │      │
│        │ • XSS                                       │      │
│        │ • 路径遍历                                  │      │
│        │ • CSRF                                      │      │
│        │ • 认证绕过                                  │      │
│        └────────────────────────────────────────────┘      │
│                                                            │
│        ┌─ HIGH (代码质量) ──────────────────────────┐      │
│        │ • 函数 > 50 行                              │      │
│        │ • 文件 > 800 行                             │      │
│        │ • 嵌套 > 4 层                               │      │
│        │ • 缺少错误处理                              │      │
│        │ • console.log                               │      │
│        │ • 缺少 SPEC.md (新功能)                     │      │
│        └────────────────────────────────────────────┘      │
│                                                            │
│        ┌─ MEDIUM (最佳实践) ───────────────────────┐       │
│        │ • 可变数据模式 (mutation)                   │       │
│        │ • AI Slop (占位注释、空函数体)              │       │
│        │ • 可访问性问题                              │       │
│        └────────────────────────────────────────────┘      │
│                                                            │
│     c. 生成报告 + 判定                                      │
│                                                            │
│  ⚠️ 重要：代码审查中绝不以"缺少测试"为由阻塞！              │
│     这是 CDD 框架的铁律。                                   │
└───────────────────────────────────────────────────────────┘
```

**输出结果**：
```
=== Code Review Report ===

Reviewed: 8 files (342 lines changed)

CRITICAL (0 issues) ✅

HIGH (2 issues):
  1. [HIGH] src/lib/auth.ts:88 — Function `refreshToken` is 63 lines
     Suggestion: Extract token validation into separate function
  2. [HIGH] src/app/api/auth/register/route.ts:15-48 — Missing error handling
     for Prisma unique constraint violation
     Suggestion: Wrap in try/catch, return 409 for duplicate email

MEDIUM (1 issue):
  3. [MEDIUM] src/lib/rate-limiter.ts:22 — Mutable state pattern
     `attempts[ip] += 1` → Use immutable Map update

LOW (1 issue):
  4. [LOW] src/components/auth/LoginForm.tsx:5 — Missing JSDoc for component

Verdict: ⚠️ WARNING (2 HIGH issues — fix before merge)

Note: No test coverage requirements (CDD project — compile is the sole truth)
```

#### 场景 4：手动质量门控

```
你输入: /quality-gate src/lib/auth.ts --strict
```

**框架内部动作**：
- 不调用 Agent，直接运行质量检查管线
- 检测语言/工具链（TypeScript → Biome/Prettier + ESLint）
- 运行格式检查 + 类型检查 + lint
- `--strict` 模式：warning 也算失败

**输出结果**：
```
=== Quality Gate: src/lib/auth.ts ===

Formatter (Biome):  ✅ PASS
TypeCheck (tsc):    ✅ PASS
Lint (ESLint):      ⚠️ 2 warnings
  - Line 55: no-console (console.log)
  - Line 88: max-lines-per-function (63 > 50)

Strict Mode: ❌ FAIL (warnings treated as errors in strict mode)
```

---

### Phase 4: 会话管理与上下文优化

> **这是高级用法的核心。掌握上下文管理 = 掌握 AI 编程的精髓。**

#### 场景 5：保存会话状态

当你做到一半需要休息，或者上下文快满了：

```
你输入: /save-session
```

**框架内部动作**：
```
┌───────────────────────────────────────────────────────────┐
│  /save-session 命令                                        │
│                                                            │
│  1. 不调用 Agent，直接收集状态                              │
│  2. 收集信息:                                               │
│     ├── git diff → 文件变更列表                             │
│     ├── 对话历史 → 关键决策                                 │
│     ├── 遇到的错误 → 记录失败方案                           │
│     └── 编译状态 → 最后一次 CDD 结果                       │
│  3. 写入 ~/.claude/sessions/2026-03-13-abc123-session.tmp  │
│  4. 显示完整内容供确认                                      │
└───────────────────────────────────────────────────────────┘
```

**输出文件内容（简要）**：
```markdown
# Session: realm-of-legends / 2026-03-13

## What We Are Building
用户认证系统（邮箱注册、OAuth、JWT刷新、防暴力破解）

## What WORKED ✅
- Prisma schema 设计完成并成功迁移
- NextAuth v5 Credentials Provider 配置成功
- 注册 API 通过 CDD 验证

## What Did NOT Work ❌
- 尝试用 jose 库做 JWT 手动签名 → 与 NextAuth 内置 JWT 冲突
  原因: NextAuth v5 已内置 JWT 处理，不需要额外库

## Current State of Files
| File | Status |
|------|--------|
| prisma/schema.prisma | ✅ Complete |
| src/lib/auth.ts | ✅ Complete |
| src/app/api/auth/register/route.ts | ✅ Complete |
| src/components/auth/LoginForm.tsx | 🔧 In Progress |
| src/lib/rate-limiter.ts | ❌ Not Started |

## Decisions Made
1. 使用 NextAuth v5 内置 JWT 而非 jose 手动管理
2. Rate limiter 使用内存模式（单实例够用）

## Exact Next Step
完成 LoginForm.tsx 的表单验证逻辑，然后实现 rate-limiter.ts
```

#### 场景 6：恢复会话

新的一天，继续工作：

```
你输入: /resume-session
```

**框架内部动作**：
```
┌───────────────────────────────────────────────────────────┐
│  /resume-session 命令                                      │
│                                                            │
│  1. 查找最新会话文件:                                       │
│     ~/.claude/sessions/2026-03-13-abc123-session.tmp       │
│  2. 读取完整内容                                            │
│  3. 生成结构化简报:                                         │
│     ├── 项目名称                                           │
│     ├── 正在做什么                                         │
│     ├── 当前状态（已完成/进行中/未开始）                     │
│     ├── 不要重试的方案（关键！）                            │
│     ├── 阻塞问题                                           │
│     └── 下一步操作                                         │
│  4. ⏸️ 等待用户指令（不会自动开始工作）                     │
└───────────────────────────────────────────────────────────┘
```

**其他会话管理命令**：
```
/sessions list                    # 列出所有历史会话
/sessions list --date 2026-03-12  # 查看特定日期的会话
/sessions load abc123             # 加载特定会话
/sessions alias abc123 auth-v1    # 给会话取别名
/sessions info auth-v1            # 查看会话详细统计
```

#### 场景 7：创建检查点

在关键节点创建快照：

```
你输入: /checkpoint create auth-mvp-done
```

**框架内部动作**：
- 运行 `/verify quick`（编译检查）
- 创建 git stash 或 commit 快照
- 记录到 `.claude/checkpoints.log`

```
你输入: /checkpoint verify auth-mvp-done   # 对比当前状态与检查点
你输入: /checkpoint list                    # 列出所有检查点
```

#### 场景 8：上下文压缩（Strategic Compact）— 高级技巧

> **何时压缩上下文？这是最容易被忽视但最重要的技巧。**

**自动提醒机制**：
框架内置 `suggest-compact.js` Hook，在以下时机提醒你：
- 工具调用次数超过 50 次（可通过 `COMPACT_THRESHOLD` 环境变量配置）
- 之后每 25 次调用再次提醒

**手动压缩决策指南**：

| 时机 | 应该压缩？ | 原因 |
|------|-----------|------|
| 研究阶段 → 开始写代码 | ✅ 是 | 研究细节不再需要，释放上下文给代码 |
| 规划完成 → 开始实施 | ✅ 是 | SPEC 已写入文件，AI 可以重新读取 |
| 调试结束 → 开始新功能 | ✅ 是 | 调试上下文与新功能无关 |
| 尝试失败 → 换方案 | ✅ 是 | 旧方案的细节会干扰新方案 |
| 正在编码中途 | ❌ 否 | 中途压缩会丢失当前工作上下文 |
| 正在修复关联错误 | ❌ 否 | 错误之间的关联信息会丢失 |

**压缩后什么会保留？什么会丢失？**

| 保留 ✅ | 丢失 ❌ |
|---------|---------|
| AGENTS.md 规则 | 中间推理过程 |
| TodoWrite 任务列表 | 已读文件的内容缓存 |
| Memory MCP 存储 | 对话上下文细节 |
| Git 状态和文件 | 工具调用历史 |
| 磁盘上的所有文件 | 会话中的临时分析 |

**最佳实践：压缩前先保存**：
```
你输入: /save-session          # 先保存完整状态
你输入: /compact               # 然后压缩上下文 (OpenCode 内置命令)
你输入: /resume-session        # 压缩后加载关键上下文
```

---

### Phase 5: 自主学习与模式进化

> **框架最"智能"的部分：让 AI 从你的编码习惯中学习。**

#### 场景 9：提取学习模式（/learn）

经过几轮 CDD 修复循环后，你发现了一些反复出现的模式：

```
你输入: /learn
```

**框架内部动作**：
```
┌────────────────────────────────────────────────────────────┐
│  /learn 命令                                                │
│                                                             │
│  1. 不调用 Agent，直接分析当前会话                           │
│  2. 扫描会话中的 4 类模式:                                   │
│     ├── 错误解决模式:                                       │
│     │   "NextAuth v5 的 JWT 回调签名变了,                    │
│     │    jwt({ token, user }) 不再接收 account 参数"         │
│     │   → 可复用性: HIGH (所有 NextAuth v5 项目)             │
│     │                                                       │
│     ├── 调试技巧:                                           │
│     │   "Prisma migrate dev 失败时先检查 DATABASE_URL 格式"  │
│     │   → 可复用性: HIGH                                     │
│     │                                                       │
│     ├── 变通方案:                                           │
│     │   "微信 OAuth 没有 TypeScript 类型, 需要手写 .d.ts"    │
│     │   → 可复用性: MEDIUM (微信相关项目)                     │
│     │                                                       │
│     └── 项目特定模式:                                       │
│         "本项目的 API 路由都用 Zod schema 验证请求体"         │
│         → 可复用性: LOW (仅本项目)                            │
│                                                             │
│  3. 生成 Skill 文件草稿                                      │
│  4. 询问用户确认后保存到:                                     │
│     ~/.claude/skills/learned/nextauth-v5-jwt-pattern.md     │
└────────────────────────────────────────────────────────────┘
```

**输出的 Skill 文件**：
```markdown
# NextAuth v5 JWT Callback Pattern

## Problem
NextAuth v5 的 jwt 回调函数签名与 v4 不同，不再接收 `account` 参数，
导致 TypeScript 编译错误 TS2345。

## Solution
使用新的回调签名:
```typescript
callbacks: {
  jwt({ token, user }) {  // 不是 jwt({ token, user, account })
    if (user) {
      token.id = user.id
      token.role = user.role
    }
    return token
  }
}
```

## When to Use
- 所有使用 NextAuth v5 (Auth.js) 的项目
- 从 NextAuth v4 迁移到 v5 时
```

#### 场景 10：带质量门控的学习（/learn-eval）

```
你输入: /learn-eval
```

**框架内部动作**：
- 执行与 `/learn` 相同的模式提取
- **额外步骤**：运行质量门控检查清单：
  - grep 搜索已有 skills 中是否已有重叠
  - 检查 MEMORY.md 中是否已记录
  - 判断应该新建还是追加到现有 skill
  - 确认跨项目可复用性
- 输出判定：**Save** / **Improve then Save** / **Absorb into [X]** / **Drop**
- 自动决定保存位置：
  - 全局 `~/.claude/skills/learned/`（跨项目通用模式）
  - 项目级 `.claude/skills/learned/`（项目特有模式）

#### 场景 11：查看学习状态

```
你输入: /instinct-status
```

**框架内部动作**：
- 运行 `instinct-cli.py status`
- 读取项目 instincts 和全局 instincts
- 合并显示

**输出结果**：
```
=== Instinct Status ===

Project: realm-of-legends (ID: a3f8c2)
  Project instincts: 12
  Global instincts:  8
  Total: 20

Domain: typescript (5 instincts)
  ████████░░ 0.82  NextAuth v5 JWT callback signature
  ███████░░░ 0.75  Prisma unique constraint error handling
  ██████░░░░ 0.65  Zod schema validation pattern
  █████░░░░░ 0.55  Next.js App Router metadata export
  ████░░░░░░ 0.45  Tailwind dynamic class generation

Domain: security (3 instincts)
  █████████░ 0.91  Rate limiter for auth endpoints
  ████████░░ 0.80  Environment variable validation
  ███████░░░ 0.72  CORS configuration for API routes

Domain: workflow (4 instincts)
  ████████░░ 0.83  CDD fix loop for TS2345 errors
  ███████░░░ 0.70  Prisma migration troubleshooting
  ██████░░░░ 0.60  Git commit message convention
  █████░░░░░ 0.50  Pre-PR checklist sequence
```

#### 场景 12：进化 — 将本能升级为技能/命令/Agent

```
你输入: /evolve
```

**框架内部动作**：
```
┌────────────────────────────────────────────────────────────┐
│  /evolve 命令                                               │
│                                                             │
│  1. 运行 instinct-cli.py evolve                             │
│  2. 分析所有 instincts，聚类相关模式:                        │
│                                                             │
│     Cluster A: "NextAuth 相关模式" (3 instincts, avg 0.76)  │
│       → 建议进化为: Skill (自动触发的行为)                    │
│       → 理由: 这些模式在遇到 NextAuth 代码时应自动应用        │
│                                                             │
│     Cluster B: "CDD 错误修复模式" (4 instincts, avg 0.72)   │
│       → 建议进化为: 追加到现有 build-error-resolver Agent    │
│       → 理由: 这些是编译错误的快速查表                        │
│                                                             │
│     Cluster C: "项目工作流模式" (3 instincts, avg 0.64)      │
│       → 建议进化为: Command (用户可调用的序列)                │
│       → 理由: 这是可重复的操作序列                            │
│                                                             │
│  3. 输出建议报告                                             │
│  4. 使用 --generate 标志可自动生成进化文件                    │
└────────────────────────────────────────────────────────────┘
```

```
你输入: /evolve --generate   # 自动生成进化后的文件
```

#### 场景 13：跨项目迁移本能

```
# 导出当前项目的高置信度 instincts
你输入: /instinct-export --min-confidence 0.8 --output realm-instincts.yaml

# 在另一个项目中导入
你输入: /instinct-import realm-instincts.yaml --scope project

# 将在多个项目中出现的 instinct 提升为全局
你输入: /promote                     # 自动检测候选
你输入: /promote --dry-run           # 预览不执行
你输入: /promote instinct-abc123     # 提升特定 instinct

# 查看所有项目和它们的 instinct 统计
你输入: /projects
```

---

### Phase 6: 多 Agent 协作与自主循环

> **框架的最高级用法：多个 AI Agent 并行协作。**

#### 场景 14：多 Agent 编排（/orchestrate）

当你需要一个完整的功能开发流程自动化：

```
你输入: /orchestrate feature 实现游戏资料库的角色图鉴系统，包含角色列表、
       筛选、详情页、3D 模型预览
```

**框架内部动作**：
```
┌─────────────────────────────────────────────────────────────┐
│  /orchestrate feature 命令                                   │
│                                                              │
│  按顺序协调 4 个 Agent，每个 Agent 的输出作为下一个的输入:    │
│                                                              │
│  Agent 1: planner (只读)                                    │
│  ├── 调用 context7 + exa-web-search 研究                     │
│  ├── 输出 SPEC + 实施计划                                    │
│  └── 传递给下一个 Agent ─────────────────┐                   │
│                                           │                  │
│  Agent 2: code-reviewer (只读)       ◄────┘                  │
│  ├── 审查计划的架构合理性                                     │
│  ├── 标记潜在安全问题                                        │
│  └── 传递审查意见 ───────────────────┐                       │
│                                       │                      │
│  Agent 3: architect (只读)       ◄────┘                      │
│  ├── 评估系统设计                                            │
│  ├── 检查可扩展性                                            │
│  └── 生成 ADR (架构决策记录) ────┐                           │
│                                   │                          │
│  最终输出: 编排报告               ◄┘                          │
│  ├── 所有 Agent 的汇总                                       │
│  ├── 文件变更清单                                            │
│  ├── 安全状态                                                │
│  └── 判定: Ship / Needs Work / Blocked                       │
└─────────────────────────────────────────────────────────────┘
```

**其他编排模式**：
```
/orchestrate bugfix 登录页面在移动端点击注册按钮无响应
/orchestrate refactor 将所有 API 路由统一使用 Repository 模式
/orchestrate security 全站安全审查
/orchestrate custom "planner,architect,code-reviewer" 设计微服务拆分方案
```

#### 场景 15：自主循环（/loop-start）

当你有一个大批量的重复性任务（如修复 50 个 lint 警告）：

```
你输入: /loop-start sequential --mode safe
```

**框架内部动作**：
```
┌─────────────────────────────────────────────────────────────┐
│  /loop-start 命令                                            │
│                                                              │
│  1. 触发 loop-operator Agent                                 │
│                                                              │
│  2. 安全检查:                                                │
│     ├── 编译通过？ ✅                                        │
│     ├── Hook profiles 启用？ ✅                               │
│     ├── 回滚路径存在？ ✅ (git stash)                        │
│     └── 明确的停止条件？ ✅                                   │
│                                                              │
│  3. 创建循环计划:                                             │
│     → 文件: .claude/plans/lint-fix-loop.md                   │
│                                                              │
│  4. 输出启动命令:                                             │
│     claude -p "fix next lint warning" --allowedTools Edit     │
│                                                              │
│  Loop 模式:                                                  │
│  ├── sequential: 逐个处理，每次一个任务                       │
│  ├── continuous-pr: 每完成一批自动创建 PR                     │
│  ├── rfc-dag: RFC 驱动的依赖图执行（最复杂）                  │
│  └── infinite: 无限循环直到显式停止                           │
│                                                              │
│  安全模式:                                                    │
│  ├── safe: 严格质量门控，每步验证                             │
│  └── fast: 减少门控，加速执行                                 │
└─────────────────────────────────────────────────────────────┘
```

**监控循环状态**：
```
你输入: /loop-status

=== Loop Status ===
Pattern:     sequential
Phase:       Iteration 12/50
Last OK:     Iteration 11 (2 min ago)
Failing:     None
Cost Drift:  $0.45 (within budget)
Recommend:   Continue ✅
```

#### 场景 16：模型路由优化（/model-route）

不是所有任务都需要最强的模型。明智地选择模型可以节省成本：

```
你输入: /model-route 修复 3 个 typo 错误
```

**输出**：
```
Recommended Model: haiku
Confidence: HIGH
Rationale: Deterministic, low-risk mechanical changes. No architectural reasoning needed.
Fallback: sonnet (if haiku fails to understand context)
```

```
你输入: /model-route 设计游戏资料库的数据库 schema 和 API 架构
```

**输出**：
```
Recommended Model: opus
Confidence: HIGH
Rationale: Architectural decision requiring deep analysis of data relationships,
           query patterns, and future scalability. Benefits from maximum reasoning.
Fallback: sonnet (if budget constrained)
```

**模型选择速查**：

| 模型 | 适用场景 | 成本比 |
|------|---------|--------|
| **Haiku** | 改 typo、改配置、简单重命名、格式化 | 1x |
| **Sonnet** | 功能开发、重构、常规编码（默认） | 3x |
| **Opus** | 架构设计、复杂调试、深度分析 | 10x |

---

### Phase 7: 文档生成与交接

#### 场景 17：生成代码地图（/update-codemaps）

```
你输入: /update-codemaps
```

**框架内部动作**：
```
┌─────────────────────────────────────────────────────────────┐
│  /update-codemaps 命令                                       │
│                                                              │
│  1. 触发 doc-updater Agent (subagent, 完整权限)              │
│                                                              │
│  2. 扫描项目结构:                                            │
│     ├── 识别: Next.js 15 App Router 项目                     │
│     ├── 源码目录: src/                                       │
│     └── 入口点: src/app/layout.tsx                           │
│                                                              │
│  3. 生成/更新 docs/CODEMAPS/:                                │
│     ├── architecture.md  — 系统架构图、服务边界、数据流       │
│     ├── frontend.md      — 页面树、组件层级、状态管理         │
│     ├── backend.md       — API 路由、中间件、Service 映射     │
│     ├── data.md          — 数据库表、关系、迁移               │
│     └── dependencies.md  — 外部服务、第三方集成               │
│                                                              │
│  4. 变更 > 30% 时请求用户确认再覆盖                          │
│  5. 每个文件 < 500 行（Token 效率）                          │
└─────────────────────────────────────────────────────────────┘
```

#### 场景 18：同步文档（/update-docs）

```
你输入: /update-docs
```

**框架内部动作**：
- 触发 `doc-updater` Agent
- 从源代码生成文档（非手写）
- 更新：
  - 脚本参考表（从 package.json 的 scripts 生成）
  - 环境变量文档（从 .env.example 生成）
  - 贡献指南 `docs/CONTRIBUTING.md`
  - 运行手册 `docs/RUNBOOK.md`
- 保留手动编写的段落（用 `<!-- AUTO-GENERATED -->` 标记自动段落）
- 检查超过 90 天未更新的文档并标记

#### 场景 19：结构化交接（/handoff）

当你需要将工作交给团队成员或新的 AI 会话：

```
你输入: /handoff
```

**框架内部动作**：
```
┌──────────────────────────────────────────────────────────┐
│  /handoff 命令                                            │
│                                                           │
│  1. 不调用 Agent，直接收集状态                             │
│  2. 生成结构化交接文档:                                    │
│     ├── 当前状态 (completed / in-progress / pending)       │
│     ├── SPEC 状态 (位置、阶段、关键设计决策)               │
│     ├── CDD 状态 (编译命令、最后结果、未解决错误)          │
│     ├── 文件变更列表 (from git diff)                       │
│     ├── 关键决策记录 (含理由)                              │
│     ├── 已知风险/阻塞                                     │
│     └── 具体下一步操作                                    │
│  3. 输出到 docs/handoff-2026-03-13.md                     │
│  4. 同时保存到 memory MCP (跨会话记忆)                     │
└──────────────────────────────────────────────────────────┘
```

**简要版**：
```
你输入: /handoff brief   # 只输出状态 + 下一步
```

---

## 第四章：32 个命令完整速查表

### 核心开发命令

| 命令 | 调用的 Agent | 调用的 MCP | 用途 |
|------|------------|-----------|------|
| `/plan <功能>` | planner (只读) | context7, exa-web-search | Spec-First 完整规划 |
| `/spec <功能>` | planner (只读) | context7, exa-web-search | 只输出 SPEC，不含步骤 |
| `/cdd` | 无 | 无 | 快速编译验证 |
| `/build-fix` | build-error-resolver (读写) | sequential-thinking, mcp-language-server | CDD 修复编译错误 |
| `/verify [quick\|full\|pre-commit\|pre-pr]` | 自动触发 build-error-resolver | sequential-thinking | CDD 全面验证 |
| `/code-review` | code-reviewer (只读) | 无 | 安全+质量审查 |
| `/refactor-clean` | refactor-cleaner (读写) | 无 | 死代码清理 |
| `/quality-gate [path] [--fix] [--strict]` | 无 | 无 | 手动质量检查 |

### 多 Agent 与自动化

| 命令 | 调用的 Agent | 用途 |
|------|------------|------|
| `/orchestrate <type> <desc>` | 多个 Agent 顺序协作 | 多 Agent 工作流编排 |
| `/loop-start [pattern] [--mode]` | loop-operator | 启动自主循环 |
| `/loop-status [--watch]` | 无 | 监控循环状态 |
| `/model-route [task] [--budget]` | 无 | 推荐最优模型 |

### 会话管理

| 命令 | 用途 |
|------|------|
| `/save-session` | 保存会话状态到文件 |
| `/resume-session [date\|path]` | 恢复历史会话 |
| `/sessions list\|load\|alias\|info` | 管理会话历史 |
| `/checkpoint create\|verify\|list` | 创建/验证工作流检查点 |
| `/handoff [brief]` | 生成交接文档 |

### 自主学习

| 命令 | 用途 |
|------|------|
| `/learn` | 从会话中提取可复用模式 |
| `/learn-eval` | 带质量门控的模式提取 |
| `/instinct-status` | 查看已学习的本能 |
| `/instinct-export [--domain] [--min-confidence]` | 导出本能 |
| `/instinct-import <file\|url> [--dry-run]` | 导入本能 |
| `/evolve [--generate]` | 将本能进化为 skill/command/agent |
| `/promote [--dry-run] [id]` | 提升项目本能到全局 |
| `/projects` | 列出所有项目及本能统计 |
| `/skill-create [--commits N] [--instincts]` | 从 Git 历史生成 Skill |

### 文档与配置

| 命令 | 调用的 Agent | 用途 |
|------|------------|------|
| `/update-codemaps` | doc-updater (读写) | 生成代码架构地图 |
| `/update-docs` | doc-updater (读写) | 从代码同步文档 |
| `/harness-audit [scope]` | harness-optimizer | 审计框架配置 |
| `/setup-pm [--detect\|--global\|--project]` | 无 | 配置包管理器 |
| `/eval define\|check\|report\|list` | 无 | 管理评估标准 |
| `/claw` | 无 | 启动交互式 REPL |

---

## 第五章：8 个 Agent 详解

### Agent 权限矩阵

| Agent | 读文件 | 写文件 | 编辑 | Bash | 调用的 MCP | 典型触发 |
|-------|--------|--------|------|------|-----------|---------|
| **planner** | ✅ | ❌ | ❌ | ❌ | context7, exa-web-search | `/plan`, `/spec` |
| **architect** | ✅ | ❌ | ❌ | ❌ | 无 | 架构决策时主动触发 |
| **build-error-resolver** | ✅ | ✅ | ✅ | ✅ | sequential-thinking, mcp-language-server | `/build-fix`, `/verify` |
| **code-reviewer** | ✅ | ❌ | ❌ | ✅(git) | 无 | `/code-review` |
| **refactor-cleaner** | ✅ | ✅ | ✅ | ✅ | 无 | `/refactor-clean` |
| **doc-updater** | ✅ | ✅ | ✅ | ✅ | 无 | `/update-codemaps`, `/update-docs` |
| **loop-operator** | ✅ | ✅ | ✅ | ✅ | 无 | `/loop-start` |
| **harness-optimizer** | ✅ | ✅ | ✅ | ✅ | 无 | `/harness-audit` |

### Agent 设计原则

**只读 Agent（planner, architect, code-reviewer）**：
- 只能分析和建议，不能修改代码
- 防止"分析者"越权直接改代码
- 所有修改必须由人类确认后由其他 Agent 或主会话执行

**读写 Agent（build-error-resolver, refactor-cleaner, doc-updater, loop-operator, harness-optimizer）**：
- 可以直接修改文件
- 但有严格的安全护栏（如 build-error-resolver 的 3 轮限制）
- 每次修改后必须验证（CDD 编译检查）

---

## 第六章：16 个 Skill 详解

### Skill 分类

| 类别 | Skill 名称 | 触发条件 |
|------|-----------|---------|
| **CDD 核心** | verification-loop | 代码变更后需要验证 |
| **规划** | blueprint | 复杂多 PR 任务需要蓝图 |
| **搜索** | search-first | 实现新功能前搜索现有方案 |
| **学习** | continuous-learning-v2 | 会话中自动提取模式（Hook 驱动） |
| | continuous-learning | V1 版本，Stop Hook 驱动 |
| **上下文** | strategic-compact | 长会话的上下文压缩策略 |
| **评估** | eval-harness | 评估驱动开发 (EDD) |
| **编码** | coding-standards | 编码风格和最佳实践 |
| **工程模式** | agentic-engineering | Agent 工程方法论 |
| | ai-first-engineering | AI 优先的团队工程模型 |
| | autonomous-loops | 自主循环架构模式 |
| **后端** | api-design | REST API 设计模式 |
| | backend-patterns | 后端架构模式 |
| **缓存** | content-hash-cache-pattern | 内容哈希缓存模式 |
| **配置** | configure-ecc | ECC 框架安装向导 |
| **模板** | project-guidelines-example | 项目特定 Skill 模板 |

### Skill vs Agent vs Command 的区别

| 维度 | Skill | Agent | Command |
|------|-------|-------|---------|
| **触发方式** | 自动匹配（by 描述） | 被命令/主会话调用 | 用户手动输入 `/xxx` |
| **权限** | 继承加载者权限 | 独立权限（只读/读写） | 取决于调用的 Agent |
| **作用** | 提供知识和工作流指导 | 执行具体任务 | 用户交互入口 |
| **生命周期** | 按需加载到上下文 | 作为 subagent 独立运行 | 单次调用 |
| **类比** | 教科书/手册 | 专业工人 | 按钮/开关 |

---

## 第七章：5 个 MCP 工具详解

### MCP 矩阵

```
┌──────────────────────────────────────────────────────────────────────────┐
│                        5 个核心 MCP 工具                                  │
│                                                                          │
│  ┌─────────────────┐  ┌─────────────────┐  ┌──────────────────────────┐ │
│  │  exa-web-search  │  │    context7     │  │  mcp-language-server     │ │
│  │  ─────────────── │  │  ───────────── │  │  ────────────────────── │ │
│  │  全网搜索:        │  │  官方文档查询:  │  │  本地 LSP 诊断:          │ │
│  │  • 代码示例       │  │  • API 参考     │  │  • 类型错误检测          │ │
│  │  • 报错解决方案   │  │  • 配置指南     │  │  • 编译状态              │ │
│  │  • 最佳实践       │  │  • 版本差异     │  │  • 代码补全建议          │ │
│  │                   │  │  • 防止 AI 幻觉 │  │  • CDD 核心引擎         │ │
│  │  用于: Spec 研究  │  │  用于: Spec 研究│  │  用于: /verify, /cdd     │ │
│  └─────────────────┘  └─────────────────┘  └──────────────────────────┘ │
│                                                                          │
│  ┌─────────────────────────────┐  ┌─────────────────────────────────┐   │
│  │         memory              │  │     sequential-thinking         │   │
│  │  ─────────────────────────  │  │  ─────────────────────────────  │   │
│  │  跨会话记忆:                │  │  链式推理:                       │   │
│  │  • 项目架构决策             │  │  • 编译错误根因分析              │   │
│  │  • 编译配置                 │  │  • 多步骤逻辑推导               │   │
│  │  • 学到的模式               │  │  • 影响范围评估                 │   │
│  │  • 团队约定                 │  │  • 修复方案设计                 │   │
│  │                             │  │                                 │   │
│  │  用于: /handoff, /learn     │  │  用于: /build-fix (强制调用)    │   │
│  │  自动被 Hook 持久化         │  │  编译失败时由 Agent 自动调用    │   │
│  └─────────────────────────────┘  └─────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────────────┘
```

### MCP 使用场景对照

| 场景 | 使用的 MCP | 调用者 |
|------|-----------|--------|
| 研究新技术的 API | context7 + exa-web-search | planner Agent |
| 查 NextAuth v5 的最新用法 | context7 | planner Agent, 你手动 |
| 搜索某个编译错误的解决方案 | exa-web-search | build-error-resolver, 你手动 |
| 检查代码有没有类型错误 | mcp-language-server | /verify, /cdd |
| 分析编译错误的根因 | sequential-thinking | build-error-resolver (强制) |
| 记录"选了 Prisma 而非 Drizzle"的决策 | memory | /handoff, /learn, Hook |
| 下次新会话记起项目上下文 | memory | /resume-session, Hook |

---

## 第八章：Hook 系统与自动化

### 8.1 Hook 生命周期

```
会话开始
  │
  ├── SessionStart Hook → session-start.js (加载上次上下文)
  │
  ├── 用户输入
  │     │
  │     ├── PreToolUse Hooks (工具执行前)
  │     │   ├── suggest-compact.js   → 工具调用 >50 次时提醒压缩
  │     │   ├── doc-file-warning.js  → 写非标准文档文件时警告
  │     │   ├── observe.sh           → 记录操作到 observations.jsonl (学习系统)
  │     │   └── auto-tmux-dev.js     → 长时间命令建议用 tmux
  │     │
  │     ├── [工具执行]
  │     │
  │     └── PostToolUse Hooks (工具执行后)
  │         ├── post-edit-format.js     → 编辑后自动格式化 (Biome/Prettier)
  │         ├── post-edit-typecheck.js  → 编辑 .ts/.tsx 后自动类型检查
  │         ├── post-edit-console-warn.js → 检测 console.log 并警告
  │         ├── quality-gate.js         → 编辑后运行质量检查
  │         └── observe.sh              → 记录操作 (学习系统)
  │
  ├── 会话结束
  │     │
  │     └── Stop Hooks
  │         ├── check-console-log.js    → 最终 console.log 检查
  │         ├── session-end.js          → 持久化会话状态
  │         ├── evaluate-session.js     → 评估会话提取模式
  │         └── cost-tracker.js         → 记录 Token 和成本
  │
  └── 上下文压缩前
        │
        └── PreCompact Hook
            └── pre-compact.js          → 压缩前保存关键状态
```

### 8.2 Hook Profiles

通过环境变量控制 Hook 激活级别：

```bash
export ECC_HOOK_PROFILE=minimal    # 最少 Hook（快速开发）
export ECC_HOOK_PROFILE=standard   # 标准 Hook（默认）
export ECC_HOOK_PROFILE=strict     # 全部 Hook（严格质量）

# 禁用特定 Hook
export ECC_DISABLED_HOOKS=suggest-compact,doc-file-warning
```

| Profile | 启用的 Hook | 适用场景 |
|---------|------------|---------|
| minimal | 只有编译检查 + 格式化 | 快速原型、小修改 |
| standard | 编译 + 格式化 + 质量 + 学习 | 日常开发 |
| strict | 全部 Hook 包括安全审查 | PR 前、发布前 |

---

## 第九章：高级技巧与精髓

### 9.1 Token 优化策略

**问题**：长会话中上下文窗口会被填满，导致 AI 响应质量下降。

**策略矩阵**：

| 策略 | 命令/操作 | 何时使用 | Token 节省 |
|------|----------|---------|-----------|
| 上下文压缩 | `/compact` | 工具调用 >50 次 | 高 |
| 会话保存+新开 | `/save-session` → 新会话 → `/resume-session` | 切换任务主题 | 最高 |
| 模型降级 | `/model-route` → haiku | 简单机械任务 | 3-10x 成本 |
| 分散到子 Agent | `/orchestrate` / 并行 Task | 独立任务 | 分散到多窗口 |
| 代码地图代替全文 | `/update-codemaps` | 大项目导航 | 避免读大文件 |
| 精准搜索代替全局读 | 使用 Grep/Glob 而非读整个文件 | 查找特定代码 | 中 |

**黄金规则**：
- 上下文窗口的最后 20% 避免做大型重构
- 单文件编辑、简单 bug 修复对上下文压力最小
- 跨多文件的功能开发在上下文较新时做

### 9.2 Spec-First 的深层价值

**不只是"先写文档"，而是让 AI 从源头杜绝幻觉**：

```
错误方式（无 Spec）:
  你: "帮我用 NextAuth 实现登录"
  AI: "好的，我来写..." → 使用 v4 语法 → 编译错误 → 反复修复 → 浪费 Token

正确方式（Spec-First）:
  你: "/spec NextAuth v5 登录系统"
  AI: [调用 context7 查阅 v5 官方文档]
      [调用 exa-web-search 搜索 v5 迁移指南]
      → 输出基于真实文档的 SPEC
  你: "确认"
  AI: [按照 SPEC 编码] → 一次编译通过 → 节省 80% Token
```

### 9.3 CDD 循环的精髓

**build-error-resolver 的"最小修复"原则**：

```
❌ 错误做法: 遇到类型错误 → 重写整个函数
✅ 正确做法: 遇到类型错误 → sequential-thinking 分析 → 只改出错的那一行

为什么？
1. 最小修改 = 最少副作用
2. 每次修改 < 5% 的文件 → 可控
3. 如果修复引入了更多错误 → 自动回滚
4. 3 轮后仍失败 → 停下来报告，不无限循环
```

### 9.4 Memory MCP 的跨会话威力

**Memory 不只是"记事本"，它是 AI 的长期记忆**：

```
会话 1:
  /plan → 决定用 Prisma 而非 Drizzle
  /handoff → 自动保存决策到 memory MCP:
    Entity: "realm-of-legends-orm-decision"
    Observation: "选择 Prisma 因为团队更熟悉，且与 NextAuth 集成更好"

会话 2 (第二天):
  AI 自动从 memory 加载项目上下文
  → 知道用的是 Prisma，不会建议用 Drizzle
  → 知道上次做到哪里了
  → 知道哪些方案试过但失败了
```

### 9.5 并行任务执行的最佳实践

**什么时候并行？什么时候串行？**

```
✅ 可以并行（无依赖）:
  - 安全审查 + 性能分析 + 类型检查
  - 前端组件 + 后端 API（如果接口已定义）
  - 多个独立页面的开发

❌ 必须串行（有依赖）:
  - 数据库 schema → API → 前端（上游决定下游）
  - SPEC → 编码（SPEC 未确认不能编码）
  - 编译修复的多轮循环（每轮依赖上一轮的结果）
```

**实战：并行开发三个独立页面**：

OpenCode 会使用 Task 工具同时启动 3 个子 Agent：
```
Agent 1: 实现首页 (src/app/page.tsx)
Agent 2: 实现下载页 (src/app/download/page.tsx)
Agent 3: 实现关于页 (src/app/about/page.tsx)

三个 Agent 各自独立编码 → 各自 /cdd 验证 → 合并结果
```

### 9.6 continuous-learning-v2 深度解析

**V2 vs V1 的关键区别**：

| 维度 | V1 (continuous-learning) | V2 (continuous-learning-v2) |
|------|-------------------------|---------------------------|
| 触发时机 | 会话结束时 (Stop Hook) | 实时 (每次 PreToolUse/PostToolUse) |
| 学习粒度 | 整个会话 → 一个 Skill | 每个操作 → 一个 Instinct |
| 项目隔离 | 无 | 按 git remote hash 隔离 |
| 置信度 | 无 | 0.3 (tentative) → 0.9 (near-certain) |
| 进化能力 | 无 | Instinct → Skill / Command / Agent |

**V2 的自动学习流程**：
```
1. observe.sh Hook 在每次工具调用时记录到 observations.jsonl
2. 后台 observer agent (Haiku 模型，低成本) 分析观察记录
3. 检测模式: 用户纠正、错误解决、重复工作流
4. 创建原子 Instinct (一个触发条件 → 一个动作)
5. 随着重复观察，置信度从 0.3 逐渐上升到 0.9
6. /evolve 聚类相关 instincts → 进化为完整 Skill/Command/Agent
7. /promote 将跨项目出现的 instinct 提升为全局知识
```

### 9.7 Blueprint — 大型项目的终极规划工具

当项目复杂到需要多个 PR、多个会话来完成时：

```
你输入: 我需要一个完整的蓝图来重构整个认证系统，从 session-based 迁移到 JWT
```

**Blueprint Skill 自动触发**（因为检测到"多 PR 级别的任务"）：

```
Blueprint 5 阶段流水线:

Phase 1: Research (预检)
  ├── 检查 git 状态、远程仓库、默认分支
  ├── 读取项目结构和已有计划
  └── 读取 memory 中的历史决策

Phase 2: Design (拆分)
  ├── 将目标拆成 3-12 个 one-PR-sized 步骤
  ├── 标注依赖边（哪些步骤可以并行）
  ├── 为每步分配模型级别 (haiku/sonnet/opus)
  └── 为每步设计回滚策略

Phase 3: Draft (撰写)
  └── 写入 plans/realm-of-legends-jwt-migration.md
      每个步骤包含:
      - 自包含上下文简报（新 Agent 可以冷启动执行）
      - 任务清单
      - 验证命令
      - 退出标准

Phase 4: Review (对抗审查)
  └── 用最强模型 (Opus) 做对抗审查:
      - 检查反模式目录
      - 检查依赖图完整性
      - 检查每步的退出标准是否可验证
      - 修复所有 CRITICAL 发现

Phase 5: Register (注册)
  ├── 保存计划文件
  ├── 更新 memory 索引
  └── 输出步骤数和并行度摘要
```

---

## 第十章：Intent Gate 意图分类决策树

每次你给 AI 一个指令，框架首先对你的意图进行分类，然后选择正确的工作流：

```
你的指令
   │
   ├── "实现 XXX 功能" ──────→ implement
   │   │
   │   └── 完整流程:
   │       /spec or /plan → 确认 → 编码 → /cdd → /verify → /code-review
   │
   ├── "编译报错了" / "XXX 不工作" ──→ fix
   │   │
   │   └── 直接进入:
   │       /build-fix → /cdd 验证
   │
   ├── "重构 XXX" / "清理代码" ──→ refactor
   │   │
   │   └── 走 refactor-cleaner:
   │       /refactor-clean → /cdd 验证
   │
   ├── "调研一下 XXX" / "对比 A 和 B" ──→ research
   │   │
   │   └── 只用 MCP 研究:
   │       context7 + exa-web-search → 报告（不写代码）
   │
   └── "改个 typo" / "加个字段" ──→ quick-edit
       │
       └── 直接修改:
           Edit → /cdd 验证
```

**关键规则**：只有 `implement` 需要完整的 Spec-First 流程。不要对一个 typo 修复生成 SPEC.md。

---

## 附录 A：命令-Agent-Skill-MCP 调用关系图

```
用户命令          →  调用的 Agent         →  Agent 使用的 MCP/Skill
─────────────────────────────────────────────────────────────────────
/plan             →  planner (只读)       →  context7, exa-web-search
/spec             →  planner (只读)       →  context7, exa-web-search
/build-fix        →  build-error-resolver →  sequential-thinking, mcp-language-server
/cdd              →  (无 Agent)           →  (直接运行编译命令)
/verify           →  (auto) build-error.. →  sequential-thinking, mcp-language-server
/code-review      →  code-reviewer (只读) →  (无外部 MCP)
/refactor-clean   →  refactor-cleaner     →  (无外部 MCP)
/orchestrate      →  多个 Agent 顺序      →  取决于调用的 Agent
/loop-start       →  loop-operator        →  (无外部 MCP)
/update-codemaps  →  doc-updater          →  (无外部 MCP)
/update-docs      →  doc-updater          →  (无外部 MCP)
/harness-audit    →  harness-optimizer    →  (无外部 MCP)
/learn            →  (无 Agent)           →  memory (保存模式)
/learn-eval       →  (无 Agent)           →  memory (保存模式)
/evolve           →  (无 Agent)           →  (运行 instinct-cli.py)
/handoff          →  (无 Agent)           →  memory (保存交接上下文)
/save-session     →  (无 Agent)           →  (写入文件)
/resume-session   →  (无 Agent)           →  (读取文件)

Skill 自动触发:
──────────────
verification-loop        → 代码变更后自动加载
blueprint                → 检测到多 PR 任务时自动加载
strategic-compact        → suggest-compact.js Hook 触发
search-first             → 新功能实现前自动加载
continuous-learning-v2   → observe.sh Hook 持续运行
coding-standards         → 编码时自动加载
```

---

## 附录 B：常见问题 FAQ

### Q1: /plan 和 /spec 有什么区别？
- `/spec` = 只输出规范（API 设计、数据模型、依赖）
- `/plan` = 规范 + 实施步骤（分阶段、依赖图、风险评估）
- 小功能用 `/spec`，大功能用 `/plan`

### Q2: /cdd 和 /verify 有什么区别？
- `/cdd` = 只跑编译（最快，1 秒出结果）
- `/verify` = 编译 + 类型 + Lint + Debug 审计 + AI Slop 审计 + Git 状态
- 写代码时频繁用 `/cdd`，提交前用 `/verify`

### Q3: /build-fix 修了 3 轮还是失败怎么办？
Agent 会自动停止并报告：
- 错误文本
- 推理过程
- 尝试过的修复
- 建议的下一步（可能需要用 `/plan` 重新设计）

### Q4: 什么时候用 /learn vs /learn-eval？
- `/learn` = 快速提取，直接保存
- `/learn-eval` = 带质量门控，检查是否与已有 skill 重复，决定保存位置
- 日常用 `/learn`，重要模式用 `/learn-eval`

### Q5: 如何在团队中共享学习到的模式？
```
/instinct-export --min-confidence 0.8 --output team-patterns.yaml
# 发给队友
/instinct-import team-patterns.yaml --scope project
```

### Q6: 上下文快满了，AI 回复质量下降怎么办？
```
/save-session          # 保存当前状态
/compact               # 压缩上下文（OpenCode 内置）
/resume-session        # 加载关键上下文继续工作
```

### Q7: 如何知道用哪个模型最划算？
```
/model-route 你要做的任务描述
```
简单任务 → Haiku（省钱），常规开发 → Sonnet（默认），架构设计 → Opus（最强）

### Q8: Hook 太多太慢了怎么办？
```bash
export ECC_HOOK_PROFILE=minimal     # 只保留核心 Hook
# 或者禁用特定 Hook
export ECC_DISABLED_HOOKS=suggest-compact,quality-gate
```

### Q9: 如何用框架做 Unity/UE5 项目？
编译命令矩阵支持多语言：
| 项目 | 编译命令 |
|------|---------|
| Unity (C#) | `dotnet build` |
| UE5 (C++) | `UnrealBuildTool` |
| Cocos (TS) | `npx tsc --noEmit` |
| Rust | `cargo check` |
| Go | `go build ./...` |
| Python | `pyright .` 或 `mypy .` |

框架的 `/cdd` 和 `/build-fix` 会自动检测项目类型并使用对应的编译命令。

### Q10: 这个框架和普通的 AI 编程有什么本质区别？

| 维度 | 普通 AI 编程 | 本框架 |
|------|------------|--------|
| 验证 | 没有系统化验证 | CDD: 编译 = 唯一真理 |
| 研究 | AI 凭记忆编码（可能过时） | Spec-First: 先查文档再编码 |
| 错误处理 | AI 盲猜修复 | 强制 sequential-thinking 推理 |
| 学习 | 每次重新开始 | 跨会话记忆 + 本能学习系统 |
| 协作 | 单 Agent | 8 Agent 编排 + 自主循环 |
| 质量 | 人工审查 | 自动 Hook + 质量门控 |
| 成本 | 一律用最强模型 | 模型路由按任务分级 |

---

## 结语

本指南覆盖了 OpenCode Simple 框架的：
- **32 个命令**的完整用法和内部执行流程
- **8 个 Agent** 的权限矩阵和协作模式
- **16 个 Skill** 的触发条件和工作流
- **5 个 MCP** 的使用场景和调用关系
- **Hook 系统**的生命周期和配置
- **高级技巧**：Token 优化、上下文压缩、并行执行、自主学习、蓝图规划

核心精髓三句话：
1. **编译通过 = 任务完成**（CDD 铁律）
2. **先查文档，再写代码**（Spec-First 防幻觉）
3. **让 AI 从错误中学习**（Continuous Learning V2）

掌握了这三点，你就掌握了这个框架 90% 的价值。
