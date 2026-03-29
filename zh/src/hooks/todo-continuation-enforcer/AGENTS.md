# src/hooks/todo-continuation-enforcer/ — Boulder 延续机制

**生成日期:** 2026-03-06

## 概览

14 个文件（约 2061 行代码）。“Boulder” — 延续层（Continuation Tier）钩子，当存在未完成的待办事项时，强制 Sisyphus 继续执行。在 `session.idle` 时触发，在 2 秒倒计时提示后注入延续提示词。

## 它的工作原理

```
session.idle
  → 是主会话吗（非 prometheus/compaction）？(DEFAULT_SKIP_AGENTS)
  → 最近没有检测到中断？(ABORT_WINDOW_MS = 3s)
  → 待办事项仍未完成？(todo.ts)
  → 没有正在运行的后台任务？
  → 冷却时间已过？(CONTINUATION_COOLDOWN_MS = 30s)
  → 失败次数 < 最大值？(MAX_CONSECUTIVE_FAILURES = 5)
  → 开始 2 秒倒计时提示 → 注入 CONTINUATION_PROMPT
```

## 关键文件

| 文件 | 用途 |
|------|---------|
| `handler.ts` | `createTodoContinuationHandler()` — 事件路由器，委托给 idle/non-idle 处理器 |
| `idle-event.ts` | `handleSessionIdle()` — session.idle 的主决策门控 |
| `non-idle-events.ts` | `handleNonIdleEvent()` — 处理 session.error（中断检测） |
| `session-state.ts` | `SessionStateStore` — 每会话失败/中断/冷却状态 |
| `todo.ts` | 通过会话存储检查待办事项完成状态 |
| `countdown.ts` | 注入前的 2 秒倒计时提示 |
| `abort-detection.ts` | 检测 MessageAbortedError / AbortError |
| `continuation-injection.ts` | 构建并向会话中注入 CONTINUATION_PROMPT |
| `message-directory.ts` | 用于消息注入交换的临时目录 |
| `constants.ts` | 时间常量，CONTINUATION_PROMPT，跳过的代理 |
| `types.ts` | `SessionState`，处理器参数类型 |

## 常量

```typescript
DEFAULT_SKIP_AGENTS = ["prometheus", "compaction", "plan"]
CONTINUATION_COOLDOWN_MS = 30_000     // 两次注入之间 30 秒
MAX_CONSECUTIVE_FAILURES = 5          // 然后暂停 5 分钟（指数退避）
FAILURE_RESET_WINDOW_MS = 5 * 60_000  // 用于重置失败次数的 5 分钟窗口
COUNTDOWN_SECONDS = 2
ABORT_WINDOW_MS = 3000                // 中断信号后的宽限期
```

## 每个会话的状态

```typescript
interface SessionState {
  failureCount: number       // 连续失败次数
  lastFailureAt?: number     // 时间戳
  abortDetectedAt?: number   // 在 ABORT_WINDOW_MS 后重置
  cooldownUntil?: number     // 在此之后允许下一次注入
  countdownTimer?: Timer     // 活动倒计时的引用
}
```

## 与 ATLAS 的关系

`todoContinuationEnforcer` **仅**处理**主 Sisyphus 会话**。
`atlasHook` 使用不同的决策门控处理 **boulder/ralph/subagent 会话**。
两者都在 `session.idle` 时触发，但会首先检查会话类型。