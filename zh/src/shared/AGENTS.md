# src/shared/ — 13 个分类中的 95+ 个实用工具文件

**生成日期:** 2026-03-06

## 概览

在整个插件中使用的横切（Cross-cutting）实用工具。从 `index.ts` 进行桶导出 (Barrel-exported)。日志记录器将内容写入 `/tmp/oh-my-opencode.log`。

## 分类地图

| 分类 | 文件数 | 关键导出 |
|----------|-------|-------------|
| **模型解析** | 17 | `resolveModel()`、`checkModelAvailability()`、`AGENT_MODEL_REQUIREMENTS` |
| **Tmux 集成** | 11 | `createTmuxSession()`、`spawnPane()`、`closePane()`、服务器健康状态 |
| **配置与路径** | 10 | `resolveOpenCodeConfigDir()`、`getDataPath()`、`parseJSONC()` |
| **会话管理** | 8 | `SessionCursor`、`trackInjectedPath()`、`SessionToolsStore` |
| **Git 工作树** | 7 | `parseGitStatusPorcelain()`、`collectGitDiffStats()`、`formatFileChanges()` |
| **命令执行** | 7 | `executeCommand()`、`executeHookCommand()`、嵌入式命令注册表 |
| **迁移** | 6 | `migrateConfigFile()`、AGENT_NAME_MAP、HOOK_NAME_MAP、MODEL_VERSION_MAP |
| **字符串与工具实用程序** | 6 | `toSnakeCase()`、`normalizeToolName()`、`parseFrontmatter()` |
| **代理配置** | 5 | `getAgentVariant()`、`AGENT_DISPLAY_NAMES`、`AGENT_TOOL_RESTRICTIONS` |
| **OpenCode 集成** | 5 | `injectServerAuth()`、`detectExternalPlugins()`、客户端访问器 |
| **类型助手** | 4 | `deepMerge()`、`DynamicTruncator`、`matchPattern()`、`isRecord()` |
| **其他** | 8 | `log()`、`readFile()`、`extractZip()`、`downloadBinary()`、`findAvailablePort()` |

## 模型解析流水线

```
resolveModel(input)
  1. 覆盖 (Override): UI 选择的模型（仅限主代理）
  2. 分类默认: 来自分类配置
  3. 提供者后备 (Provider fallback): AGENT_MODEL_REQUIREMENTS 链
  4. 系统默认: 最终的后备方案
```

关键文件：`model-resolver.ts`（入口）、`model-resolution-pipeline.ts`（编排）、`model-requirements.ts`（后备链）、`model-availability.ts`（模糊匹配）。

## 迁移系统

在加载时自动转换旧配置：
- `agent-names.ts`: 旧代理名称 → 新代理名称（例如，`junior` → `sisyphus-junior`）
- `hook-names.ts`: 旧钩子名称 → 新钩子名称
- `model-versions.ts`: 旧模型 ID → 当前模型 ID
- `agent-category.ts`: 旧代理配置 → 分类系统

## 最常导入

| 实用工具 | 导入次数 | 用途 |
|---------|-------------|---------|
| `logger.ts` | 62 | `/tmp/oh-my-opencode.log` |
| `data-path.ts` | 11 | XDG 存储解析 |
| `model-requirements.ts` | 11 | 代理后备链 |
| `system-directive.ts` | 11 | 系统消息过滤 |
| `frontmatter.ts` | 10 | YAML 元数据提取 |