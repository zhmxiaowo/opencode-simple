---
name: verification-loop
description: "CDD (Compile-Driven Development) verification system. No tests — compile + LSP is the sole truth."
origin: ECC-CDD
---

# CDD Verification Loop Skill

A compile-driven verification system. **不运行任何测试**。唯一的验证标准是编译通过 + LSP 无 Error。

## When to Use

Invoke this skill:
- After completing a feature or significant code change
- Before creating a PR
- When you want to ensure compile gates pass
- After refactoring

## Verification Phases

### Phase 1: Build / Compile Verification

自动检测项目类型并运行对应命令：

```bash
# TypeScript/Node.js
npx tsc --noEmit --pretty 2>&1 | head -30
# OR npm run build / pnpm build

# C# / Unity
dotnet build --no-restore 2>&1 | tail -20

# C++ / Unreal / CMake
cmake --build build/ 2>&1 | tail -20

# Rust
cargo check 2>&1 | tail -20

# Go
go build ./... 2>&1 | tail -20

# Python (type check)
pyright . 2>&1 | head -30
```

If build fails:
1. **STOP** — do not proceed
2. Invoke `sequential-thinking` for root cause analysis
3. Use **build-error-resolver** agent to fix
4. Re-run compile until Exit Code 0

### Phase 2: LSP Diagnostics Check

Use `mcp-language-server` to verify:
- 0 Error-level diagnostics
- Review Warning-level diagnostics (optional fix)

### Phase 3: Lint Check
```bash
# JavaScript/TypeScript
npm run lint 2>&1 | head -30

# Python
ruff check . 2>&1 | head -30
```

### Phase 4: Security Scan
```bash
# Check for secrets
grep -rn "sk-" --include="*.ts" --include="*.js" --include="*.cs" . 2>/dev/null | head -10
grep -rn "api_key" --include="*.ts" --include="*.js" --include="*.cs" . 2>/dev/null | head -10

# Check for debug logging
grep -rn "console.log\|Debug.Log\|UE_LOG" src/ 2>/dev/null | head -10
```

### Phase 5: Diff Review
```bash
# Show what changed
git diff --stat
git diff HEAD~1 --name-only
```

Review each changed file for:
- Unintended changes
- Missing error handling
- Potential edge cases

## Output Format

After running all phases, produce a CDD verification report:

```
CDD VERIFICATION REPORT
=======================

Compile:   [PASS/FAIL] (Exit Code: X)
Types/LSP: [PASS/FAIL] (X errors)
Lint:      [PASS/FAIL] (X warnings)
Security:  [PASS/FAIL] (X issues)
Diff:      [X files changed]

Overall:   [READY/NOT READY] for PR

Issues to Fix:
1. ...
2. ...
```

## Continuous Mode

For long sessions, run verification after major changes:

```markdown
Set a mental checkpoint:
- After completing each function
- After finishing a component
- Before moving to next task

Run: /verify
```

## Integration with Hooks

This skill complements PostToolUse hooks but provides deeper verification.
Hooks catch issues immediately; this skill provides comprehensive CDD review.
