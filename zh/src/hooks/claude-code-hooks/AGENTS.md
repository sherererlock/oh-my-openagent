# src/hooks/claude-code-hooks/ — Claude Code 兼容性

**生成日期:** 2026-03-06

## 概览

19 个文件，约 2110 行代码。提供 Claude Code `settings.json` 的兼容层。解析 CC（Claude Code）权限规则，并将 CC 钩子（PreToolUse、PostToolUse）映射到 OpenCode 钩子。

## 它的功能

1. 解析 Claude Code `settings.json` 的权限格式
2. 将 CC 钩子类型映射到 OpenCode 事件类型
3. 执行 CC 权限规则（按工具允许/拒绝）
4. 支持 CC 的 `.claude/settings.json` 和 `.claude/settings.local.json`

## CC → OPENCODE 钩子映射

| CC 钩子 | OpenCode 事件 |
|---------|---------------|
| PreToolUse | tool.execute.before |
| PostToolUse | tool.execute.after |
| Notification | event (session.idle) |
| Stop | event (session.idle) |

## 权限系统

CC 权限格式：
```json
{
  "permissions": {
    "allow": ["Edit", "Write"],
    "deny": ["Bash(rm:*)"]
  }
}
```

通过 `shared/` 中的 `permission-compat` 转换为 OpenCode 的工具限制。

## 文件

关键文件：`settings-loader.ts` (解析 CC 设置)、`hook-mapper.ts` (CC 到 OC 的映射)、`permission-handler.ts` (规则执行)、`types.ts` (CC 类型定义)。