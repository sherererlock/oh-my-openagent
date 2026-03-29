# src/cli/ — CLI 命令行工具: install, run, doctor, mcp-oauth

**生成时间:** 2026-03-06

## 概览

基于 Commander.js 的 CLI，包含 5 个命令。入口：`index.ts` → `cli-program.ts` 中的 `runCli()`。

## 命令

| 命令 | 用途 | 核心逻辑 |
|---------|---------|-----------|
| `install` | 交互式/非交互式安装设置 | 供应商选择 → 配置生成 → 插件注册 |
| `run <message>` | 非交互式会话启动器 | 智能体解析 (标志位 → 环境变量 → 配置 → Sisyphus) |
| `doctor` | 4 大类健康检查 | 系统 (System)、配置 (Config)、工具 (Tools)、模型 (Models) |
| `get-local-version` | 版本检测 | 已安装版本 vs npm 最新版本 |
| `mcp-oauth` | OAuth 令牌管理 | 登录 (login, PKCE)、登出 (logout)、状态查询 (status) |

## 目录结构

```
cli/
├── index.ts                     # 入口点 → runCli()
├── cli-program.ts               # Commander.js 程序定义 (5 个命令)
├── install.ts                   # 路由至 TUI 或 CLI 安装器
├── cli-installer.ts             # 非交互式 (控制台输出)
├── tui-installer.ts             # 交互式 (@clack/prompts)
├── model-fallback.ts            # 基于供应商可用性生成模型配置
├── provider-availability.ts     # 供应商检测
├── fallback-chain-resolution.ts # 降级链路逻辑
├── config-manager/              # 20 个配置实用工具
│   ├── 插件注册，供应商配置
│   ├── JSONC 操作，认证插件
│   └── npm dist-tags，二进制检测
├── doctor/
│   ├── runner.ts                # 并行执行检查
│   ├── formatter.ts             # 输出格式化
│   └── checks/                  # 15 个检查文件，分为 4 大类
│       ├── system.ts            # 二进制文件、插件、版本
│       ├── config.ts            # JSONC 有效性、Zod 模式验证
│       ├── tools.ts             # AST-Grep、LSP、GH CLI、MCP
│       └── model-resolution.ts  # 缓存、解析、覆盖配置 (6 个子文件)
├── run/                         # 会话启动器
│   ├── runner.ts                # 主编排
│   ├── agent-resolver.ts        # 标志位 → 环境变量 → 配置 → Sisyphus
│   ├── session-resolver.ts      # 创建/恢复会话
│   ├── event-handlers.ts        # 事件处理
│   └── poll-for-completion.ts   # 等待待办事项/后台任务完成
└── mcp-oauth/                   # OAuth 令牌管理
```

## 模型降级系统

没有单一的全局优先级。CLI 安装时的解析使用 `model-fallback-requirements.ts` 中按智能体划分的降级链路。

常见模式：当智能体链路包含 Claude/OpenAI/Gemini 时优先使用它们；`librarian` 偏好 ZAI；`sisyphus` 通过 Kimi 降级到 GLM-5；`hephaestus` 需要兼容 OpenAI 的供应商。

## DOCTOR 检查项

| 类别 | 验证内容 |
|----------|-----------|
| **System (系统)** | 二进制文件存在、版本 >=1.0.150、插件已注册、版本匹配 |
| **Config (配置)** | JSONC 有效性、Zod 模式验证、模型覆盖配置语法 |
| **Tools (工具)** | AST-Grep、comment-checker、LSP 服务器、GH CLI、MCP 服务器 |
| **Models (模型)** | 缓存存在、模型解析、智能体/分类覆盖配置、可用性 |

## 如何添加新的 DOCTOR 检查项

1. 创建 `src/cli/doctor/checks/{name}.ts`
2. 导出符合 `DoctorCheck` 接口的检查函数
3. 在 `checks/index.ts` 中进行注册
