# src/config/ — Zod v4 模式 (Schema) 系统

**生成时间:** 2026-03-06

## 概览

由 24 个模式文件组合成 `OhMyOpenCodeConfigSchema`。使用 `safeParse()` 进行 Zod v4 验证。所有字段均为可选 — 省略的字段将使用插件默认值。

## 模式树 (SCHEMA TREE)

```
config/schema/
├── oh-my-opencode-config.ts    # 根模式: OhMyOpenCodeConfigSchema (组合了以下所有模式)
├── agent-names.ts              # BuiltinAgentNameSchema (11个), OverridableAgentNameSchema (14个)
├── agent-overrides.ts          # AgentOverrideConfigSchema (每个智能体 21 个字段)
├── categories.ts               # 8 个内置分类 + 自定义分类
├── hooks.ts                    # HookNameSchema (48 个钩子)
├── skills.ts                   # SkillsConfigSchema (来源、路径、递归)
├── commands.ts                 # BuiltinCommandNameSchema
├── experimental.ts             # 实验性功能开关 (plugin_load_timeout_ms 最小 1000)
├── sisyphus.ts                 # SisyphusConfigSchema (任务系统)
├── sisyphus-agent.ts           # SisyphusAgentConfigSchema
├── ralph-loop.ts               # RalphLoopConfigSchema
├── tmux.ts                     # TmuxConfigSchema + TmuxLayoutSchema
├── websearch.ts                # provider: "exa" | "tavily"
├── claude-code.ts              # CC (Claude Code) 兼容性设置
├── comment-checker.ts          # AI 注释检测配置
├── notification.ts             # 操作系统通知设置
├── git-master.ts               # commit_footer: boolean | string
├── browser-automation.ts       # provider: playwright | agent-browser | playwright-cli
├── background-task.ts          # 每个模型/供应商的并发限制
├── fallback-models.ts          # FallbackModelsConfigSchema
├── runtime-fallback.ts         # RuntimeFallbackConfigSchema
├── babysitting.ts              # 不稳定智能体监控
├── dynamic-context-pruning.ts  # 动态上下文裁剪设置
├── start-work.ts              # StartWorkConfigSchema (自动提交等)
└── internal/permission.ts      # AgentPermissionSchema

```

## 根模式字段 (28个)

`$schema`, `new_task_system_enabled`, `default_run_agent`, `disabled_mcps`, `disabled_agents`, `disabled_skills`, `disabled_hooks`, `disabled_commands`, `disabled_tools`, `hashline_edit`, `agents`, `categories`, `claude_code`, `sisyphus_agent`, `comment_checker`, `experimental`, `auto_update`, `skills`, `ralph_loop`, `background_task`, `notification`, `babysitting`, `git_master`, `browser_automation_engine`, `websearch`, `tmux`, `sisyphus`, `start_work`, `_migrations`

## 智能体覆盖字段 (21个)

`model`, `variant`, `category`, `skills`, `temperature`, `top_p`, `prompt`, `prompt_append`, `tools`, `disable`, `description`, `mode`, `color`, `permission`, `maxTokens`, `thinking`, `reasoningEffort`, `textVerbosity`, `providerOptions`

## 如何添加配置

1. 创建带有 Zod 模式的 `src/config/schema/{name}.ts`
2. 将字段添加到 `oh-my-opencode-config.ts` 的根模式中
3. 通过 `z.infer<typeof YourSchema>` 引用以获取 TypeScript 类型
4. 在处理器中通过 `pluginConfig.{name}` 进行访问
