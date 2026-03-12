# Development Workflow

> This file extends [common/git-workflow.md](./git-workflow.md) with the full feature development process that happens before git operations.

The Feature Implementation Workflow describes the development pipeline: research, spec, implementation, CDD verification, code review, and then committing to git.

## Intent Gate（意图门控）

在执行任何操作之前，**必须**先分类用户意图，选择正确的工作流路径：

| 意图 | 判定条件 | 工作流路径 |
|------|---------|-----------|
| **implement** | 新功能、新模块 | Research → Plan (含 SPEC) → Code → CDD → Review |
| **fix** | 编译错误、Bug 修复、行为异常 | 直接进入 CDD 修复循环 (build-error-resolver) |
| **refactor** | 重构、清理、性能优化 | refactor-cleaner agent → CDD 验证 |
| **research** | 技术调研、方案对比、可行性分析 | 只调用 MCP 研究，不写代码 |
| **quick-edit** | 改个配置、改个文案、加个字段 | 直接修改 → CDD 验证 |

**关键规则**：只有 `implement` 需要走完整的 Spec-First 流程。不要对一个 typo 修复强制生成 SPEC.md。

## Feature Implementation Workflow

0. **Research & Reuse** _(mandatory before any new implementation)_
   - **GitHub code search first:** Run `gh search repos` and `gh search code` to find existing implementations, templates, and patterns before writing anything new.
   - **Exa MCP for research:** Use `exa-web-search` MCP during the planning phase for broader research, data ingestion, and discovering prior art.
   - **Context7 for docs:** Use `context7` MCP to look up the latest official documentation for the target technology/framework.
   - **Check package registries:** Search npm, PyPI, crates.io, and other registries before writing utility code. Prefer battle-tested libraries over hand-rolled solutions.
   - **Search for adaptable implementations:** Look for open-source projects that solve 80%+ of the problem and can be forked, ported, or wrapped.
   - Prefer adopting or porting a proven approach over writing net-new code when it meets the requirement.

1. **Plan + Spec-First (无 Spec 不编码)**
   - Use `/plan` command → planner agent 会自动完成 MCP 研究 + SPEC 输出 + 步骤分解
   - 小功能可用 `/spec` 只输出规范
   - **必须等待用户确认后方可编码**

2. **CDD 编译驱动验证**
   - 使用 **build-error-resolver** agent
   - 运行编译命令 → Exit Code 0
   - 检查 LSP 诊断 → 0 Error
   - 遇到错误 → 调用 sequential-thinking 推理
   - 绿灯即完成

3. **Code Review**
   - Use **code-reviewer** agent immediately after writing code
   - Address CRITICAL and HIGH issues
   - Fix MEDIUM issues when possible
   - **注意：代码审查中不得以"缺少测试"为由阻塞**

4. **Commit & Push**
   - Detailed commit messages
   - Follow conventional commits format
   - See [git-workflow.md](./git-workflow.md) for commit message format and PR process
