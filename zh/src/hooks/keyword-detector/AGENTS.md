# src/hooks/keyword-detector/ — 模式关键字注入

**生成日期:** 2026-03-06

## 概览

8 个文件 + 3 个模式子目录（约 1665 行代码）。位于 `messages.transform` 上的转换层（Transform Tier）钩子。扫描用户的首条消息以查找模式关键字（ultrawork、search、analyze），并注入特定于模式的系统提示词。

## 关键字

| 关键字 | 模式 | 效果 |
|---------|---------|--------|
| `ultrawork` / `ulw` | `/\b(ultrawork|ulw)\b/i` | 全面编排模式 — 并行代理、深度探索、不懈执行 |
| 搜索模式 (Search) | `SEARCH_PATTERN` (来自 `search/`) | 网络/文档搜索焦点提示词注入 |
| 分析模式 (Analyze) | `ANALYZE_PATTERN` (来自 `analyze/`) | 深度分析模式提示词注入 |

## 结构

```
keyword-detector/
├── index.ts           # 桶导出 (Barrel export)
├── hook.ts            # createKeywordDetectorHook() — chat.message 处理器
├── detector.ts        # detectKeywordsWithType() + extractPromptText()
├── constants.ts       # KEYWORD_DETECTORS 数组，从子模块重新导出
├── types.ts           # KeywordDetector, DetectedKeyword 类型
├── ultrawork/
│   ├── index.ts
│   ├── message.ts     # getUltraworkMessage() — 按代理/模型动态生成提示词
│   └── isPlannerAgent.ts
├── search/
│   ├── index.ts
│   ├── pattern.ts     # SEARCH_PATTERN 正则表达式
│   └── message.ts     # SEARCH_MESSAGE
└── analyze/
    ├── index.ts
    ├── pattern.ts     # ANALYZE_PATTERN 正则表达式
    └── message.ts     # ANALYZE_MESSAGE
```

## 检测逻辑

```
chat.message (用户输入)
  → extractPromptText(parts)
  → isSystemDirective? → 跳过
  → removeSystemReminders(text)  # 剥离 <SYSTEM_REMINDER> 块
  → detectKeywordsWithType(cleanText, agentName, modelID)
  → isPlannerAgent(agentName)? → 过滤掉 ultrawork
  → 对于每个检测到的关键字: 将模式消息注入输出
```

## 守卫 (Guards)

- **系统指令跳过**: 标记为系统指令的消息不会被扫描（防止无限循环）
- **规划代理过滤**: Prometheus/plan 代理不会接收 `ultrawork` 注入
- **会话代理跟踪**: 使用 `getSessionAgent()` 获取实际的代理（而不仅仅是输入提示）
- **感知模型的消息**: `getUltraworkMessage(agentName, modelID)` 会根据活动模型调整消息