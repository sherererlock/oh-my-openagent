# src/mcp/ — 3 个内置的远程 MCP

**生成日期:** 2026-03-06

## 概览

三层 MCP 系统的第 1 层。通过 `createBuiltinMcps(disabledMcps, config)` 创建 3 个远程 HTTP MCP。

## 内置 MCP

| 名称 | URL | 环境变量 | 工具 |
|------|-----|----------|-------|
| **websearch** | `mcp.exa.ai` (默认) 或 `mcp.tavily.com` | `EXA_API_KEY` (可选)，`TAVILY_API_KEY` (如果是 tavily) | 网络搜索 |
| **context7** | `mcp.context7.com/mcp` | `CONTEXT7_API_KEY` (可选) | 库文档 |
| **grep_app** | `mcp.grep.app` | 无 | GitHub 代码搜索 |

## 注册模式

```typescript
// 静态导出 (context7, grep_app)
export const context7 = {
  type: "remote" as const,
  url: "https://mcp.context7.com/mcp",
  enabled: true,
  oauth: false as const,
}

// 带有配置的工厂函数 (websearch)
export function createWebsearchConfig(config?: WebsearchConfig): RemoteMcpConfig
```

## 启用/禁用

```jsonc
// 方法 1: disabled_mcps 数组
{ "disabled_mcps": ["websearch", "context7"] }

// 方法 2: enabled 标志
{ "mcp": { "websearch": { "enabled": false } } }
```

## 三层系统

| 层级 | 来源 | 机制 |
|------|--------|-----------|
| 1. 内置 | `src/mcp/` | 3 个远程 HTTP，由 `createBuiltinMcps()` 创建 |
| 2. Claude Code | `.mcp.json` | 通过 `claude-code-mcp-loader` 进行 `${VAR}` 展开 |
| 3. 技能嵌入 (Skill-embedded) | SKILL.md YAML | 由 `SkillMcpManager` 管理 (stdio + HTTP) |

## 文件

| 文件 | 用途 |
|------|---------|
| `index.ts` | `createBuiltinMcps()` 工厂函数 |
| `types.ts` | `McpNameSchema`: "websearch" \| "context7" \| "grep_app" |
| `websearch.ts` | 带有配置的 Exa/Tavily 提供者 |
| `context7.ts` | 带有可选认证头的 Context7 |
| `grep-app.ts` | Grep.app (无认证) |