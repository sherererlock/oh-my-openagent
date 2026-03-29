# src/features/ — 19 个特性模块

**生成时间:** 2026-03-06

## 概览

独立的特性模块，连接到 `plugin/` 层中。每个模块都是自包含的，拥有自己的类型、实现和测试。

## 模块映射

| 模块 | 文件数 | 复杂度 | 用途 |
|--------|-------|------------|---------|
| **opencode-skill-loader** | 33 | 高 (HIGH) | 从 4 个作用域加载 YAML frontmatter 技能 |
| **background-agent** | 31 | 高 (HIGH) | 任务生命周期、并发（每个模型 5 个）、轮询、生成器 (spawner) 模式 |
| **tmux-subagent** | 30 | 高 (HIGH) | Tmux 面板管理、网格规划、会话编排 |
| **mcp-oauth** | 18 | 高 (HIGH) | 针对 MCP 服务器的 OAuth 2.0 + PKCE + DCR (RFC 7591) |
| **builtin-skills** | 17 | 低 (LOW) | 6 个技能: git-master, playwright, playwright-cli, agent-browser, dev-browser, frontend-ui-ux |
| **skill-mcp-manager** | 12 | 中 (MEDIUM) | 每个会话的 MCP 客户端生命周期 (stdio + HTTP) |
| **claude-code-plugin-loader** | 10 | 中 (MEDIUM) | 从 `.opencode/plugins/` 统一发现插件 |
| **builtin-commands** | 11 | 低 (LOW) | 命令模板: refactor, init-deep, handoff 等 |
| **claude-tasks** | 7 | 中 (MEDIUM) | 任务模式 (schema) + 文件存储 + OpenCode 待办事项同步 |
| **claude-code-mcp-loader** | 6 | 中 (MEDIUM) | 支持 `${VAR}` 环境变量扩展的 `.mcp.json` 加载 |
| **context-injector** | 6 | 中 (MEDIUM) | 将 AGENTS.md/README.md 注入上下文 |
| **run-continuation-state** | 5 | 低 (LOW) | 跨会话的 `run` 命令延续状态持久化 |
| **hook-message-injector** | 5 | 中 (MEDIUM) | 为钩子注入系统消息 |
| **boulder-state** | 5 | 低 (LOW) | 多步操作的持久化状态 |
| **task-toast-manager** | 4 | 中 (MEDIUM) | 任务进度通知 |
| **tool-metadata-store** | 3 | 低 (LOW) | 工具执行元数据缓存 |
| **claude-code-session-state** | 3 | 低 (LOW) | 子智能体会话状态跟踪 |
| **claude-code-command-loader** | 3 | 低 (LOW) | 从 `.opencode/commands/` 加载命令 |
| **claude-code-agent-loader** | 3 | 低 (LOW) | 从 `.opencode/agents/` 加载智能体 |

## 核心模块

### background-agent (31 个文件, ~10k 行代码)

核心编排引擎。`BackgroundManager` 管理任务生命周期：
- 状态：pending (等待) → running (运行中) → completed (完成)/error (错误)/cancelled (取消)/interrupt (中断)
- 并发：通过 `ConcurrencyManager` (FIFO 队列) 限制每个模型/供应商的并发
- 轮询：3 秒间隔，通过空闲事件 + 稳定性检测 (10 秒未改变) 来判定完成
- spawner/：8 个职责单一的文件，通过 `SpawnerContext` 接口进行组合

### opencode-skill-loader (33 个文件, ~3.2k 行代码)

4 个作用域的技能发现（项目 > opencode > 用户 > 全局）：
- 从 SKILL.md 文件中解析 YAML frontmatter
- 带有优先级去重机制的技能合并
- 带有变量替换的模板解析
- 针对特定模型技能的供应商门控机制

### tmux-subagent (30 个文件, ~3.6k 行代码)

状态优先的 tmux 集成：
- `TmuxSessionManager`: 面板生命周期、网格规划
- 生成操作决策器 + 目标查找器
- 用于会话健康状态的轮询管理器
- 用于面板创建/销毁的事件处理器

### builtin-skills (6 个技能对象)

| 技能 | 大小 | MCP | 工具 |
|-------|------|-----|-------|
| git-master | 1111 行 | — | Bash |
| playwright | 312 行 | @playwright/mcp | — |
| agent-browser | (在 playwright.ts 中) | — | Bash(agent-browser:*) |
| playwright-cli | 268 行 | — | Bash(playwright-cli:*) |
| dev-browser | 221 行 | — | Bash |
| frontend-ui-ux | 79 行 | — | — |

浏览器变体由 `browserProvider` 配置选择：playwright (默认) | playwright-cli | agent-browser。
