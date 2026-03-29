# src/features/tmux-subagent/ — Tmux 窗格管理

**生成日期：** 2026-03-06

## 概览

28 个文件。状态优先的 tmux 集成，管理用于后台代理会话的窗格。处理分割决策、网格规划、轮询和生命周期事件。

## 核心架构

```
TmuxSessionManager (manager.ts)
  ├─→ DecisionEngine: 我们应该生成/关闭窗格吗？
  ├─→ ActionExecutor: 执行生成/关闭/替换操作
  ├─→ PollingManager: 监控窗格健康状况
  └─→ EventHandlers: 响应会话创建/删除事件
```

## 关键文件

| 文件 | 目的 |
|------|---------|
| `manager.ts` | `TmuxSessionManager` — 主类，会话跟踪，事件路由 |
| `decision-engine.ts` | 评估窗口状态 → 生成带有操作的 `SpawnDecision` |
| `action-executor.ts` | 执行 `PaneAction[]` (关闭, 生成, 替换) |
| `grid-planning.ts` | 给定窗口尺寸计算窗格布局 |
| `spawn-action-decider.ts` | 决定生成 vs 替换 vs 跳过 |
| `spawn-target-finder.ts` | 寻找用于分割或替换的最佳窗格 |
| `polling-manager.ts` | 被跟踪会话的健康状况轮询 |
| `types.ts` | `TrackedSession`, `WindowState`, `PaneAction`, `SpawnDecision` |

## 窗格生命周期

```
session.created → spawn-action-decider → grid-planning → action-executor → 跟踪会话
session.deleted → 清理被跟踪会话 → 如果为空则关闭窗格
```

## 布局约束

- `MIN_PANE_WIDTH`: 52 个字符
- `MIN_PANE_HEIGHT`: 11 行
- 保留主窗格 (绝不分割至低于最小值)
- 代理窗格从剩余空间中分割出来

## 事件处理程序

| 文件 | 事件 |
|------|-------|
| `session-created-handler.ts` | 新的后台会话 → 生成窗格 |
| `session-deleted-handler.ts` | 会话结束 → 关闭窗格 |
| `session-created-event.ts` | 事件类型定义 |
