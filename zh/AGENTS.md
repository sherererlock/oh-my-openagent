# oh-my-opencode — O P E N C O D E 插件

**生成时间:** 2026-03-06 | **提交:** 7fe44024 | **分支:** dev

## 概述

OpenCode 插件 (npm: `oh-my-opencode`)，它通过多智能体编排、48个生命周期钩子、26个工具、技能/命令/MCP 系统以及 Claude Code 兼容性，扩展了 Claude Code (OpenCode 分支)。包含 1268 个 TypeScript 文件，16 万行代码。

## 结构

```
oh-my-opencode/
├── src/
│   ├── index.ts              # 插件入口: loadConfig → createManagers → createTools → createHooks → createPluginInterface
│   ├── plugin-config.ts      # JSONC 多级配置: 用户 → 项目 → 默认 (Zod v4)
│   ├── agents/               # 11个智能体 (Sisyphus, Hephaestus, Oracle, Librarian, Explore, Atlas, Prometheus, Metis, Momus, Multimodal-Looker, Sisyphus-Junior)
│   ├── hooks/                # 48个生命周期钩子，分布在专用模块和独立文件中
│   ├── tools/                # 26个工具，分布在15个目录中
│   ├── features/             # 19个功能模块 (后台智能体、技能加载器、tmux、MCP-OAuth 等)
│   ├── shared/               # 13个类别的95+个实用工具文件
│   ├── config/               # Zod v4 模式系统 (24个文件)
│   ├── cli/                  # CLI: install, run, doctor, mcp-oauth (Commander.js)
│   ├── mcp/                  # 3个内置远程 MCP (websearch, context7, grep_app)
│   ├── plugin/               # 8个 OpenCode 钩子处理器 + 48个钩子组合
│   └── plugin-handlers/      # 6阶段配置加载管道
├── packages/                 # Monorepo: cli-runner, 12个平台二进制文件
└── local-ignore/             # 仅供开发的测试固定装置
```

## 初始化流程

```
OhMyOpenCodePlugin(ctx)
  ├─→ loadPluginConfig()         # JSONC 解析 → 项目/用户合并 → Zod 验证 → 迁移
  ├─→ createManagers()           # TmuxSessionManager, BackgroundManager, SkillMcpManager, ConfigHandler
  ├─→ createTools()              # SkillContext + AvailableCategories + ToolRegistry (26个工具)
  ├─→ createHooks()              # 3层: 核心(39) + 延续(7) + 技能(2) = 48个钩子
  └─→ createPluginInterface()    # 8个 OpenCode 钩子处理器 → PluginInterface
```

## 8个 OPENCODE 钩子处理器

| 处理器 | 目的 |
|---------|---------|
| `config` | 6阶段: 提供商 → 插件组件 → 智能体 → 工具 → MCP → 命令 |
| `tool` | 26个注册工具 |
| `chat.message` | 首次消息变体，会话设置，关键字检测 |
| `chat.params` | Anthropic 努力水平调整 |
| `chat.headers` | Copilot x-initiator 请求头注入 |
| `event` | 会话生命周期 (创建、删除、空闲、错误) |
| `tool.execute.before` | 工具执行前钩子 (文件保护、标签截断、规则注入) |
| `tool.execute.after` | 工具执行后钩子 (输出截断、元数据存储) |
| `experimental.chat.messages.transform` | 上下文注入，思考块验证 |

## 查找指南

| 任务 | 位置 | 备注 |
|------|----------|-------|
| 添加新智能体 | `src/agents/` + `src/agents/builtin-agents/` | 遵循 createXXXAgent 工厂模式 |
| 添加新钩子 | `src/hooks/{name}/` + 注册在 `src/plugin/hooks/create-*-hooks.ts` | 将事件类型匹配到层级 |
| 添加新工具 | `src/tools/{name}/` + 注册在 `src/plugin/tool-registry.ts` | 遵循 createXXXTool 工厂模式 |
| 添加新功能模块 | `src/features/{name}/` | 独立模块，连接在 plugin/ 中 |
| 添加新 MCP | `src/mcp/` + 注册在 `createBuiltinMcps()` | 仅限远程 HTTP |
| 添加新技能 | `src/features/builtin-skills/skills/` | 实现 BuiltinSkill 接口 |
| 添加新命令 | `src/features/builtin-commands/` | 位于 templates/ 中的模板 |
| 添加新 CLI 命令 | `src/cli/cli-program.ts` | Commander.js 子命令 |
| 添加新诊断检查 | `src/cli/doctor/checks/` | 注册在 checks/index.ts |
| 修改配置模式 | `src/config/schema/` + 更新根模式 | Zod v4, 添加到 OhMyOpenCodeConfigSchema |
| 添加新类别 | `src/tools/delegate-task/constants.ts` | DEFAULT_CATEGORIES + CATEGORY_MODEL_REQUIREMENTS |

## 多级配置

```
项目 (.opencode/oh-my-opencode.jsonc)  →  用户 (~/.config/opencode/oh-my-opencode.jsonc)  →  默认
```

- `agents`, `categories`, `claude_code`: 递归深度合并
- `disabled_*` 数组: 集合并集 (拼接 + 去重)
- 所有其他字段: 覆盖替换基础值
- Zod `safeParse()` 会为省略的字段填充默认值
- `migrateConfigFile()` 自动转换旧键

字段：智能体 (14个可覆盖，每个21个字段)，类别 (8个内置 + 自定义)，disabled_* 数组 (智能体，钩子，mcp，技能，命令，工具)，19个特定功能配置。

## 三层 MCP 系统

| 层级 | 来源 | 机制 |
|------|--------|-----------|
| 内置 | `src/mcp/` | 3个远程 HTTP: websearch (Exa/Tavily), context7, grep_app |
| Claude Code | `.mcp.json` | 通过 claude-code-mcp-loader 进行 `${VAR}` 环境变量扩展 |
| 技能嵌入 | SKILL.md YAML | 由 SkillMcpManager 管理 (stdio + HTTP) |

## 约定

- **运行时**: 仅限 Bun — 绝对不要使用 npm/yarn
- **TypeScript**: 严格模式, ESNext, bundler 模块解析, `bun-types` (绝对不要使用 `@types/node`)
- **测试模式**: Bun 测试 (`bun:test`), 同目录的 `*.test.ts`, given/when/then 风格 (使用 `#given`/`#when`/`#then` 前缀的嵌套 describe)
- **CI 测试分离**: 严重依赖 mock 的测试隔离运行 (单独的 `bun test` 进程)，其余的批量运行
- **工厂模式**: 所有的工具、钩子、智能体都使用 `createXXX()`
- **钩子层级**: 会话 (23) → 工具保护 (12) → 转换 (4) → 延续 (7) → 技能 (2)
- **智能体模式**: `primary` (遵循 UI 模型) vs `subagent` (自身的后备链) vs `all`
- **模型解析**: 4步: 覆盖 → 类别默认 → 提供商后备 → 系统默认
- **配置格式**: 带注释的 JSONC，Zod v4 验证，snake_case 键
- **命名规范**: 所有文件/目录使用 kebab-case
- **模块结构**: index.ts 桶导出 (barrel exports)，没有包罗万象的文件 (禁止 utils.ts, helpers.ts)，200 行代码软限制
- **导入**: 模块内相对路径导入，跨模块通过桶导入 (`import { log } from "./shared"`)
- **无路径别名**: 没有 `@/` — 仅限相对路径导入

## 反模式

- 绝对不要使用 `as any`, `@ts-ignore`, `@ts-expect-error`
- 绝对不要抑制 lint/类型错误
- 除非用户明确要求，否则绝对不要在代码/注释中添加表情符号
- 除非明确要求，否则绝对不要提交 (commit)
- 绝对不要直接运行 `bun publish` — 请使用 GitHub Actions
- 绝对不要在本地修改 `package.json` 版本
- 测试: given/when/then — 绝对不要使用 Arrange-Act-Assert 注释
- 注释: 避免 AI 生成的注释模式 (由 comment-checker 钩子强制执行)
- 绝对不要创建包罗万象的文件 (`utils.ts`, `helpers.ts`, `service.ts`)
- 空的 catch 块 `catch(e) {}` — 始终要处理错误
- 绝对不要在生成的内容中使用破折号 (—)、连接号 (–) 或 AI 填充短语
- index.ts 仅作为入口点 — 绝对不要将业务逻辑堆积在那里

## 命令

```bash
bun test                    # Bun 测试套件
bun run build              # 构建插件 (ESM + 声明文件 + 模式)
bun run build:all          # 构建 + 平台二进制文件
bun run typecheck           # tsc --noEmit
bunx oh-my-opencode install # 交互式设置
bunx oh-my-opencode doctor  # 健康诊断
bunx oh-my-opencode run     # 非交互式会话
```

## CI/CD

| 工作流 | 触发条件 | 目的 |
|----------|---------|---------|
| ci.yml | 推送/PR 到 master/dev | 测试 (分离: mock隔离 + 批量), 类型检查, 构建, 模式自动提交 |
| publish.yml | 手动触发 | 版本提升, npm 发布, 平台二进制文件, GitHub 发布, 合并到 master |
| publish-platform.yml | 由 publish 调用 | 通过 bun compile 生成 12 个平台二进制文件 (darwin/linux/windows) |
| sisyphus-agent.yml | @提及 / 触发 | AI 智能体处理 issues/PRs |
| cla.yml | issue_comment/PR | 贡献者的 CLA 助手 |
| lint-workflows.yml | 推送至 .github/ | 工作流文件上的 actionlint + shellcheck |

## 备注

- 日志记录器写入 `/tmp/oh-my-opencode.log` — 查看那里以进行调试
- 后台任务: 每个模型/提供商 5 个并发 (可配置)
- 插件加载超时: Claude Code 插件为 10 秒
- 模型后备优先级: Claude > OpenAI > Gemini > Copilot > OpenCode Zen > Z.ai > Kimi
- 配置迁移会在旧键 (智能体名称、钩子名称、模型版本) 上自动运行
- 构建: bun build (ESM) + tsc --emitDeclarationOnly，外部依赖: @ast-grep/napi
- 测试设置: 通过 bunfig.toml 预加载 `test-setup.ts`，严重依赖 mock 的测试在 CI 中隔离运行
- 98 个桶导出文件 (index.ts) 建立了模块边界
- 架构规则通过 `.sisyphus/rules/modular-code-enforcement.md` 强制执行
