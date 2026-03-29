# src/tools/background-task/ — 后台任务工具包装器

**生成日期:** 2026-03-06

## 概览

18 个文件。`background_output` 和 `background_cancel` 的工具层包装器。**不**实现后台执行引擎 — 引擎位于 `src/features/background-agent/` 中。此目录提供面向 LLM 的工具接口。

## 三个工具

| 工具 | 工厂函数 | 用途 |
|------|---------|---------|
| `background_output` | `createBackgroundOutput` | 从正在运行/已完成的后台任务获取结果 |
| `background_cancel` | `createBackgroundCancel` | 取消正在运行的任务 |
| `createBackgroundTask` | 内部使用 | 两者共享的工厂函数 |

## 关键文件

| 文件 | 用途 |
|------|---------|
| `create-background-output.ts` | `background_output` 工具：按 task_id 获取任务结果 |
| `create-background-cancel.ts` | `background_cancel` 工具：按 taskId 取消，或使用 all=true 取消全部 |
| `create-background-task.ts` | 带有公共参数的共享工具工厂 |
| `clients.ts` | 后台输出和取消的客户端接口 |
| `session-messages.ts` | 从 OpenCode 获取会话消息 |
| `full-session-format.ts` | 格式化完整会话输出（消息，思考块） |
| `task-result-format.ts` | 格式化任务结果以供 LLM 消费 |
| `task-status-format.ts` | 格式化任务状态（运行中/已完成/错误） |
| `message-dir.ts` | 用于消息交换的临时目录 |
| `truncate-text.ts` | 截断大输出以适应上下文 |
| `time-format.ts` | 人类可读的持续时间格式化 |
| `delay.ts` | 轮询延迟实用工具 |
| `types.ts` | `BackgroundTaskOptions`、结果/状态类型 |
| `constants.ts` | 默认超时时间、轮询间隔 |

## 后台输出模式

```
background_output(task_id, block=false)  → 检查当前状态/结果
background_output(task_id, block=true)   → 等待直到完成（默认超时时间：120 秒）
background_output(task_id, full_session=true) → 返回完整的会话转录文本
background_output(task_id, message_limit=N) → 仅最后 N 条消息
background_output(task_id, include_thinking=true) → 包含思考块
```

## 与后台引擎的关系

```
tools/background-task/  ← LLM 工具接口
features/background-agent/  ← 执行引擎 (BackgroundManager)
```

`createBackgroundOutput` 查询 `BackgroundManager.getTask(task_id)` — 它不管理任务状态。