# src/hooks/rules-injector/ — 条件规则注入

**生成日期:** 2026-03-06

## 概览

19 个文件（约 1604 行代码）。`rulesInjectorHook` — 工具守卫层（Tool Guard Tier）钩子，当读取、写入或编辑目录中的文件时，自动将 `AGENTS.md`（以及类似的规则文件）注入到上下文中。基于距离：距离目标路径最近的规则文件胜出。

## 它的工作原理

```
tool.execute.after (read/write/edit/multiedit)
  → 从工具输出中提取文件路径
  → 查找目标路径附近的规则文件 (finder.ts)
  → 本次会话中是否已经注入过？(cache.ts)
  → 将规则内容注入到工具输出中 (injector.ts)
```

## 追踪的工具

`["read", "write", "edit", "multiedit"]` — 仅在文件操作工具上触发。

## 关键文件

| 文件 | 用途 |
|------|---------|
| `hook.ts` | `createRulesInjectorHook()` — 连接缓存 + 注入器，处理工具事件 |
| `injector.ts` | `createRuleInjectionProcessor()` — 编排 查找 → 缓存 → 注入 流程 |
| `finder.ts` | `findRuleFiles()` + `calculateDistance()` — 定位目标路径附近的 AGENTS.md |
| `rule-file-finder.ts` | 遍历目录树以查找 AGENTS.md / .rules 文件 |
| `rule-file-scanner.ts` | 扫描目录中的规则文件 |
| `matcher.ts` | 将文件路径与规则文件的作用域进行匹配 |
| `rule-distance.ts` | 计算文件和规则文件之间的路径距离 |
| `project-root-finder.ts` | 查找项目根目录（在 .git、package.json 处停止） |
| `output-path.ts` | 从工具输出文本中提取文件路径 |
| `cache.ts` | `createSessionCacheStore()` — 每个会话的注入去重 |
| `storage.ts` | 在多次工具调用之间持久化已注入的路径 |
| `parser.ts` | 解析规则文件内容 |
| `constants.ts` | 规则文件名：`AGENTS.md`、`.rules`、`CLAUDE.md` |
| `types.ts` | `RuleFile`、`InjectionResult`、`RuleFileScope` |

## 规则文件发现

优先级（距离目标文件由近到远）：
1. 与目标文件相同的目录
2. 父目录直至项目根目录
3. 项目根目录本身

距离相同的情况：全部注入。每个会话级别的去重可防止重复注入。

## 截断

使用 `DynamicTruncator` — 根据模型上下文窗口调整注入大小（拥有 1M 上下文的模型获取完整内容，较小的模型获取截断的摘要）。