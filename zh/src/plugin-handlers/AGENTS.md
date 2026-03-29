# src/plugin-handlers/ — 6 阶段配置加载流水线

**生成日期:** 2026-03-06

## 概览

13 个非测试文件，实现了 `ConfigHandler` — 即 `config` 钩子处理器。执行 6 个顺序阶段，向 OpenCode 注册代理、工具、MCP 和命令。

## 6 阶段流水线

| 阶段 | 处理器 | 用途 |
|-------|---------|---------|
| 1 | `applyProviderConfig` | 缓存模型上下文限制，检测 anthropic-beta 请求头 |
| 2 | `loadPluginComponents` | 发现 Claude Code 插件（10 秒超时，错误隔离） |
| 3 | `applyAgentConfig` | 从 5 个来源加载代理，技能发现，计划降级 (plan demotion) |
| 4 | `applyToolConfig` | 特定于代理的工具权限 |
| 5 | `applyMcpConfig` | 合并内置 + CC + 插件 MCP |
| 6 | `applyCommandConfig` | 从 9 个并行来源合并命令/技能 |

## 文件

| 文件 | 行数 | 用途 |
|------|-------|---------|
| `config-handler.ts` | ~200 | 主编排器，6 阶段顺序执行 |
| `plugin-components-loader.ts` | ~100 | CC 插件发现（10 秒超时） |
| `agent-config-handler.ts` | ~300 | 从 5 个来源进行代理加载 + 技能发现 |
| `mcp-config-handler.ts` | ~150 | 合并内置 + CC + 插件 MCP |
| `command-config-handler.ts` | ~200 | 9 个用于命令/技能的并行来源 |
| `tool-config-handler.ts` | ~100 | 特定于代理的工具授权/拒绝 |
| `provider-config-handler.ts` | ~80 | 提供者配置 + 模型缓存 |
| `prometheus-agent-config-builder.ts` | ~100 | 带有模型解析的 Prometheus 配置 |
| `plan-model-inheritance.ts` | 28 | 计划降级逻辑 |
| `agent-priority-order.ts` | ~30 | sisyphus, hephaestus, prometheus, atlas 优先 |
| `agent-key-remapper.ts` | ~30 | 代理键 → 显示名称 |
| `category-config-resolver.ts` | ~40 | 用户与默认分类查找 |
| `index.ts` | ~10 | 桶导出 (Barrel exports) |

## 工具权限

| 代理 | 授予 | 拒绝 |
|-------|---------|--------|
| Librarian | grep_app_* | — |
| Atlas, Sisyphus, Prometheus | task, task_*, teammate | — |
| Hephaestus | task | — |
| Default (所有其他) | — | grep_app_*, task_*, teammate, LSP |

## 多级配置合并

```
用户 (~/.config/opencode/oh-my-opencode.jsonc)
  ↓ 深度合并 (deepMerge)
项目 (.opencode/oh-my-opencode.jsonc)
  ↓ Zod 默认值
最终配置
```

- `agents`、`categories`、`claude_code`：深度合并
- `disabled_*` 数组：集合并集（去重）