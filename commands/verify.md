# CDD Verification Command

Run CDD (Compile-Driven Development) verification on current codebase state.

> **核心原则**：编译通过 + LSP 无 Error = 验证通过。不运行任何测试。

## Instructions

Execute verification in this exact order:

1. **Build / Compile Check**
   - 自动检测项目类型并运行对应编译命令
   - If it fails, invoke `sequential-thinking` to analyze, then report errors and STOP

2. **Type / LSP Check**
   - Run type checker (tsc / pyright / dotnet build etc.)
   - Report all errors with file:line
   - Ensure 0 Error-level LSP diagnostics

3. **Lint Check**
   - Run linter (if configured)
   - Report warnings and errors

4. **Console.log / Debug Audit**
   - Search for debug logging in source files
   - Report locations

5. **AI Slop Audit**
   - Search for AI-generated placeholder comments: `// TODO: implement`, `// Add your code here`, `// ... rest of`, `/* placeholder */`
   - Search for hallucinated imports (modules that don't exist in node_modules / project)
   - Search for dead code: empty function bodies, unreachable returns, unused variables
   - Report locations with suggested removals

6. **Git Status**
   - Show uncommitted changes
   - Show files modified since last commit

## Output

Produce a concise CDD verification report:

```
CDD VERIFICATION: [PASS/FAIL]

Compile:  [OK/FAIL] (Exit Code: X)
Types:    [OK/X errors]
LSP:      [OK/X errors]
Lint:     [OK/X issues]
Secrets:  [OK/X found]
Logs:     [OK/X debug statements]
AI Slop:  [OK/X placeholder comments / X dead code]

Ready for PR: [YES/NO]
```

If any critical issues, list them with fix suggestions.
If compile fails, auto-invoke **build-error-resolver** agent.

## Arguments

$ARGUMENTS can be:
- `quick` - Only compile + types
- `full` - All checks (default)
- `pre-commit` - Checks relevant for commits
- `pre-pr` - Full checks plus security scan
