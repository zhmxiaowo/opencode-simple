# Build and Fix (CDD Mode)

Incrementally fix build and type errors with minimal, safe changes.
**CDD 原则**：编译失败时必须调用 sequential-thinking 分析，禁止盲目试错。

## Step 1: Detect Build System

Identify the project's build tool and run the build:

| Indicator | Build Command |
|-----------|---------------|
| `package.json` with `build` script | `npm run build` or `pnpm build` |
| `tsconfig.json` (TypeScript only) | `npx tsc --noEmit` |
| `Cargo.toml` | `cargo check 2>&1` |
| `pom.xml` | `mvn compile` |
| `build.gradle` | `./gradlew compileKotlin` |
| `go.mod` | `go build ./...` |
| `pyproject.toml` | `pyright .` or `mypy .` |
| `*.csproj` / `*.sln` | `dotnet build --no-restore` |
| `CMakeLists.txt` | `cmake --build build/` |
| `*.uproject` | `UnrealBuildTool` |

## Step 2: Parse and Group Errors

1. Run the build command and capture stderr
2. Group errors by file path
3. Sort by dependency order (fix imports/types before logic errors)
4. Count total errors for progress tracking

## Step 3: Fix Loop (CDD Mode — Think Before Fix)

For each error:

1. **Read the file** — Use Read tool to see error context (10 lines around the error)
2. **Think first** — Invoke `sequential-thinking` MCP to analyze root cause (禁止盲目试错)
3. **Diagnose** — Identify root cause (missing import, wrong type, syntax error)
4. **Fix minimally** — Use Edit tool for the smallest change that resolves the error
5. **Re-run build** — Verify the error is gone and no new errors introduced
6. **Move to next** — Continue with remaining errors

## Step 4: Guardrails

Stop and ask the user if:
- A fix introduces **more errors than it resolves**
- The **same error persists after 3 attempts** (likely a deeper issue)
- The fix requires **architectural changes** (not just a build fix)
- Build errors stem from **missing dependencies** (need `npm install`, `cargo add`, etc.)

## Step 5: Summary

Show results:
- Errors fixed (with file paths)
- Errors remaining (if any)
- New errors introduced (should be zero)
- Suggested next steps for unresolved issues

## Recovery Strategies

| Situation | Action |
|-----------|--------|
| Missing module/import | Check if package is installed; suggest install command |
| Type mismatch | Read both type definitions; fix the narrower type |
| Circular dependency | Identify cycle with import graph; suggest extraction |
| Version conflict | Check `package.json` / `Cargo.toml` for version constraints |
| Build tool misconfiguration | Read config file; compare with working defaults |

Fix one error at a time for safety. Prefer minimal diffs over refactoring.
