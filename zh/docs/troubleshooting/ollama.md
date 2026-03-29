# Ollama 故障排除

## 流式传输问题：JSON 解析错误

### 问题

当使用 Ollama 作为 oh-my-openagent 代理的提供商时，你可能会遇到：

```
JSON Parse error: Unexpected EOF
```

当代理尝试工具调用（例如，使用 `mcp_grep_search` 的 `explore` 代理）时会发生这种情况。

### 根本原因

当在 API 请求中使用 `stream: true` 时，Ollama 会返回 **NDJSON**（换行符分隔的 JSON）：

```json
{"message":{"tool_calls":[{"function":{"name":"read","arguments":{"filePath":"README.md"}}}]}, "done":false}
{"message":{"content":""}, "done":true}
```

Claude Code SDK 期望单个 JSON 对象，而不是多行 NDJSON，从而导致解析错误。

**发生这种情况的原因：**
- **Ollama API**：根据设计，将流式响应作为 NDJSON 返回
- **Claude Code SDK**：没有正确处理用于工具调用的 NDJSON 响应
- **oh-my-openagent**：传递 SDK 的行为（无法在此层修复）

## 解决方案

### 选项 1：禁用流式传输（推荐）

将你的 Ollama 提供商配置为使用 `stream: false`：

```json
{
  "provider": "ollama",
  "model": "qwen3-coder",
  "stream": false
}
```

**优点：**
- 立即生效
- 无需更改代码
- 简单的配置

**缺点：**
- 响应时间略慢（无流式传输）
- 交互式反馈较少

### 选项 2：仅使用非工具代理

如果你需要流式传输，请避免使用调用工具的代理：

- **安全**：简单的文本生成，非工具任务
- **有问题**：任何带有工具调用的代理（explore、librarian 等）

### 选项 3：等待 SDK 修复

正确的修复需要 Claude Code SDK 做到：

1. 检测 NDJSON 响应
2. 分别解析每一行
3. 合并多行中的 `tool_calls`
4. 返回单个合并的响应

**跟踪**：https://github.com/code-yeongyu/oh-my-openagent/issues/1124

## 变通实现

在 SDK 修复之前，以下是如何实现 NDJSON 解析（供 SDK 维护者参考）：

```typescript
async function parseOllamaStreamResponse(response: string): Promise<object> {
  const lines = response.split('\n').filter(line => line.trim());
  const mergedMessage = { tool_calls: [] };

  for (const line of lines) {
    try {
      const json = JSON.parse(line);
      if (json.message?.tool_calls) {
        mergedMessage.tool_calls.push(...json.message.tool_calls);
      }
      if (json.message?.content) {
        mergedMessage.content = json.message.content;
      }
    } catch (e) {
      // 跳过格式错误的行
      console.warn('Skipping malformed NDJSON line:', line);
    }
  }

  return mergedMessage;
}
```

## 测试

验证修复是否有效：

```bash
# 用 curl 测试 (在使用 stream: false 时应该可以工作)
curl -s http://localhost:11434/api/chat \
  -d '{
    "model": "qwen3-coder",
    "messages": [{"role": "user", "content": "Read file README.md"}],
    "stream": false,
    "tools": [{"type": "function", "function": {"name": "read", "description": "Read a file", "parameters": {"type": "object", "properties": {"filePath": {"type": "string"}}, "required": ["filePath"]}}}]
  }'
```

## 相关问题

- **oh-my-openagent**: https://github.com/code-yeongyu/oh-my-openagent/issues/1124
- **Ollama API 文档**: https://github.com/ollama/ollama/blob/main/docs/api.md

## 获取帮助

如果你遇到此问题：

1. 检查你的 Ollama 提供商配置
2. 将 `stream: false` 设置为变通方法
3. 将任何其他错误报告给问题跟踪器
4. 提供你的配置（不含机密信息）以进行调试
