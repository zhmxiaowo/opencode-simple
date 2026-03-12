# CDD（编译驱动开发）核心准则

> 本框架的最高信仰：**编译通过 = 任务完成**。绝对禁止 TDD。

## 三大铁律

### 铁律 1：绝对禁止 TDD

- **禁止编写**任何测试文件（`.test.ts`, `_test.go`, `.spec.js`, `test_*.py` 等）
- **禁止创建** `__tests__/`, `tests/`, `spec/` 等测试目录
- **禁止引用**任何测试框架（Jest, Vitest, pytest, Google Test, NUnit 等）
- **禁止在代码审查中**以 "缺少测试" 为由阻塞
- 如果用户明确要求写测试，礼貌拒绝并解释 CDD 原则

### 铁律 2：无 Spec 不编码（Spec-First）

> **适用范围**：Intent Gate 分类为 `implement` 的新功能请求。`fix` / `quick-edit` / `refactor` 不强制。

在写任何实现代码之前，**必须**完成以下流程：

1. **查阅文档**：使用 `context7` MCP 获取目标技术的最新官方文档
2. **搜索参考**：使用 `exa-web-search` MCP 搜索最佳实践和已知陷阱
3. **输出 SPEC**：通过 `/plan` 命令生成包含 SPEC 的实施计划（或 `/spec` 只输出规范），包含：
   - 功能需求摘要
   - API/接口设计（签名、参数、返回值）
   - 数据模型/结构体定义
   - 依赖项及版本约束
   - 已知限制和兼容性说明
   - 参考文档链接
4. **等待确认**：SPEC 必须经用户确认后方可开始编码

### 铁律 3：编译驱动验证（CDD）

代码的**唯一**验证标准：

```
编译器/类型检查器 Exit Code == 0  AND  LSP Error 数量 == 0
```

#### CDD 验证流程

1. **编写代码** → 运行编译命令
2. **编译通过** → ✅ 任务完成
3. **编译失败** → 进入 CDD 修复循环：
   a. **禁止盲目试错**：强制调用 `sequential-thinking` MCP 进行逻辑推导
   b. 分析编译器错误信息 + LSP 诊断
   c. 定位根因，设计最小修复方案
   d. 应用修复，重新编译
   e. 若 3 次循环后仍失败 → 停下来，向用户报告分析结果

#### 通用编译命令矩阵

| 项目类型 | 编译/检查命令 |
|----------|--------------|
| TypeScript/Node | `npx tsc --noEmit` |
| Unity (C#) | `dotnet build` 或 Unity Editor 编译 |
| Unreal Engine (C++) | `UnrealBuildTool` |
| Cocos Creator (TS) | `npx tsc --noEmit` |
| Rust | `cargo check` |
| Go | `go build ./...` |
| Python (类型检查) | `pyright .` 或 `mypy .` |
| C/C++ | `cmake --build .` 或 `make` |
| Kotlin/Gradle | `./gradlew compileKotlin` |
| Swift | `swift build` |

## 工作流总览

```
用户需求
    │
    ▼
┌─────────────────────────────┐
│  Intent Gate: 意图分类        │
│  implement / fix / refactor  │
│  / research / quick-edit     │
└──────────┬──────────────────┘
           │ implement
           ▼
┌─────────────────────────────┐
│  Phase 0: Research           │
│  context7 + exa 查阅文档     │
└──────────┬──────────────────┘
           │
           ▼
┌─────────────────────────────┐
│  Phase 1: /plan (含 SPEC)    │
│  输出 SPEC + 步骤 → 等待确认  │
└──────────┬──────────────────┘
           │ 用户确认
           ▼
┌─────────────────────────────┐
│  Phase 2: Implementation    │
│  按 SPEC 编写代码            │
└──────────┬──────────────────┘
           │
           ▼
┌─────────────────────────────┐
│  Phase 3: CDD Verify        │
│  编译 → Exit 0 + 0 LSP Err  │
│  失败 → sequential-thinking  │
│  → 最小修复 → 重新编译       │
└──────────┬──────────────────┘
           │ 绿灯
           ▼
┌─────────────────────────────┐
│  Phase 4: Code Review       │
│  code-reviewer agent        │
│  (安全+质量+AI slop，无测试)  │
└──────────┬──────────────────┘
           │
           ▼
        ✅ 完成
```

> **fix / quick-edit** 跳过 Phase 0-1，直接进入 Phase 2-3。
> **refactor** 走 refactor-cleaner agent → Phase 3。
> **research** 只做 Phase 0，不写代码。

## Memory 与学习

- 每次 CDD 修复循环的经验通过 `continuous-learning-v2` hooks 自动捕获
- 编译错误模式会被提取为 instincts，加速未来同类问题的解决
- 使用 `memory` MCP 记录跨会话的项目上下文（架构决策、编译配置等）
