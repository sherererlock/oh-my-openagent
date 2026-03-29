# src/agents/ — 11 个智能体定义

**生成时间:** 2026-03-06

## 概览

智能体工厂遵循 `createXXXAgent(model) → AgentConfig` 模式。每个工厂具有静态的 `mode` 属性。通过 `buildAgent()` 组合工厂、分类和技能来构建。

## 智能体清单

| 智能体 | 模型 | 温度 (Temp) | 模式 (Mode) | 降级链路 (Fallback Chain) | 用途 |
|-------|-------|------|------|----------------|---------|
| **Sisyphus** | claude-opus-4-6 max | 0.1 | all | k2p5 → kimi-k2.5 → gpt-5.4 medium → glm-5 → big-pickle | 主编排器，负责规划和委派 |
| **Hephaestus** | gpt-5.4 medium | 0.1 | all | — | 自主的深度工作者 |
| **Oracle** | gpt-5.4 high | 0.1 | subagent | gemini-3.1-pro high → claude-opus-4-6 max | 只读顾问 |
| **Librarian** | minimax-m2.7 | 0.1 | subagent | minimax-m2.7-highspeed → claude-haiku-4-5 → gpt-5-nano | 外部文档/代码搜索 |
| **Explore** | grok-code-fast-1 | 0.1 | subagent | minimax-m2.7-highspeed → minimax-m2.7 → claude-haiku-4-5 → gpt-5-nano | 上下文 grep 搜索 |
| **Multimodal-Looker** | gpt-5.3-codex medium | 0.1 | subagent | k2p5 → gemini-3-flash → glm-4.6v → gpt-5-nano | PDF/图像分析 |
| **Metis** | claude-opus-4-6 max | **0.3** | subagent | gpt-5.4 high → gemini-3.1-pro high | 规划前顾问 |
| **Momus** | gpt-5.4 xhigh | 0.1 | subagent | claude-opus-4-6 max → gemini-3.1-pro high | 计划审查员 |
| **Atlas** | claude-sonnet-4-6 | 0.1 | primary | gpt-5.4 medium | 待办事项编排器 |
| **Prometheus** | claude-opus-4-6 max | 0.1 | — | gpt-5.4 high → gemini-3.1-pro | 战略规划师（内部） |
| **Sisyphus-Junior** | claude-sonnet-4-6 | 0.1 | all | 用户可配置 | 基于分类生成的执行者 |

## 工具限制

| 智能体 | 禁用的工具 |
|-------|-------------|
| Oracle | write, edit, task, call_omo_agent |
| Librarian | write, edit, task, call_omo_agent |
| Explore | write, edit, task, call_omo_agent |
| Multimodal-Looker | 除了 read 以外的所有工具 |
| Atlas | task, call_omo_agent |
| Momus | write, edit, task |

## 目录结构

```
agents/
├── sisyphus.ts            # 559 行，主编排器
├── hephaestus.ts          # 507 行，自主工作者
├── oracle.ts              # 只读顾问
├── librarian.ts           # 外部搜索
├── explore.ts             # 代码库 grep
├── multimodal-looker.ts   # 视觉/PDF
├── metis.ts               # 规划前
├── momus.ts               # 计划审查
├── atlas/agent.ts         # 待办事项编排器
├── types.ts               # AgentFactory, AgentMode
├── agent-builder.ts       # buildAgent() 组合器
├── utils.ts               # 智能体工具类
├── builtin-agents.ts      # createBuiltinAgents() 注册表
└── builtin-agents/        # maybeCreateXXXConfig 条件工厂
    ├── sisyphus-agent.ts
    ├── hephaestus-agent.ts
    ├── atlas-agent.ts
    ├── general-agents.ts  # collectPendingBuiltinAgents
    └── available-skills.ts
```

## 工厂模式

```typescript
const createXXXAgent: AgentFactory = (model: string) => ({
  instructions: "...",
  model,
  temperature: 0.1,
  // ...config
})
createXXXAgent.mode = "subagent" // 或 "primary" 或 "all"
```

模型解析：4步过程：覆盖配置 (override) → 分类默认 (category-default) → 供应商降级 (provider-fallback) → 系统默认 (system-default)。定义在 `shared/model-requirements.ts` 中。

## 模式 (MODES)

- **primary**: 遵循 UI 选择的模型，使用降级链路
- **subagent**: 使用自身的降级链路，忽略 UI 选择
- **all**: 在两种上下文中均可用 (Sisyphus-Junior)
```