# src/hooks/atlas/ — 主 Boulder 编排器

**生成日期:** 2026-03-06

## 概览

17 个文件（约 1976 行代码）。`atlasHook` — 延续层（Continuation Tier）钩子，用于监听 `session.idle` 事件，并在 boulder 会话（ralph-loop、任务生成的代理）有未完成的工作时强制其继续执行。同时它还负责对子代理（subagent）会话执行写入/编辑策略。

## ATLAS 的功能

Atlas 是“会话的守护者”——它跟踪每个会话并决定：
1. 这个会话是否应该被强制继续？（如果是带有未完成待办事项的 boulder 会话）
2. 是否应该阻止写入/编辑操作？（针对特定会话类型的策略执行）
3. 是否应该注入验证提醒？（在工具执行后）

## 决策门控 (session.idle)

```
session.idle 事件
  → 这是一个 boulder/ralph/atlas 会话吗？(session-last-agent.ts)
  → 是否有中断信号？(is-abort-error.ts)
  → 失败次数 < 最大值？(state.promptFailureCount)
  → 没有正在运行的后台任务？
  → 代理匹配预期？(recent-model-resolver.ts)
  → 计划已完成？(待办事项状态)
  → 冷却时间已过？(两次注入间 5 秒)
  → 注入延续提示词 (boulder-continuation-injector.ts)
```

## 关键文件

| 文件 | 用途 |
|------|---------|
| `atlas-hook.ts` | `createAtlasHook()` — 组合事件 + 工具处理器，维护会话状态 |
| `event-handler.ts` | `createAtlasEventHandler()` — session.idle 事件的决策门控 |
| `boulder-continuation-injector.ts` | 构建并向会话中注入延续提示词 |
| `system-reminder-templates.ts` | 延续提醒消息的模板 |
| `tool-execute-before.ts` | 根据会话策略阻止写入/编辑 |
| `tool-execute-after.ts` | 在工具后注入验证提醒 |
| `write-edit-tool-policy.ts` | 策略：哪些会话可以进行写入/编辑？ |
| `verification-reminders.ts` | 用于验证工作的提醒内容 |
| `session-last-agent.ts` | 确定哪个代理拥有该会话 |
| `recent-model-resolver.ts` | 解析最近消息中使用的模型 |
| `subagent-session-id.ts` | 检测会话是否为子代理会话 |
| `sisyphus-path.ts` | 解析 `.sisyphus/` 目录路径 |
| `is-abort-error.ts` | 检测会话输出中的中断信号 |
| `types.ts` | `SessionState`、`AtlasHookOptions`、`AtlasContext` |

## 每个会话的状态

```typescript
interface SessionState {
  promptFailureCount: number  // 在延续失败时递增
  // 在成功延续时重置
}
```

在暂停 5 分钟之前的最大连续失败次数：5（在 todo-continuation-enforcer 中呈指数退避）。

## 与其他钩子的关系

- **atlasHook** (延续层): 主编排器，处理 boulder 会话
- **todoContinuationEnforcer** (延续层): 针对主 Sisyphus 会话的“Boulder”机制
- 两者都注入到 `session.idle` 中，但服务于不同的会话类型