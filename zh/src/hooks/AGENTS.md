# src/hooks/ — 48 个生命周期钩子

**生成日期：** 2026-03-06

## 概览

横跨专用模块和独立文件的 48 个钩子。三层组合：核心 (Core, 39) + 延续 (Continuation, 7) + 技能 (Skill, 2)。所有钩子都遵循 `createXXXHook(deps) → HookFunction` 工厂模式。

## 钩子层级

### 第 1 层: 会话钩子 (23) — `create-session-hooks.ts`
## 结构
```
hooks/
├── atlas/                      # 主编排器 (757 行)
├── anthropic-context-window-limit-recovery/ # 自动总结
├── anthropic-effort/            # 推理工作量级别调整
├── anthropic-image-context/     # Anthropic 的图像上下文处理
├── auto-slash-command/         # 检测 /command 模式
├── auto-update-checker/        # 插件更新检查
├── background-notification/    # 操作系统通知
├── beast-mode-system/          # 猛兽模式系统提示注入
├── category-skill-reminder/    # 提醒类别技能
├── claude-code-hooks/          # settings.json 兼容层
├── comment-checker/            # 防止 AI 生成废话
├── compaction-context-injector/ # 压缩时注入上下文
├── compaction-todo-preserver/  # 压缩时保留待办事项
├── delegate-task-retry/        # 重试失败的委托
├── directory-agents-injector/  # 自动注入 AGENTS.md
├── directory-readme-injector/  # 自动注入 README.md
├── edit-error-recovery/        # 从失败中恢复
├── hashline-edit-diff-enhancer/ # 针对哈希行编辑的增强差异输出
├── hashline-read-enhancer/     # 为 Read 输出添加 LINE#ID 哈希
├── interactive-bash-session/   # Tmux 会话管理
├── json-error-recovery/        # JSON 解析错误纠正
├── keyword-detector/           # ultrawork/search/analyze 模式
├── model-fallback/             # 供应商级别的模型回退
├── no-hephaestus-non-gpt/      # 阻止 Hephaestus 使用非 GPT 模型
├── no-sisyphus-gpt/            # 阻止 Sisyphus 使用 GPT 模型
├── non-interactive-env/        # 非 TTY 环境处理
├── prometheus-md-only/         # Planner 只读模式
├── question-label-truncator/   # 自动截断问题标签
├── ralph-loop/                 # 自我参照的开发循环
├── read-image-resizer/         # 调整图像大小以提高上下文效率
├── rules-injector/             # 条件规则
├── runtime-fallback/           # API 错误时自动切换模型
├── session-recovery/           # 从崩溃中自动恢复
├── sisyphus-junior-notepad/    # Sisyphus Junior 记事本
├── start-work/                 # Sisyphus 工作会话启动器
├── stop-continuation-guard/    # 保护停止延续
├── task-reminder/              # 任务系统使用提醒
├── task-resume-info/           # 已取消任务的恢复信息
├── tasks-todowrite-disabler/   # 当任务系统激活时禁用 TodoWrite
├── think-mode/                 # 动态思考预算
├── thinking-block-validator/   # 确保有效的 <thinking>
├── todo-continuation-enforcer/ # 强制完成 TODO
├── unstable-agent-babysitter/  # 监控不稳定的代理行为
├── write-existing-file-guard/  # 写入前需要读取
└── index.ts                    # 钩子聚合 + 注册
```

| 钩子 | 事件 | 目的 |
|------|-------|---------|
| contextWindowMonitor | session.idle | 跟踪上下文窗口使用情况 |
| preemptiveCompaction | session.idle | 在达到限制前触发压缩 |
| sessionRecovery | session.error | 可恢复错误自动重试 |
| sessionNotification | session.idle | 完成时的操作系统通知 |
| thinkMode | chat.params | 模型变体切换 (扩展思考) |
| anthropicContextWindowLimitRecovery | session.error | 多策略上下文恢复 (截断, 压缩) |
| autoUpdateChecker | session.created | 检查 npm 上的插件更新 |
| agentUsageReminder | chat.message | 提醒可用的代理 |
| nonInteractiveEnv | chat.message | 调整 `run` 命令的行为 |
| interactiveBashSession | tool.execute | 交互式工具的 Tmux 会话 |
| ralphLoop | event | 自我参照的开发循环 (巨石延续) |
| editErrorRecovery | tool.execute.after | 重试失败的文件编辑 |
| delegateTaskRetry | tool.execute.after | 重试失败的任务委托 |
| startWork | chat.message | `/start-work` 命令处理器 |
| prometheusMdOnly | tool.execute.before | 强制 Prometheus 仅写入 .md |
| sisyphusJuniorNotepad | chat.message | 为子代理注入记事本 |
| questionLabelTruncator | tool.execute.before | 截断过长的问题标签 |
| taskResumeInfo | chat.message | 在恢复时注入任务上下文 |
| anthropicEffort | chat.params | 调整推理工作量级别 |
| modelFallback | chat.params | 错误时的供应商级别模型回退 |
| noSisyphusGpt | chat.message | 阻止 Sisyphus 使用 GPT 模型 (toast 警告) |
| noHephaestusNonGpt | chat.message | 阻止 Hephaestus 使用非 GPT 模型 |
| runtimeFallback | event | API 供应商错误时自动切换模型 |

### 第 2 层: 工具保护钩子 (12) — `create-tool-guard-hooks.ts`

| 钩子 | 事件 | 目的 |
|------|-------|---------|
| commentChecker | tool.execute.after | 阻止 AI 生成的注释模式 |
| toolOutputTruncator | tool.execute.after | 按令牌数量截断过大的工具输出 |
| directoryAgentsInjector | tool.execute.before | 将目录 AGENTS.md 注入上下文 |
| directoryReadmeInjector | tool.execute.before | 将目录 README.md 注入上下文 |
| emptyTaskResponseDetector | tool.execute.after | 检测空任务响应 |
| rulesInjector | tool.execute.before | 条件规则注入 (AGENTS.md, config) |
| tasksTodowriteDisabler | tool.execute.before | 任务系统激活时禁用 TodoWrite |
| writeExistingFileGuard | tool.execute.before | 写入现有文件前需要先执行 Read |
| hashlineReadEnhancer | tool.execute.after | 用行哈希增强 Read 输出 |
| jsonErrorRecovery | tool.execute.after | 检测 JSON 解析错误，注入纠正提醒 |

### 第 3 层: 转换钩子 (4) — `create-transform-hooks.ts`

| 钩子 | 事件 | 目的 |
|------|-------|---------|
| claudeCodeHooks | messages.transform | Claude Code settings.json 兼容性 |
| keywordDetector | messages.transform | 检测 ultrawork/search/analyze 模式 |
| contextInjectorMessagesTransform | messages.transform | 将 AGENTS.md/README.md 注入上下文 |
| thinkingBlockValidator | messages.transform | 验证思考块结构 |

### 第 4 层: 延续钩子 (7) — `create-continuation-hooks.ts`

| 钩子 | 事件 | 目的 |
|------|-------|---------|
| stopContinuationGuard | chat.message | `/stop-continuation` 命令处理器 |
| compactionContextInjector | session.compacted | 压缩后重新注入上下文 |
| compactionTodoPreserver | session.compacted | 在压缩过程中保留待办事项 |
| todoContinuationEnforcer | session.idle | **巨石 (Boulder)**: 当存在未完成的 todo 时强制延续 |
| unstableAgentBabysitter | session.idle | 监控不稳定的代理行为 |
| backgroundNotificationHook | event | 后台任务完成通知 |
| atlasHook | event | 用于巨石/后台会话的主编排器 |

### 第 5 层: 技能钩子 (2) — `create-skill-hooks.ts`

| 钩子 | 事件 | 目的 |
|------|-------|---------|
| categorySkillReminder | chat.message | 提醒类别+技能委托 |
| autoSlashCommand | chat.message | 自动检测用户输入中的 `/command` |

## 关键钩子 (复杂)

### anthropic-context-window-limit-recovery (31 个文件, 约 2232 行代码)
达到上下文限制时的多策略恢复。策略：截断、压缩、总结。

### atlas (17 个文件, 约 1976 行代码)
巨石会话的主编排器。决策门：会话类型 → 中止检查 → 失败计数 → 后台任务 → 代理匹配 → 计划完整性 → 冷却 (5秒)。在 session.idle 时注入延续提示。

### ralph-loop (14 个文件, 约 1687 行代码)
通过 `/ralph-loop` 命令进行自我参照的开发循环。状态持久化在 `.sisyphus/ralph-loop.local.md`。检测 AI 输出中的 `<promise>DONE</promise>`。默认最大迭代次数为 100。

### todo-continuation-enforcer (13 个文件, 约 2061 行代码)
"巨石" (Boulder) 机制。当 todo 未完成时强制代理继续。2 秒倒计时 toast → 注入延续。指数退避：基础 30 秒，每次失败 ×2，连续失败最多 5 次然后暂停 5 分钟。

### keyword-detector (约 1665 行代码)
从用户输入中检测模式：ultrawork, search, analyze, prove-yourself。注入特定模式的系统提示。

### rules-injector (19 个文件, 约 1604 行代码)
从 AGENTS.md、config、技能规则进行条件规则注入。评估条件以确定应用哪些规则。

## 独立钩子 (在 src/hooks/ 根目录)

| 文件 | 目的 |
|------|---------|
| context-window-monitor.ts | 跟踪上下文窗口百分比 |
| preemptive-compaction.ts | 在达到硬性限制之前触发压缩 |
| tool-output-truncator.ts | 按令牌数量截断工具输出 |
| session-notification.ts + 4 helpers | 会话完成时的操作系统通知 |
| empty-task-response-detector.ts | 检测空/失败的任务响应 |
| session-todo-status.ts | Todo 完成状态跟踪 |

## 如何添加一个钩子

1. 使用 `createXXXHook(deps)` 工厂创建 `src/hooks/{name}/index.ts`
2. 在适当的层级文件 (`src/plugin/hooks/create-{tier}-hooks.ts`) 中注册
3. 将钩子名称添加到 `src/config/schema/hooks.ts` HookNameSchema
4. 钩子接收 `(event, ctx)` — 返回值取决于事件类型
