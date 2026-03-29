# 贡献指南 - Oh My OpenCode

首先，感谢您抽出时间进行贡献！本文档提供了为 oh-my-opencode 做贡献的指南和说明。

## 目录

- [行为准则](#行为准则)
- [入门指南](#入门指南)
  - [先决条件](#先决条件)
  - [开发环境设置](#开发环境设置)
  - [在本地测试您的更改](#在本地测试您的更改)
- [项目结构](#项目结构)
- [开发工作流](#开发工作流)
  - [构建命令](#构建命令)
  - [代码风格与约定](#代码风格与约定)
- [进行更改](#进行更改)
  - [添加新智能体](#添加新智能体)
  - [添加新钩子](#添加新钩子)
  - [添加新工具](#添加新工具)
  - [添加新 MCP 服务器](#添加新-mcp-服务器)
- [拉取请求流程](#拉取请求流程)
- [发布](#发布)
- [获取帮助](#获取帮助)

## 行为准则

请保持尊重、包容和建设性。我们在这里共同打造更好的工具。

## 语言政策

**英语是本仓库所有交流的主要语言。**

这包括：

- Issues 和错误报告
- 拉取请求和代码审查
- 文档和注释
- 讨论和社区互动

### 为什么使用英语？

- **全球可访问性**：英语使来自所有地区的贡献者能够有效协作
- **一致性**：单一语言使讨论保持井然有序且可搜索
- **开源最佳实践**：大多数成功的开源项目都使用英语作为通用语言

### 英语方面需要帮助？

如果英语不是您的第一语言，请不要担心！我们重视您的贡献，而不苛求完美的语法。您可以：

- 使用翻译工具帮助撰写消息
- 向其他社区成员寻求帮助
- 专注于清晰、简单的沟通，而不是完美的散文

## 入门指南

### 先决条件

- **Bun** (最新版本) - 唯一受支持的包管理器
- **TypeScript 5.7.3+** - 用于类型检查和声明
- **OpenCode 1.0.150+** - 用于测试插件

### 开发环境设置

```bash
# 克隆仓库
git clone https://github.com/code-yeongyu/oh-my-openagent.git
cd oh-my-openagent

# 安装依赖 (仅限 bun - 绝对不要使用 npm/yarn)
bun install

# 构建项目
bun run build
```

### 在本地测试您的更改

进行更改后，您可以在 OpenCode 中测试您的本地构建：

1. **构建项目**:

   ```bash
   bun run build
   ```

2. **更新您的 OpenCode 配置** (`~/.config/opencode/opencode.json` 或 `opencode.jsonc`):

   ```json
   {
     "plugin": ["file:///absolute/path/to/oh-my-opencode/dist/index.js"]
   }
   ```

   例如，如果您的项目在 `/Users/yourname/projects/oh-my-opencode`：

   ```json
   {
     "plugin": ["file:///Users/yourname/projects/oh-my-opencode/dist/index.js"]
   }
   ```

   > **注意**: 如果插件数组中存在 `"oh-my-opencode"`，请将其删除，以避免与 npm 版本发生冲突。

3. **重启 OpenCode** 以加载更改。

4. **验证** 插件是否已加载，检查 OmO 智能体是否可用或启动消息。

## 项目结构

```
oh-my-opencode/
├── src/
│   ├── index.ts         # 插件入口 (OhMyOpenCodePlugin)
│   ├── plugin-config.ts # JSONC 多级配置 (Zod v4)
│   ├── agents/          # 11个智能体 (Sisyphus, Hephaestus, Oracle, Librarian, Explore, Atlas, Prometheus, Metis, Momus, Multimodal-Looker, Sisyphus-Junior)
│   ├── hooks/           # 用于编排、恢复、UX和上下文管理的生命周期钩子
│   ├── tools/           # 26个工具，分布在15个目录中
│   ├── mcp/             # 3个内置远程 MCP (websearch, context7, grep_app)
│   ├── features/        # 19个功能模块 (后台智能体、技能加载器、tmux、MCP-OAuth 等)
│   ├── config/          # Zod v4 模式系统
│   ├── shared/          # 跨领域的实用工具
│   ├── cli/             # CLI: install, run, doctor, mcp-oauth (Commander.js)
│   ├── plugin/          # 8个 OpenCode 钩子处理器 + 钩子组合
│   └── plugin-handlers/ # 6阶段配置加载管道
├── packages/            # Monorepo: comment-checker, opencode-sdk
└── dist/                # 构建输出 (ESM + .d.ts)
```

## 开发工作流

### 构建命令

```bash
# 仅类型检查
bun run typecheck

# 完整构建 (ESM + TypeScript 声明 + JSON 模式)
bun run build

# 清理构建输出
bun run clean

# 从头开始重新构建
bun run clean && bun run build

# 仅构建模式 (修改 src/config/schema.ts 后)
bun run build:schema
```

### 代码风格与约定

| 约定       | 规则                                                                      |
| ---------------- | ------------------------------------------------------------------------- |
| 包管理器  | **仅限 Bun** (`bun run`, `bun build`, `bunx`)                             |
| 类型            | 使用 `bun-types`，不要使用 `@types/node`                                        |
| 目录命名 | kebab-case (`ast-grep/`, `claude-code-hooks/`)                            |
| 文件操作  | 绝对不要在代码中使用 bash 命令 (mkdir/touch/rm) 来创建文件        |
| 工具结构   | 每个工具: `index.ts`, `types.ts`, `constants.ts`, `tools.ts`, `utils.ts` |
| 钩子模式     | `createXXXHook(input: PluginInput)` 函数命名                       |
| 导出          | 桶模式 (Barrel pattern) (在 index.ts 中使用 `export * from "./module"`)                   |

**反模式 (不要做)**:

- 使用 npm/yarn 代替 bun
- 使用 `@types/node` 代替 `bun-types`
- 使用 `as any`, `@ts-ignore`, `@ts-expect-error` 抑制 TypeScript 错误
- 通用的 AI 生成的冗长注释
- 直接运行 `bun publish` (仅使用 GitHub Actions)
- 在 `package.json` 中本地修改版本

## 进行更改

### 添加新智能体

1. 在 `src/agents/` 中创建一个新的 `.ts` 文件
2. 按照现有模式定义智能体配置
3. 添加到 `src/agents/index.ts` 中的 `builtinAgents`
4. 如果需要，更新 `src/agents/types.ts`
5. 运行 `bun run build:schema` 以更新 JSON 模式

```typescript
// src/agents/my-agent.ts
import type { AgentConfig } from "./types";

export const myAgent: AgentConfig = {
  name: "my-agent",
  model: "anthropic/claude-opus-4-6",
  description: "此智能体功能的描述",
  prompt: `您的智能体的系统提示词在这里`,
  temperature: 0.1,
  // ... 其他配置
};
```

### 添加新钩子

1. 在 `src/hooks/` 中创建一个新目录 (kebab-case)
2. 实现返回事件处理程序的 `createXXXHook()` 函数
3. 从 `src/hooks/index.ts` 中导出

```typescript
// src/hooks/my-hook/index.ts
import type { PluginInput } from "@opencode-ai/plugin";

export function createMyHook(input: PluginInput) {
  return {
    onSessionStart: async () => {
      // 钩子逻辑在这里
    },
  };
}
```

### 添加新工具

1. 在 `src/tools/` 中创建一个新目录，包含所需文件:
   - `index.ts` - 主要导出
   - `types.ts` - TypeScript 接口
   - `constants.ts` - 常量和工具描述
   - `tools.ts` - 工具实现
   - `utils.ts` - 辅助函数
2. 添加到 `src/tools/index.ts` 中的 `builtinTools`

### 添加新 MCP 服务器

1. 在 `src/mcp/` 中创建配置
2. 添加到 `src/mcp/index.ts`
3. 如果需要外部设置，在 README 中记录

## 拉取请求流程

1. **复刻 (Fork)** 仓库并从 `dev` 创建您的分支
2. **进行更改**，遵循上述约定
3. **在本地构建并测试**:
   ```bash
   bun run typecheck  # 确保没有类型错误
   bun run build      # 确保构建成功
   ```
4. **在 OpenCode 中测试**，使用上面描述的本地构建方法
5. **提交 (Commit)**，附带清晰的、描述性的消息:
   - 使用现在时 ("Add feature" 而不是 "Added feature")
   - 如果适用，引用 issues ("Fix #123")
6. **推送 (Push)** 到您的复刻仓库并创建拉取请求 (Pull Request)
7. 在 PR 描述中清晰地 **描述** 您的更改

### PR 检查清单

- [ ] 代码遵循项目约定
- [ ] `bun run typecheck` 通过
- [ ] `bun run build` 成功
- [ ] 已使用 OpenCode 进行本地测试
- [ ] 如果需要，已更新文档 (README, AGENTS.md)
- [ ] 没有更改 `package.json` 中的版本

## 发布

**重要提示**: 发布完全通过 GitHub Actions 处理。

- **绝对不要** 直接运行 `bun publish` (OIDC 出处问题)
- **绝对不要** 在本地修改 `package.json` 的版本
- 维护者使用 GitHub Actions workflow_dispatch:
  ```bash
  gh workflow run publish -f bump=patch  # 或 minor/major
  ```

## 获取帮助

- **项目知识**: 查看 `AGENTS.md` 以获取详细的项目文档
- **代码模式**: 审查 `src/` 中的现有实现
- **Issues**: 提出一个 issue 来报告错误或请求功能
- **讨论**: 发起一个讨论来提问或分享想法

---

感谢您为 Oh My OpenCode 做出贡献！您的努力有助于让 AI 辅助编程对每个人都变得更好。
