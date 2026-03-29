# src/hooks/anthropic-context-window-limit-recovery/ — 多策略上下文恢复

**生成日期：** 2026-03-06

## 概览

31 个文件（约 2232 行代码）。最复杂的钩子。通过按顺序应用的多项策略从上下文窗口限制错误中恢复。

## 恢复策略 (按优先级顺序)

| 策略 | 文件 | 机制 |
|----------|------|-----------|
| **空内容恢复** | `empty-content-recovery.ts` | 处理消息中的空/null 内容块 |
| **去重** | `deduplication-recovery.ts` | 从上下文中删除重复的工具结果 |
| **目标令牌截断** | `target-token-truncation.ts` | 截断最大的工具输出以符合目标比例 |
| **激进截断** | `aggressive-truncation-strategy.ts` | 最后的手段截断，以最少的输出保留 |
| **总结重试** | `summarize-retry-strategy.ts` | 压缩 + 总结，然后重试 |

## 关键文件

| 文件 | 目的 |
|------|---------|
| `recovery-hook.ts` | 主钩子入口 — `session.error` 处理器，策略编排 |
| `executor.ts` | 按顺序执行恢复策略 |
| `parser.ts` | 解析 Anthropic 令牌限制错误消息 |
| `state.ts` | `AutoCompactState` — 每个会话的重试/截断跟踪 |
| `types.ts` | `ParsedTokenLimitError`, `RetryState`, `TruncateState`, 配置常量 |
| `storage.ts` | 持久化工具结果以便后续截断 |
| `tool-result-storage.ts` | 存储/检索单个工具调用结果 |
| `message-builder.ts` | 恢复后构建重试消息 |

## 重试配置

- 最大尝试次数：2
- 初始延迟：2秒，退避 ×2，最大 30秒
- 最大截断尝试次数：20
- 目标令牌比例：0.5 (截断至限制的 50%)
- 每个令牌预估字符数：4

## 修剪系统

`pruning-*.ts` 文件处理智能输出修剪：
- `pruning-deduplication.ts` — 删除跨工具结果的重复内容
- `pruning-tool-output-truncation.ts` — 截断过大的工具输出
- `pruning-types.ts` — 修剪特定的类型定义

## SDK 变体

`empty-content-recovery-sdk.ts` 和 `tool-result-storage-sdk.ts` 提供了用于 OpenCode 客户端交互的基于 SDK 的实现。
