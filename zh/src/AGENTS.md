# src/ — 插件源码

**生成时间:** 2026-03-06

## 概览

入口点 `index.ts` 编排了 5 步初始化过程：loadConfig（加载配置） → createManagers（创建管理器） → createTools（创建工具） → createHooks（创建钩子） → createPluginInterface（创建插件接口）。

## 核心文件

| 文件 | 用途 |
|------|---------|
| `index.ts` | 插件入口，导出 `OhMyOpenCodePlugin` |
| `plugin-config.ts` | JSONC 解析、多级合并、Zod v4 校验 |
| `create-managers.ts` | TmuxSessionManager（Tmux会话管理器）、BackgroundManager（后台管理器）、SkillMcpManager（技能MCP管理器）、ConfigHandler（配置处理器） |
| `create-tools.ts` | SkillContext（技能上下文） + AvailableCategories（可用分类） + ToolRegistry（包含26个工具的注册表） |
| `create-hooks.ts` | 三层架构：Core(39) + Continuation(7) + Skill(2) = 48 个钩子 |
| `plugin-interface.ts` | 8 个 OpenCode 钩子处理器：config、tool、chat.message、chat.params、chat.headers、event、tool.execute.before、tool.execute.after |

## 配置加载

```
loadPluginConfig(directory, ctx)
  1. 用户级: ~/.config/opencode/oh-my-opencode.jsonc
  2. 项目级: .opencode/oh-my-opencode.jsonc
  3. mergeConfigs(user, project) → 对 agents/categories 进行深度合并 (deepMerge)，对 disabled_* 数组进行集合并集 (Set union) 操作
  4. Zod safeParse → 为省略的字段填充默认值
  5. migrateConfigFile() → 旧键名自动转换
```

## 钩子组合

```
createHooks()
  ├─→ createCoreHooks()           # 39 个钩子
  │   ├─ createSessionHooks()     # 23个: contextWindowMonitor, thinkMode, ralphLoop, modelFallback, runtimeFallback, noSisyphusGpt, noHephaestusNonGpt, anthropicEffort, intentGate...
  │   ├─ createToolGuardHooks()   # 12个: commentChecker, rulesInjector, writeExistingFileGuard, jsonErrorRecovery, hashlineReadEnhancer...
  │   └─ createTransformHooks()   # 4个: claudeCodeHooks, keywordDetector, contextInjector, thinkingBlockValidator
  ├─→ createContinuationHooks()   # 7个: todoContinuationEnforcer, atlas, stopContinuationGuard, compactionContextInjector...
  └─→ createSkillHooks()          # 2个: categorySkillReminder, autoSlashCommand
```
