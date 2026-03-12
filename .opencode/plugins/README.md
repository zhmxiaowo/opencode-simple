# OpenCode Plugins

OpenCode 的插件系统，替代 Claude Code 的 Hooks 机制。

## 概述

插件放置在 `.opencode/plugins/` 目录下，使用 JS/TS 编写。

## 支持的事件

| 事件 | 触发时机 |
|------|----------|
| `tool.execute.before` | 工具执行前 |
| `tool.execute.after` | 工具执行后 |
| `session.created` | 会话创建时 |
| `session.idle` | 会话空闲时 |

## 示例

```typescript
import { plugin } from "@opencode-ai/plugin";

export default plugin({
  name: "example",
  subscribe: ["tool.execute.after"],
  async handle(event) {
    // 处理事件
  }
});
```

## 迁移说明

本项目的 `hooks/hooks.json` 使用 Claude Code Hook 格式。
如需在 OpenCode 中实现同等功能，请将 Hook 逻辑迁移为插件。

参考文档：https://opencode.ai/docs/plugins
