# src/features/claude-tasks/ — 任务模式 + 存储

**生成日期：** 2026-03-06

## 概览

4 个非测试文件（约 622 行代码）。基于文件的任务持久化，具有原子写入、锁定和 OpenCode 待办事项 API 同步功能。

## 任务模式 (TASK SCHEMA)

```typescript
interface Task {
  id: string              // T-{uuid} 自动生成
  subject: string         // 简短标题
  description?: string    // 详细描述
  status: "pending" | "in_progress" | "completed" | "deleted"
  activeForm?: string     // 当前表单/模板
  blocks?: string[]       // 此任务阻塞的任务
  blockedBy?: string[]    // 阻塞此任务的任务
  owner?: string          // 代理/会话
  metadata?: Record<string, unknown>
  repoURL?: string        // 关联的仓库
  parentID?: string       // 父任务 ID
  threadID?: string       // 会话 ID (自动记录)
}
```

## 文件

| 文件 | 目的 |
|------|---------|
| `types.ts` | 任务接口 + 状态类型 |
| `storage.ts` | `readJsonSafe()`, `writeJsonAtomic()`, `acquireLock()`, `generateTaskId()` |
| `session-storage.ts` | 每个会话的任务存储，threadID 自动记录 |
| `index.ts` | 桶导出 (Barrel exports) |

## 存储

- 位置：`.sisyphus/tasks/` 目录
- 格式：JSON 文件，每个任务一个文件
- 原子写入：临时文件 → 重命名
- 锁定：用于并发访问的基于文件的锁
- 同步：每次更新后将更改推送到 OpenCode 待办事项 API
