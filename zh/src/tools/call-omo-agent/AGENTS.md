# src/tools/call-omo-agent/ — 直接代理调用工具

**生成日期:** 2026-03-06

## 概览

23 个文件。`call_omo_agent` 工具 — 直接调用指定的代理（仅限 explore，librarian）。与 `delegate-task` 不同：没有分类系统，不加载技能，没有模型选择。固定的代理集合，相同的执行模式（后台/同步）。

## 与 delegate-task 的区别

| 方面 | `call_omo_agent` | `delegate-task` (`task`) |
|--------|-----------------|--------------------------|
| 代理选择 | 命名代理 (explore/librarian) | 分类或 subagent_type |
| 技能加载 | 无 | 支持 `load_skills[]` |
| 模型选择 | 来自代理的后备链 | 来自分类配置 |
| 用例 | 快速的上下文 grep | 带有技能的完全委托 |

## 允许的代理

仅限 `explore` 和 `librarian` — 通过 `constants.ts` 中的 `ALLOWED_AGENTS` 常量强制执行。不区分大小写的验证。

## 执行模式

与 delegate-task 相同的两种模式：

| 模式 | 文件 | 描述 |
|------|------|-------------|
| **后台 (Background)** | `background-agent-executor.ts` | 通过 `BackgroundManager` 异步执行 |
| **同步 (Sync)** | `sync-executor.ts` | 创建会话 → 等待空闲 → 返回结果 |

## 关键文件

| 文件 | 用途 |
|------|---------|
| `tools.ts` | `createCallOmoAgent()` 工厂 — 验证代理，路由到执行器 |
| `background-executor.ts` | 根据 `run_in_background` 路由到后台或同步 |
| `background-agent-executor.ts` | 通过 `BackgroundManager.launch()` 启动 |
| `sync-executor.ts` | 同步会话：创建 → 发送提示词 → 轮询 → 获取结果 |
| `session-creator.ts` | 创建用于同步执行的 OpenCode 会话 |
| `subagent-session-creator.ts` | 使用代理特定的配置创建会话 |
| `subagent-session-prompter.ts` | 向会话中注入提示词 |
| `completion-poller.ts` | 轮询直到会话空闲 |
| `session-completion-poller.ts` | 会话特定的完成检查 |
| `session-message-output-extractor.ts` | 提取最后一条助手消息作为结果 |
| `message-processor.ts` | 处理原始消息内容 |
| `message-dir.ts` + `message-storage-directory.ts` | 用于消息交换的临时存储 |
| `types.ts` | `CallOmoAgentArgs`、`AllowedAgentType`、`ToolContextWithMetadata` |

## 会话延续

传递 `session_id` 以恢复现有会话，而不是创建新会话 — 在两个执行器中均有处理。