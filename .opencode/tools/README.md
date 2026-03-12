# OpenCode Custom Tools

自定义工具目录。可在此创建 AI 助手可调用的自定义工具。

## 格式

使用 TypeScript/JavaScript 编写，导出 `tool()` 定义：

```typescript
import { tool } from "@opencode-ai/plugin";

export default tool({
  name: "my-tool",
  description: "My custom tool description",
  parameters: {
    // JSON Schema 参数定义
  },
  async execute(params) {
    // 工具实现
    return { result: "..." };
  }
});
```

参考文档：https://opencode.ai/docs/custom-tools
