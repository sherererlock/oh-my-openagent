# src/features/background-agent/ — 核心编排引擎

**生成时间:** 2026-03-06

## 概览

30 个文件（约 10k 行代码）。管理异步任务生命周期：启动 (launch) → 排队 (queue) → 运行 (run) → 轮询 (poll) → 完成/错误 (complete/error)。基于每个模型/供应商限制并发（默认为 5）。是多智能体编排的核心。

## 任务生命周期

```
LaunchInput (启动输入) → pending (等待) → [ConcurrencyManager 队列] → running (运行中) → polling (轮询中) → completed (完成)/error (错误)/cancelled (取消)/interrupt (中断)
```

## 核心文件

| 文件 | 用途 |
|------|---------|
| `manager.ts` | `BackgroundManager` — 主类：启动 (launch)、取消 (cancel)、获取任务 (getTask)、列出任务 (listTasks) |
| `spawner.ts` | 任务生成：创建会话 → 注入提示词 → 开始轮询 |
| `concurrency.ts` | `ConcurrencyManager` — 按并发键划分的 FIFO 队列，负责槽位 (slot) 的获取和释放 |
| `task-poller.ts` | 3 秒间隔轮询，通过空闲事件 + 稳定性检测 (10 秒未改变) 来判定完成 |
| `result-handler.ts` | 处理已完成的任务：提取结果、通知父级、清理资源 |
| `state.ts` | 内存中的任务存储 (基于 Map) |
| `types.ts` | `BackgroundTask`, `LaunchInput`, `ResumeInput`, `BackgroundTaskStatus` |

## SPAWNER 子目录 (6 个文件)

| 文件 | 用途 |
|------|---------|
| `spawner-context.ts` | 组合所有生成器依赖项的 `SpawnerContext` 接口 |
| `background-session-creator.ts` | 为后台任务创建 OpenCode 会话 |
| `concurrency-key-from-launch-input.ts` | 从模型/供应商派生并发键 |
| `parent-directory-resolver.ts` | 为子会话解析工作目录 |
| `tmux-callback-invoker.ts` | 在会话创建时通知 TmuxSessionManager |

## 完成检测

结合了两个信号：
1. **会话空闲事件 (Session idle event)** — OpenCode 报告会话变为空闲状态
2. **稳定性检测 (Stability detection)** — 消息计数在 10 秒内未改变（在 3 秒间隔的轮询中连续 3 次及以上保持稳定）

两者必须同时满足才能将任务标记为完成。防止在短暂的停顿中过早完成。

## 并发模型

- 键格式：`{providerID}/{modelID}` (例如，`anthropic/claude-opus-4-6`)
- 默认限制：每个键 5 个并发（可通过 `background_task` 配置项进行配置）
- FIFO 队列：当槽位满时，任务按顺序等待
- 槽位释放时机：任务完成、出错、被取消

## 通知流程

```
任务完成 → result-handler (结果处理器) → parent-session-notifier (父会话通知器) → 将系统消息注入到父会话中
```
