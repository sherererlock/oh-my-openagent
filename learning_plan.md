# Oh-My-OpenCode 学习计划与核心概念指南

本指南将帮助你系统地了解和学习 `oh-my-openagent` (Oh-My-OpenCode 插件) 的源码架构和核心机制。

## 1. 核心概念 (Core Concepts)

在深入阅读源码之前，强烈建议先理解以下四大核心概念：

*   **OpenCode 插件机制 (Plugin Interface):**
    本仓库本质上是一个 OpenCode 插件。它通过拦截和处理 OpenCode 的 8 个核心生命周期钩子（如 `config`, `tool`, `chat.message`, `tool.execute.before` 等）来扩展其能力。
*   **多智能体编排 (Multi-Agent Orchestration):**
    系统内置了 11 种不同的 Agent（如 Sisyphus, Hephaestus, Oracle 等）。你需要理解它们是如何被划分角色（主代理 `primary` vs 子代理 `subagent`），以及它们之间如何通过 Fallback 链进行协作的。
*   **多层级配置系统 (Multi-level Config):**
    项目采用 Zod v4 构建了强大的 JSONC 配置系统。配置会按照 `项目级 (.opencode/oh-my-opencode.jsonc)` -> `用户级 (~/.config/opencode/oh-my-opencode.jsonc)` -> `默认配置` 的顺序进行深度合并与回退。
*   **三层 MCP (Model Context Protocol) 架构:**
    系统支持 3 种 MCP 集成方式：
    1. Built-in (内置远程 HTTP MCP，如 websearch)
    2. Claude Code 环境变量注入
    3. Skill-embedded (通过 `SKILL.md` YAML 声明的技能 MCP)

## 2. 源码阅读路径 (Reading Path)

建议按照系统初始化的执行流进行源码阅读：

### 第一步：入口点与初始化流程
*   **文件**: [`src/index.ts`](file:///e:/workspace/GitRepository/oh-my-openagent/src/index.ts)
*   **重点**: 观察 `OhMyOpenCodePlugin` 函数。它是整个插件的入口，清晰地展示了初始化的 5 个核心步骤：
    `loadPluginConfig` -> `createManagers` -> `createTools` -> `createHooks` -> `createPluginInterface`。

### 第二步：配置解析与合并机制
*   **文件**: [`src/plugin-config.ts`](file:///e:/workspace/GitRepository/oh-my-openagent/src/plugin-config.ts) 与 [`src/config/`](file:///e:/workspace/GitRepository/oh-my-openagent/src/config/) 目录
*   **重点**: 了解系统是如何使用 Zod 进行配置校验，以及处理历史配置字段迁移 (`migrateConfigFile`) 的。

### 第三步：插件接口与钩子代理
*   **文件**: [`src/plugin-interface.ts`](file:///e:/workspace/GitRepository/oh-my-openagent/src/plugin-interface.ts) 与 [`src/plugin/`](file:///e:/workspace/GitRepository/oh-my-openagent/src/plugin/) 目录
*   **重点**: 查看 `src/plugin/` 下的文件，理解系统是如何将 OpenCode 的 8 个原生 Handler 映射并分发到内部的 48 个自定义 Lifecycle Hooks 中的。

### 第四步：Agent 与 Tool 的实现
*   **文件**: [`src/agents/`](file:///e:/workspace/GitRepository/oh-my-openagent/src/agents/) 和 [`src/tools/`](file:///e:/workspace/GitRepository/oh-my-openagent/src/tools/)
*   **重点**: 观察**工厂模式 (Factory Pattern)** 的应用。每个 Agent 和 Tool 都通过 `createXXXAgent` 或 `createXXXTool` 模式创建。重点阅读一两个基础 Agent (如 Sisyphus) 的实现逻辑。

### 第五步：特色功能模块与后台管理
*   **文件**: [`src/features/`](file:///e:/workspace/GitRepository/oh-my-openagent/src/features/)
*   **重点**: 了解后台任务管理 (`background-agent`)、MCP与技能管理 (`skill-mcp-manager`) 等独立 Feature 模块是如何挂载到主流程中的。

## 3. 开发与测试规范 (Conventions)

*   **运行时环境**: 仅支持 **Bun**，不使用 npm/yarn/node。
*   **TypeScript 规范**: 严格模式、ESNext、禁止使用 `@ts-ignore` 或 `any`。
*   **代码组织**: 强依赖 `index.ts` 作为模块导出屏障 (Barrel exports)，禁止使用 `utils.ts` 这类大杂烩文件。软限制单文件不超过 200 行。
*   **测试风格**: 使用 `bun:test`，并严格遵循 Given/When/Then (假设/当/那么) 的结构组织测试用例。

---
**下一步**: 
阅读上述计划后，你可以告诉我你想先深入探索哪个部分（例如："帮我详细分析一下 src/index.ts" 或 "我想了解 Agent 的注册和调度逻辑"）。
