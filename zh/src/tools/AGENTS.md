# src/tools/ — 跨 15 个目录的 26 个工具

**生成日期:** 2026-03-06

## 概览

通过 `createToolRegistry()` 注册的 26 个工具。两种模式：19 个工具使用工厂函数（`createXXXTool`），7 个工具（LSP + interactive_bash）直接使用 `ToolDefinition`。

## 工具目录

### 任务管理 (4)

| 工具 | 工厂函数 | 参数 |
|------|---------|------------|
| `task_create` | `createTaskCreateTool` | subject, description, blockedBy, blocks, metadata, parentID |
| `task_list` | `createTaskList` | (无) |
| `task_get` | `createTaskGetTool` | id |
| `task_update` | `createTaskUpdateTool` | id, subject, description, status, addBlocks, addBlockedBy, owner, metadata |

### 委托 (1)

| 工具 | 工厂函数 | 参数 |
|------|---------|------------|
| `task` | `createDelegateTask` | description, prompt, category, subagent_type, run_in_background, session_id, load_skills, command |

**8 个内置分类**: visual-engineering, ultrabrain, deep, artistry, quick, unspecified-low, unspecified-high, writing

### 代理调用 (1)

| 工具 | 工厂函数 | 参数 |
|------|---------|------------|
| `call_omo_agent` | `createCallOmoAgent` | description, prompt, subagent_type, run_in_background, session_id |

### 后台任务 (2)

| 工具 | 工厂函数 | 参数 |
|------|---------|------------|
| `background_output` | `createBackgroundOutput` | task_id, block, timeout, full_session, include_thinking, message_limit, since_message_id, thinking_max_chars |
| `background_cancel` | `createBackgroundCancel` | taskId, all |

### LSP 重构 (6) — 直接使用 ToolDefinition

| 工具 | 参数 |
|------|------------|
| `lsp_goto_definition` | filePath, line, character |
| `lsp_find_references` | filePath, line, character, includeDeclaration |
| `lsp_symbols` | filePath, scope (document/workspace), query, limit |
| `lsp_diagnostics` | filePath, severity |
| `lsp_prepare_rename` | filePath, line, character |
| `lsp_rename` | filePath, line, character, newName |

### 代码搜索 (4)

| 工具 | 工厂函数 | 参数 |
|------|---------|------------|
| `ast_grep_search` | `createAstGrepTools` | pattern, lang, paths, globs, context |
| `ast_grep_replace` | `createAstGrepTools` | pattern, rewrite, lang, paths, globs, dryRun |
| `grep` | `createGrepTools` | pattern, path, include (60 秒超时，10MB 限制) |
| `glob` | `createGlobTools` | pattern, path (60 秒超时，100 个文件限制) |

### 会话历史 (4)

| 工具 | 工厂函数 | 参数 |
|------|---------|------------|
| `session_list` | `createSessionManagerTools` | (无) |
| `session_read` | `createSessionManagerTools` | session_id, include_todos, limit |
| `session_search` | `createSessionManagerTools` | query, session_id, case_sensitive, limit |
| `session_info` | `createSessionManagerTools` | session_id |

### 技能/命令 (2)

| 工具 | 工厂函数 | 参数 |
|------|---------|------------|
| `skill` | `createSkillTool` | name, user_message |
| `skill_mcp` | `createSkillMcpTool` | mcp_name, tool_name/resource_name/prompt_name, arguments, grep |

### 系统 (2)

| 工具 | 工厂函数 | 参数 |
|------|---------|------------|
| `interactive_bash` | 直接定义 | tmux_command |
| `look_at` | `createLookAt` | file_path, image_data, goal |

### 编辑 (1) — 条件性

| 工具 | 工厂函数 | 参数 |
|------|---------|------------|
| `hashline_edit` | `createHashlineEditTool` | file, edits[] |

## 委托分类

| 分类 | 模型 | 领域 |
|----------|-------|--------|
| visual-engineering | gemini-3.1-pro high | 前端，UI/UX |
| ultrabrain | gpt-5.4 xhigh | 困难逻辑 |
| deep | gpt-5.3-codex medium | 自主问题解决 |
| artistry | gemini-3.1-pro high | 创造性方法 |
| quick | gpt-5.4-mini | 琐碎任务 |
| unspecified-low | claude-sonnet-4-6 | 中等努力程度 |
| unspecified-high | claude-opus-4-6 max | 高努力程度 |
| writing | kimi-k2p5 | 文档 |

## 如何添加工具

1. 创建 `src/tools/{name}/index.ts` 并导出工厂函数
2. 创建 `src/tools/{name}/types.ts` 以定义参数 Schema
3. 创建 `src/tools/{name}/tools.ts` 用于实现
4. 在 `src/plugin/tool-registry.ts` 中注册