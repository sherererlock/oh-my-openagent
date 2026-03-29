# src/tools/hashline-edit/ — 基于哈希锚点的文件编辑工具

**生成日期:** 2026-03-06

## 概览

24 个文件。实现了 `hashline_edit` 工具 — 基于哈希锚点的文件编辑，其中每个行引用都包含一个内容哈希（`LINE#ID`）。在应用编辑之前验证哈希，拒绝过时的引用。

## 三操作模型

所有的编辑都恰好使用 3 种操作：

| 操作 | pos | end | lines | 效果 |
|----|-----|-----|-------|--------|
| `replace` | 必需 | 可选 | 必需 | 替换单行或范围 pos..end |
| `append` | 可选 | 可选 | 必需 | 在锚点后插入（如果没有锚点则在文件末尾 EOF 插入） |
| `prepend` | 可选 | 可选 | 必需 | 在锚点前插入（如果没有锚点则在文件开头 BOF 插入） |

使用 `replace` 且 `lines: null` 或 `lines: []` = 删除。在工具级别设置 `delete: true` = 删除文件。

## 执行流水线

```
hashline-edit-executor.ts
  → normalize-edits.ts       # 解析 RawHashlineEdit → HashlineEdit (验证操作 schema)
  → validation.ts            # 验证 LINE#ID 引用 (哈希匹配，行存在)
  → edit-ordering.ts         # 自下而上排序 (按行号降序)
  → edit-deduplication.ts    # 移除重复的操作
  → edit-operations.ts       # 使用 edit-operation-primitives.ts 应用每个操作
  → autocorrect-replacement-lines.ts  # 自动修复缩进/格式
  → hashline-edit-diff.ts    # 使用 diff-utils.ts 构建差异输出
```

## 关键文件

| 文件 | 用途 |
|------|---------|
| `tools.ts` | `createHashlineEditTool()` 工厂 — 工具 schema + 入口点 |
| `hashline-edit-executor.ts` | 主执行流程：标准化 → 验证 → 排序 → 应用 → 差异 (diff) |
| `normalize-edits.ts` | 解析 `RawHashlineEdit[]`（允许字符串 `op` 变体）→ 类型化的 `HashlineEdit[]` |
| `validation.ts` | 验证 LINE#ID：解析哈希，验证行内容与存储的哈希匹配 |
| `hash-computation.ts` | `computeLineHash(line)` → 从字符集 `ZPMQVRWSNKTXJBYH` 生成 2 字符的 CID |
| `edit-operations.ts` | 将 replace/append/prepend 应用于文件行数组 |
| `edit-operation-primitives.ts` | 底层的行数组突变原语 |
| `edit-ordering.ts` | 自下而上对编辑进行排序，以在多重编辑期间保留行号 |
| `edit-deduplication.ts` | 对重叠/相同的操作进行去重 |
| `edit-text-normalization.ts` | 标准化行内容（CRLF，BOM，尾随空格） |
| `file-text-canonicalization.ts` | 在哈希计算之前对完整文件内容进行规范化 |
| `autocorrect-replacement-lines.ts` | 从原始行中自动恢复缩进 |
| `hashline-edit-diff.ts` | 为错误/成功消息生成统一的差异 (unified diff) |
| `diff-utils.ts` | `diff` npm 库的轻量级包装器 |
| `hashline-chunk-formatter.ts` | 使用 `LINE#ID` 标签格式化行块 |
| `tool-description.ts` | `HASHLINE_EDIT_DESCRIPTION` 常量 |
| `types.ts` | `HashlineEdit`、`ReplaceEdit`、`AppendEdit`、`PrependEdit` |
| `constants.ts` | 哈希字母表，分隔符 (`#`)，管道分隔符 (`|`) |

## LINE#ID 格式

```
{line_number}#{hash_id}
```

- `hash_id`: 来自 `ZPMQVRWSNKTXJBYH` (CID 字母) 的两个字符
- 示例: `42#VK` 表示行号为 42，哈希为 `VK` 的行
- 验证: 重新计算当前行内容的哈希 → 必须与存储的哈希匹配
- 内容分隔符: 读取输出中哈希标签和内容之间的 `|` (管道)

## 自动更正行为 (内置)

- 合并的行会自动扩展回原始数量
- 从原始行中恢复缩进
- 保留 BOM 和 CRLF 行尾
- 自动剥离 `lines` 文本中的 `>>>` 前缀和 diff 标记

## 错误情况

- 哈希不匹配 → 拒绝编辑，显示当前状态的差异
- 范围重叠 → 被检测到并拒绝
- `replace` 缺少 `pos` → schema 错误
- `append`/`prepend` 且 `lines: null` → schema 错误

## 行哈希的工作原理

```typescript
// 读取时：每一行都会被打上标签
"42#VK| function hello() {"

// 编辑时：通过标签引用
{ op: "replace", pos: "42#VK", lines: "function hello(name: string) {" }

// 如果文件自读取后发生更改：哈希将不匹配 → 在损坏文件前拒绝
```