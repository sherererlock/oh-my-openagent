# src/features/opencode-skill-loader/ — 4 个作用域的技能发现

**生成日期：** 2026-03-06

## 概览

28 个文件（约 3.2k 行代码）。发现、解析、合并和解析来自 4 个作用域的 SKILL.md 文件，并具有优先级去重功能。

## 4 个作用域优先级 (最高 → 最低)

```
1. 项目 (.opencode/skills/)
2. OpenCode 配置 (~/.config/opencode/skills/)
3. 用户 (~/.config/opencode/oh-my-opencode/skills/)
4. 全局 (内置技能)
```

在更高作用域中的同名技能会覆盖较低作用域的技能。

## 关键文件

| 文件 | 目的 |
|------|---------|
| `loader.ts` | 主要的 `loadSkills()` — 协调发现 → 解析 → 合并 |
| `async-loader.ts` | 用于非阻塞技能加载的异步变体 |
| `blocking.ts` | 用于初始加载的同步变体 |
| `merger.ts` | 跨作用域的基于优先级的去重 |
| `skill-content.ts` | 从 SKILL.md 解析 YAML 前言 (frontmatter) |
| `skill-discovery.ts` | 在目录树中查找 SKILL.md 文件 |
| `skill-directory-loader.ts` | 从单个目录加载所有技能 |
| `config-source-discovery.ts` | 从配置中发现作用域目录 |
| `skill-template-resolver.ts` | 技能模板中的变量替换 |
| `skill-mcp-config.ts` | 从技能 YAML 中提取 MCP 配置 |
| `types.ts` | `LoadedSkill`, `SkillScope`, `SkillDiscoveryResult` |

## 技能格式 (SKILL.md)

```markdown
---
name: my-skill
description: 此技能的作用
tools: [Bash, Read, Write]
mcp:
  - name: my-mcp
    type: stdio
    command: npx
    args: [-y, my-mcp-server]
---

技能内容（代理的指令）...
```

## 合并子目录 (MERGER SUBDIRECTORY)

处理当来自多个作用域的技能具有重叠名称或 MCP 配置时的复杂合并逻辑。

## 模板解析 (TEMPLATE RESOLUTION)

技能内容中类似 `{{directory}}`、`{{agent}}` 的变量会在加载时基于当前上下文进行解析。
