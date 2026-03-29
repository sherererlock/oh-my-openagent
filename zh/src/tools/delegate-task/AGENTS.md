# src/tools/delegate-task/ — 任务委托引擎

**生成日期:** 2026-03-06

## 概览

49 个文件。`task` 工具实现 — 通过后台或同步会话将工作委托给子代理。解析分类、模型、技能，并管理异步和同步执行流。8+ 个内置分类。

## 两种执行模式

| 模式 | 流程 | 用例 |
|------|------|----------|
| **后台** (`run_in_background=true`) | 启动 → BackgroundManager → 轮询 → 通知父级 | 探索 (Explore)，Librarian，并行工作 |
| **同步** (`run_in_background=false`) | 创建会话 → 发送提示词 → 轮询直到空闲 → 返回结果 | 需要立即返回结果的顺序任务 |

## 关键文件

| 文件 | 用途 |
|------|---------|
| `tools.ts` | `createDelegateTask()` 工厂 — 主入口点 |
| `executor.ts` | 路由到后台或同步执行 |
| `types.ts` | `DelegateTaskArgs`、`DelegateTaskToolOptions`、`ToolContextWithMetadata` |
| `category-resolver.ts` | 映射分类名称 → 模型 + 配置 |
| `subagent-resolver.ts` | 映射 subagent_type → 代理 + 模型 |
| `model-selection.ts` | 模型可用性检查 + 后备 |
| `skill-resolver.ts` | 解析 `load_skills[]` → 用于注入的技能内容 |
| `prompt-builder.ts` | 使用技能内容、分类构建系统/用户提示词 |

## 同步执行链

```
sync-task.ts → sync-session-creator.ts → sync-prompt-sender.ts → sync-session-poller.ts → sync-result-fetcher.ts
```

每个文件处理一个步骤。`sync-continuation.ts` 处理会话延续（使用 session_id 恢复）。

## 后台执行

```
background-task.ts → BackgroundManager.launch() → (异步轮询) → background-continuation.ts
```

`background-continuation.ts` 处理现有后台任务的 `session_id` 恢复。

## 分类解析

1. 检查用户定义的分类 (`pluginConfig.categories`)
2. 回退到内置的 8 个分类
3. 从分类配置中解析模型
4. 检查模型可用性 → 如果不可用则回退

## 模型字符串解析器

`model-string-parser.ts` 处理 `"model variant"` 格式（例如，`"gpt-5.3-codex medium"` → model=`gpt-5.3-codex`, variant=`medium`）。

## 不稳定代理跟踪

`unstable-agent-task.ts` 标记来自已知不稳定分类/代理的任务（例如，免费模型）。启用 `unstableAgentBabysitter` 钩子监控。