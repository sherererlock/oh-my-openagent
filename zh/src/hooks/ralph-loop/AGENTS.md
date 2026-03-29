# src/hooks/ralph-loop/ — 自我参照开发循环

**生成日期:** 2026-03-06

## 概览

14 个文件（约 1687 行代码）。`ralphLoop` 会话层（Session Tier）钩子 — 为 `/ralph-loop` 命令提供支持。迭代执行开发循环，直到代理发出 `<promise>DONE</promise>` 或达到最大迭代次数。

## 循环生命周期

```
/ralph-loop → startLoop(sessionID, prompt, options)
  → loopState.startLoop() → 将状态持久化到 .sisyphus/ralph-loop.local.md
  → session.idle 事件 → createRalphLoopEventHandler()
    → completionPromiseDetector: 扫描输出以查找 <promise>DONE</promise>
    → 如果未完成: 注入延续提示词 → 循环
    → 如果完成或达到最大迭代次数: cancelLoop()
```

## 关键文件

| 文件 | 用途 |
|------|---------|
| `ralph-loop-hook.ts` | `createRalphLoopHook()` — 组合控制器 + 恢复机制 + 事件处理器 |
| `ralph-loop-event-handler.ts` | `createRalphLoopEventHandler()` — 处理 session.idle，驱动循环 |
| `loop-state-controller.ts` | 状态 CRUD：startLoop、cancelLoop、getState，持久化到磁盘 |
| `loop-session-recovery.ts` | 从崩溃/中断的循环会话中恢复 |
| `completion-promise-detector.ts` | 扫描会话转录文本以查找 `<promise>DONE</promise>` |
| `continuation-prompt-builder.ts` | 为下一次迭代构建延续消息 |
| `continuation-prompt-injector.ts` | 将构建好的提示词注入到活动会话中 |
| `storage.ts` | 读写 `.sisyphus/ralph-loop.local.md` 状态文件 |
| `message-storage-directory.ts` | 用于提示词注入的临时目录 |
| `with-timeout.ts` | 带有超时的 API 调用包装器（默认 5000ms） |
| `types.ts` | `RalphLoopState`、`RalphLoopOptions`、循环迭代类型 |

## 状态文件

```
.sisyphus/ralph-loop.local.md  (git 忽略)
  → sessionID, prompt, iteration count, maxIterations, completionPromise, ultrawork flag
```

## 选项

```typescript
startLoop(sessionID, prompt, {
  maxIterations?: number  // 默认为配置中的值（默认：100）
  completionPromise?: string  // 自定义“完成”信号（默认："<promise>DONE</promise>"）
  ultrawork?: boolean  // 为迭代启用 ultrawork 模式
})
```

## 导出的接口

```typescript
interface RalphLoopHook {
  event: (input) => Promise<void>  // session.idle 处理器
  startLoop: (sessionID, prompt, options?) => boolean
  cancelLoop: (sessionID) => boolean
  getState: () => RalphLoopState | null
}
```