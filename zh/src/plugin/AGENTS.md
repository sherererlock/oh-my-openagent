# src/plugin/ — 8 个 OpenCode 钩子处理器 + 钩子组合

**生成日期:** 2026-03-06

## 概览

核心胶水层。20 个源文件，组装了 8 个 OpenCode 钩子处理器，并将 48 个钩子组合到 `PluginInterface` 中。每个处理器文件对应一种 OpenCode 钩子类型。

## 处理器文件

| 文件 | OpenCode 钩子 | 用途 |
|------|---------------|---------|
| `chat-message.ts` | `chat.message` | 首条消息变体，会话设置，关键字检测 |
| `chat-params.ts` | `chat.params` | Anthropic 努力程度级别，思考模式 (think mode) |
| `event.ts` | `event` | 会话生命周期（创建、删除、空闲、错误） |
| `tool-execute-before.ts` | `tool.execute.before` | 工具前守卫（文件守卫，标签截断，规则注入） |
| `tool-execute-after.ts` | `tool.execute.after` | 工具后钩子（输出截断，注释检查，元数据） |
| `messages-transform.ts` | `experimental.chat.messages.transform` | 上下文注入，思考块验证 |
| `tool-registry.ts` | `tool` | 由工厂函数组装的 26 个工具 |
| `chat-headers.ts` | `chat.headers` | Copilot x-initiator 请求头注入 |
| `skill-context.ts` | — | 用于工具创建的技能/浏览器/分类上下文 |

## 钩子组合 (hooks/ 子目录)

| 文件 | 层级 | 数量 |
|------|------|-------|
| `create-session-hooks.ts` | 会话 (Session) | 23 |
| `create-tool-guard-hooks.ts` | 工具守卫 (Tool Guard) | 12 |
| `create-skill-hooks.ts` | 技能 (Skill) | 2 |
| `create-core-hooks.ts` | 聚合器 (Aggregator) | 会话 + 守卫 + 转换 = 39 |

## 支持文件

| 文件 | 用途 |
|------|---------|
| `available-categories.ts` | 构建 `AvailableCategory[]` 以进行代理提示词注入 |
| `session-agent-resolver.ts` | 解析哪个代理拥有该会话 |
| `session-status-normalizer.ts` | 跨 OpenCode 版本标准化会话状态 |
| `recent-synthetic-idles.ts` | 快速空闲事件去重 |
| `unstable-agent-babysitter.ts` | 跨会话跟踪不稳定代理的行为 |
| `types.ts` | `PluginContext`、`PluginInterface`、`ToolsRecord`、`TmuxConfig` |
| `ultrawork-model-override.ts` | Ultrawork 模式模型覆盖逻辑 |
| `ultrawork-db-model-override.ts` | Ultrawork 的数据库级模型覆盖 |
| `config-handler.ts` | 运行时配置加载和缓存 |

## 关键模式

- 每个处理器导出一个函数，接收 `(hookRecord, ctx, pluginConfig, managers)` → 返回 OpenCode 钩子函数
- 处理器遍历钩子记录，按顺序调用每个钩子 `(input, output)`
- 组合文件中的 `safeHook()` 包装器可捕获每个钩子的错误，而不会破坏执行链
- 工具注册表在返回之前使用 `filterDisabledTools()`