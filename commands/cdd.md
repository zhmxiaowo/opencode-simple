---
description: 快速 CDD 编译验证。只检查编译是否通过和 LSP 是否干净。相当于 /verify quick 的快捷方式。
---

# /cdd — 快速编译驱动验证

一键运行 CDD 验证循环。

## 工作流程

### 1. 自动检测项目类型

扫描当前目录，识别项目类型：

| 文件 | 项目类型 | 编译命令 |
|------|---------|---------|
| `tsconfig.json` | TypeScript | `npx tsc --noEmit --pretty` |
| `package.json` (build) | Node.js | `npm run build` |
| `*.csproj` | C# / Unity | `dotnet build --no-restore` |
| `*.uproject` | Unreal C++ | `UnrealBuildTool` |
| `Cargo.toml` | Rust | `cargo check` |
| `go.mod` | Go | `go build ./...` |
| `pyproject.toml` | Python | `pyright .` |
| `CMakeLists.txt` | C/C++ | `cmake --build build/` |

### 2. 运行编译

执行对应的编译命令，捕获输出。

### 3. 报告结果

```
CDD CHECK: [PASS ✅ / FAIL ❌]
Exit Code: X
Errors: X
Warnings: X

[如果失败，显示前 10 条错误]
```

### 4. 失败时自动建议

如果编译失败：
- 提示用户运行 `/build-fix` 进入 CDD 修复循环
- 或手动调用 `build-error-resolver` agent

## 参数

- `/cdd` — 默认编译检查
- `/cdd full` — 编译 + LSP + Lint（等同于 `/verify full`）
