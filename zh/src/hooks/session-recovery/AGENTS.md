# src/hooks/session-recovery/ — 自动会话错误恢复

**生成日期:** 2026-03-06

## 概览

16 个文件 + `storage/` 子目录。处理 `session.error` 事件的会话层（Session Tier）钩子。检测可恢复的错误类型，应用针对性的恢复策略，并透明地恢复会话。

## 恢复策略

| 错误类型 | 文件 | 恢复动作 |
|------------|------|-----------------|
| `tool_result_missing` | `recover-tool-result-missing.ts` | 从存储中重建丢失的工具结果 |
| `thinking_block_order` | `recover-thinking-block-order.ts` | 重新排序格式错误的思考块 (thinking blocks) |
| `thinking_disabled_violation` | `recover-thinking-disabled-violation.ts` | 在禁用思考时剥离思考块 |
| `empty_content_message` | `recover-empty-content-message*.ts` | 处理空/null 内容块 |

## 关键文件

| 文件 | 用途 |
|------|---------|
| `hook.ts` | `createSessionRecoveryHook()` — 错误检测，策略分发，恢复 |
| `detect-error-type.ts` | `detectErrorType(error)` → `RecoveryErrorType \| null` |
| `resume.ts` | `resumeSession()` — 重建会话上下文，触发重试 |
| `storage.ts` | 用于恢复重建的每会话消息存储 |
| `recover-tool-result-missing.ts` | 根据存储的元数据重建工具结果 |
| `recover-thinking-block-order.ts` | 修复格式错误的思考块序列 |
| `recover-thinking-disabled-violation.ts` | 从模型上下文中移除思考块 |
| `recover-empty-content-message.ts` | 处理空的助手消息 |
| `recover-empty-content-message-sdk.ts` | 用于空内容恢复的 SDK 变体 |
| `types.ts` | `StoredMessageMeta`、`StoredPart`、`ResumeConfig`、`MessageData` |

## 存储子目录

```
storage/
  ├── message-store.ts    # 内存 + 文件消息缓存
  ├── part-store.ts       # 单个消息片段存储
  └── index.ts            # 桶导出 (Barrel export)
```

每个会话存储消息元数据和片段，用于恢复重建。

## 钩子接口

```typescript
interface SessionRecoveryHook {
  handleSessionRecovery: (info: MessageInfo) => Promise<boolean>
  isRecoverableError: (error: unknown) => boolean
  setOnAbortCallback: (cb: (sessionID: string) => void) => void
  setOnRecoveryCompleteCallback: (cb: (sessionID: string) => void) => void
}
```

## 注意事项

- 使用 `processingErrors` Set 作为守卫，以防止对相同错误进行重复的恢复尝试
- 支持使用 `experimental` 配置来进行行为标志控制
- 与 `anthropic-context-window-limit-recovery` 不同（后者处理 token 限制；此钩子处理结构性错误）