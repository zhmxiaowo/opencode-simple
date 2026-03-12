# Agent Orchestration

## Available Agents

| Agent | Purpose | When to Use |
|-------|---------|-------------|
| planner | Implementation planning | Complex features, refactoring |
| architect | System design | Architectural decisions |
| build-error-resolver | CDD 编译驱动解决引擎 | 编译失败或 LSP 报错时 |
| code-reviewer | Code review (无测试要求) | After writing code |
| refactor-cleaner | Dead code cleanup | Code maintenance |
| doc-updater | Documentation | Updating docs |
| loop-operator | Autonomous loop operator | Long-running agent loops |
| harness-optimizer | Harness configuration tuning | Eval & harness optimization |

## Immediate Agent Usage

No user prompt needed:
1. Complex feature requests → Use **planner** agent
2. Code just written/modified → Use **code-reviewer** agent
3. 编译失败 → Use **build-error-resolver** agent (强制调用 sequential-thinking)
4. Architectural decision → Use **architect** agent

## CDD 验证原则

- **绝对不使用** tdd-guide, e2e-runner, security-reviewer 等已移除的 agent
- 验证标准：编译通过 (Exit Code 0) + LSP Error 为 0
- 遇到编译错误，build-error-resolver 会自动调用 sequential-thinking 进行推理

## Parallel Task Execution

ALWAYS use parallel Task execution for independent operations:

```markdown
# GOOD: Parallel execution
Launch 3 agents in parallel:
1. Agent 1: Security analysis of auth module
2. Agent 2: Performance review of cache system
3. Agent 3: Type checking of utilities

# BAD: Sequential when unnecessary
First agent 1, then agent 2, then agent 3
```

## Multi-Perspective Analysis

For complex problems, use split role sub-agents:
- Factual reviewer
- Senior engineer
- Security expert
- Consistency reviewer
- Redundancy checker
