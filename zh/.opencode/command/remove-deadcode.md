---
description: 移除本项目中未使用的代码，具有超级工作模式、LSP 验证的安全性、原子提交
---

<command-instruction>

通过大规模并行的深度智能体进行死代码移除。你是协调者 — 你扫描、验证、批处理，然后将所有的移除工作委托给并行的智能体。

<rules>
- **LSP 即是法则。** 在做任何移除决定之前，必须通过 `LspFindReferences(includeDeclaration=false)` 进行验证。
- **绝不移除入口点。** `src/index.ts`, `src/cli/index.ts`, 测试文件, 配置文件, `packages/` — 这些是禁区。
- **你不亲自移除代码。** 你只负责扫描、验证、批处理，然后启动深度智能体。它们来做实际的工作。
</rules>

<false-positive-guards>
绝对不能标记为死代码：
- `src/index.ts` 或 barrel `index.ts` 重新导出的符号
- 在测试文件中引用的符号（测试是有效的消费者）
- 带有 `@public` / `@api` JSDoc 标签的符号
- Hook 工厂 (`createXXXHook`)、工具工厂 (`createXXXTool`)、`agentSources` 中的智能体定义
- 命令模板、技能定义、MCP 配置
- `package.json` 导出中的符号
</false-positive-guards>

---

## 第 1 阶段：扫描 — 查找死代码候选者

并行运行所有这些操作：

<parallel-scan>

**TypeScript 严格模式（你的主要扫描器 — 首先运行这个）：**
```bash
bunx tsc --noEmit --noUnusedLocals --noUnusedParameters 2>&1
```
这为你提供了未使用的局部变量、导入、参数和类型的权威列表，并包含精确的文件:行号位置。

**Explore 智能体（在后台同时触发所有这些）：**

```
task(subagent_type="explore", run_in_background=true, load_skills=[],
  description="查找孤立文件",
  prompt="查找 src/ 中未被任何其他文件导入的文件。检查所有导入语句。排除：index.ts, *.test.ts, 入口点, .md, packages/。返回：文件路径。")

task(subagent_type="explore", run_in_background=true, load_skills=[],
  description="查找未使用的导出符号",
  prompt="查找 src/ 中从未被其他文件导入的导出函数/类型/常量。交叉引用：对于每个导出，在 src/ 中 grep 该符号名称 — 如果它只出现在其自身文件中，则为候选者。排除：src/index.ts 导出，测试文件。返回：文件路径、行号、符号名称、导出类型。")
```

</parallel-scan>

收集所有结果汇总成一个主候选列表。

---

## 第 2 阶段：验证 — LSP 确认（零误报）

对于来自第 1 阶段的每个候选者：

```typescript
LspFindReferences(filePath, line, character, includeDeclaration=false)
// 0 引用 → 确认为死代码
// 1+ 引用 → 不是死代码，从列表中删除
```

还要应用上述的 false-positive-guards（误报守卫）。生成一个已确认的列表：

```
| # | 文件 | 符号 | 类型 | 操作 |
|---|------|--------|------|--------|
| 1 | src/foo.ts:42 | unusedFunc | function | REMOVE |
| 2 | src/bar.ts:10 | OldType | type | REMOVE |
| 3 | src/baz.ts:7 | ctx | parameter | PREFIX _ |
```

**操作类型：**
- `REMOVE` — 完全删除符号/导入/文件
- `PREFIX _` — 签名要求的未使用函数参数 → 重命名为 `_paramName`

如果确认数量为零：报告 "未找到死代码" 并停止。

---

## 第 3 阶段：批处理 — 按文件分组以实现无冲突的并行处理

<batching-rules>

**目标：最大化并行智能体且产生零 git 冲突。**

1. 将确认的死代码项按文件路径分组
2. 同一文件中的所有项目进入同一批次（防止两个智能体编辑同一个文件）
3. 如果存在死文件（整个文件删除），则将其单独作为一个批次
4. 目标是 5-15 个批次。如果总项目数少于 5，则每个项目使用 1 个批次。

**示例批处理：**
```
批次 A: [src/hooks/foo/hook.ts — 3 个未使用导入]
批次 B: [src/features/bar/manager.ts — 2 个未使用常量，1 个死函数]
批次 C: [src/tools/baz/tool.ts — 1 个未使用参数，src/tools/baz/types.ts — 1 个未使用类型]
批次 D: [src/dead-file.ts — 整个文件删除]
```

同一目录中的文件可以放在一起批处理（只要没有两个智能体编辑同一个文件，它们就不会发生冲突）。最大化批次数量以提高并行度。

</batching-rules>

---

## 第 4 阶段：执行 — 触发并行的深度智能体

对于每个批次，触发一个深度智能体：

```
task(
  category="deep",
  load_skills=["typescript-programmer", "git-master"],
  run_in_background=true,
  description="移除死代码批次 N: [简短描述]",
  prompt="[见下面的模板]"
)
```

<agent-prompt-template>

每个深度智能体都会得到这个提示词结构（根据每个批次的具体情况填写）：

```
## 任务: 从 [文件列表] 中移除死代码

## 要移除的死代码

### [文件路径] 第 [N] 行
- 符号: `[名称]` — [类型: 未使用导入 / 未使用常量 / 未使用函数 / 未使用参数 / 死文件]
- 操作: [完全 REMOVE / 从导入列表中 REMOVE / PREFIX 加上 _]

### [文件路径] 第 [N] 行
- ...

## 协议

1. 阅读每个文件，了解目标行处的精确语法
2. 对于每个符号，运行 LspFindReferences 重新验证它是否仍然是死代码（其他智能体可能已经改变了情况）
3. 应用更改：
   - 未使用导入（该行中的唯一符号）：移除整个导入行
   - 未使用导入（众多符号之一）：仅从导入列表中移除该符号
   - 未使用常量/函数/类型：移除声明。清理尾随的空白行。
   - 未使用参数：加上 `_` 前缀（不要移除 — 签名的要求）
   - 死文件：使用 `rm` 删除
4. 在此批次中进行所有编辑后，运行：`bun run typecheck`
5. 如果类型检查失败：`git checkout -- [文件]` 并报告失败
6. 如果类型检查通过：仅暂存你的文件并提交：
   `git add [你的特定文件] && git commit -m "refactor: 从 [简短文件列表] 移除死代码"`
7. 报告你移除了什么以及提交哈希

## 关键
- 仅暂存你批次的文件（`git add [特定文件]`）。切勿 `git add -A` — 其他智能体正在并行工作。
- 如果在你的编辑后类型检查失败，撤销所有更改并报告。不要尝试修复。
- 其他文件中预先存在的测试失败是预期的。只有你这个批次的类型检查最重要。
```

</agent-prompt-template>

同时触发所有批次。等待所有批次完成。

---

## 第 5 阶段：最终验证

所有智能体完成后：

```bash
bun run typecheck   # 必须通过
bun test            # 记录任何新的失败对比预先存在的失败
bun run build       # 必须通过
```

生成摘要：

```markdown
## 死代码移除完成

### 已移除
| # | 符号 | 文件 | 类型 | 提交 | 智能体 |
|---|--------|------|------|--------|-------|
| 1 | unusedFunc | src/foo.ts | function | abc1234 | 批次 A |

### 已跳过（智能体报告失败）
| # | 符号 | 文件 | 原因 |
|---|--------|------|--------|

### 验证
- 类型检查: PASS/FAIL
- 测试: X 个通过, Y 个失败 (Z 个预先存在)
- 构建: PASS/FAIL
- 移除总计: M 个文件中的 N 个符号
- 提交总计: K 个原子提交
- 使用的并行智能体数量: P
```

---

## 作用域控制

如果提供了 `$ARGUMENTS`，则缩小扫描范围：
- 文件路径 → 仅该文件
- 目录 → 仅该目录
- 符号名称 → 仅该符号
- `all` 或空 → 全项目扫描（默认）

## 中止条件

如果出现以下情况，停止并报告：
- 找到超过 50 个候选者（要求用户缩小范围或确认继续）
- 构建失败且无法通过还原修复

</command-instruction>

<user-request>
$ARGUMENTS
</user-request>
