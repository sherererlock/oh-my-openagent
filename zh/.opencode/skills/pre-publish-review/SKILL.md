---
name: pre-publish-review
description: "核弹级 16 智能体发布前审查门闸。运行 /get-unpublished-changes 以检测自上次 npm 发布以来的所有更改，生成多达 10 个 ultrabrain 智能体进行深度的逐项更改分析，调用 /review-work（5 个智能体）进行全面审查，并使用 1 个 oracle 智能体进行整体发布综合评估。在每次 npm 发布之前使用。触发词：'pre-publish review', 'review before publish', 'release review', 'pre-release review', 'ready to publish?', 'can I publish?', 'pre-publish', 'safe to publish', 'publishing review', 'pre-publish check'。"
---

# 发布前审查 — 16 智能体发布门闸

发布到 npm 之前的三层审查。每一层涵盖不同的角度 — 它们共同捕获任何单个审查者可能遗漏的问题。

| 层级 | 智能体数 | 类型 | 检查内容 |
|-------|--------|------|-----------------|
| 逐项更改深度剖析 | 最多 10 | ultrabrain | 逐个审查每个逻辑更改组 — 正确性、边缘情况、模式一致性 |
| 全面审查 | 5 | review-work | 在完整变更集中审查目标合规性、QA 执行、代码质量、安全性和上下文挖掘 |
| 发布综合评估 | 1 | oracle | 整体发布准备状态、版本升级、破坏性更改、部署风险 |

---

## 第 0 阶段：检测未发布更改

首先运行 `/get-unpublished-changes`。这是了解更改内容的唯一事实来源。

```
skill(name="get-unpublished-changes")
```

该命令自动：
- 检测已发布的 npm 版本与本地版本
- 列出自上次发布以来的所有提交
- 读取实际的 diff（而不仅仅是提交信息）以描述真实的更改
- 按类型（feat/fix/refactor/docs）和作用域对更改进行分组
- 识别破坏性更改
- 推荐版本升级（patch/minor/major）

**保存完整输出** — 它直接输入到阶段 1 分组和所有智能体提示词中。

然后捕获智能体提示词所需的原始数据：

```bash
# 提取版本（已在 /get-unpublished-changes 输出中）
PUBLISHED=$(npm view oh-my-opencode version 2>/dev/null || echo "not published")
LOCAL=$(node -p "require('./package.json').version" 2>/dev/null || echo "unknown")

# 供智能体使用的原始数据（diff，文件列表）
COMMITS=$(git log "v${PUBLISHED}"..HEAD --oneline 2>/dev/null || echo "no commits")
COMMIT_COUNT=$(echo "$COMMITS" | wc -l | tr -d ' ')
DIFF_STAT=$(git diff "v${PUBLISHED}"..HEAD --stat 2>/dev/null || echo "no diff")
CHANGED_FILES=$(git diff --name-only "v${PUBLISHED}"..HEAD 2>/dev/null || echo "none")
FILE_COUNT=$(echo "$CHANGED_FILES" | wc -l | tr -d ' ')
```

如果 `PUBLISHED` 是 "not published"（未发布），这说明是首次发布 — 使用完整的 git 历史记录代替。
---

## 第 1 阶段：将更改解析为分组

使用 `/get-unpublished-changes` 输出作为起点 — 它已经按作用域和类型进行了分类。

**分组策略：**
1. 从 `/get-unpublished-changes` 分析开始，该分析已经按带有作用域的 feat/fix/refactor/docs 进行了分类
2. 进一步按**模块/领域**拆分 — 涉及相同模块或功能领域的更改属于同一组
3. 目标是**最多 10 个组**。如果提交少于 10 个，每个提交独立为一组。如果有超过 10 个逻辑区域，合并最小的组。
4. 对于每个组，提取：
   - **组名**：简短的描述性标签（例如，"agent-model-resolution", "hook-system-refactor"）
   - **提交**：提交哈希和信息的列表
   - **文件**：此组中更改的文件
   - **Diff**：完整 diff 中的相关部分（`git diff v${PUBLISHED}..HEAD -- {组文件}`）

---

## 第 2 阶段：生成所有智能体

在单个回合中启动所有智能体。每个智能体使用 `run_in_background=true`。不要按顺序启动。

### 第 1 层：Ultrabrain 逐项更改分析（最多 10 个）

对于每个更改组，生成一个 ultrabrain 智能体。每个智能体仅获取其 diff 的一部分 — 而不是整个变更集。

```
task(
  category="ultrabrain",
  run_in_background=true,
  load_skills=[],
  description="深度分析: {GROUP_NAME}",
  prompt="""
<review_type>逐项更改深度分析</review_type>
<change_group>{GROUP_NAME}</change_group>

<project>oh-my-opencode (npm 包)</project>
<published_version>{PUBLISHED}</published_version>
<target_version>{LOCAL}</target_version>

<commits>
{GROUP_COMMITS — 此组中每个提交的哈希和信息}
</commits>

<changed_files>
{GROUP_FILES — 此组中更改的文件}
</changed_files>

<diff>
{GROUP_DIFF — 仅包含此组文件的 diff}
</diff>

<file_contents>
{读取并包含此组中每个更改文件的完整内容}
</file_contents>

你正在审查即将发布的 npm 版本中的特定更改子集。仅专注于此更改组。其他组由并行智能体审查。

分析检查清单：

1. **意图清晰度**: 这个更改试图做什么？从代码和提交信息中是否能清楚地看出意图？如果你需要猜测，那就是一个发现项。

2. **正确性**: 跟踪至少 3 种场景的逻辑。代码是否真的做到了它所声称的那样？差一错误、空值处理、异步边缘情况、资源清理。

3. **破坏性更改**: 这个更改是否修改了任何公共 API、配置格式、CLI 行为或 Hook 契约？如果是，它是否向后兼容？现有用户会感到意外吗？

4. **模式一致性**: 新代码是否遵循了现有文件内容中可见的既定模式？在旧模式存在的地方引入新模式 = 发现项。

5. **边缘情况**: 哪些输入或条件会破坏它？空数组、未定义值、并发调用、非常大的输入、缺少配置字段。

6. **错误处理**: 错误是否被正确捕获和传播？没有空的 catch 块？没有被吞噬的 Promise？

7. **类型安全**: 有任何 `as any`, `@ts-ignore`, `@ts-expect-error` 吗？在可能严格类型的地方使用了松散的类型？

8. **测试覆盖率**: 行为变化是否被测试覆盖？测试是有意义的，还是仅仅为了凑覆盖率？

9. **副作用**: 这个更改是否会破坏其他模块中的某些东西？检查导入和导出 — 谁依赖于发生改变的东西？

10. **发布风险**: 在 SAFE（安全）/ CAUTION（需谨慎）/ RISKY（风险高）的范围内 — 你有多大把握这个更改不会在生产环境中引起问题？

输出格式：
<group_name>{GROUP_NAME}</group_name>
<verdict>PASS 或 FAIL</verdict>
<risk>SAFE / CAUTION / RISKY</risk>
<summary>关于此更改组的 2-3 句话评估</summary>
<has_breaking_changes>YES 或 NO</has_breaking_changes>
<breaking_change_details>如果是 YES，描述什么被破坏了以及对谁造成了影响</breaking_change_details>
<findings>
  对于每个发现项：
  - [CRITICAL/MAJOR/MINOR] 类别: 描述
  - 文件: 路径 (行号范围)
  - 证据: 具体代码引用
  - 建议: 如何修复
</findings>
<blocking_issues>发布前必须修复的问题。如果是 PASS 则为空。</blocking_issues>
""")
```

### 第 2 层：通过 /review-work 进行全面审查（5 个智能体）

生成一个加载 `/review-work` 技能的子智能体。review-work 技能在内部启动 5 个并行智能体：Oracle（目标验证）、unspecified-high（QA 执行）、Oracle（代码质量）、Oracle（安全性）、unspecified-high（上下文挖掘）。所有 5 个都必须通过，审查才算通过。

```
task(
  category="unspecified-high",
  run_in_background=true,
  load_skills=["review-work"],
  description="对所有未发布的更改运行 /review-work",
  prompt="""
对 v{PUBLISHED} 和 HEAD 之间未发布的更改运行 /review-work。

目标: 审查 oh-my-opencode 的 npm 发布前所有更改。这些更改跨越了 {FILE_COUNT} 个文件中的 {COMMIT_COUNT} 个提交。

约束条件:
- 这是一个发布到 npm 的插件 — 公共 API 的稳定性很重要
- TypeScript 严格模式，Bun 运行时
- 禁止使用 `as any`, `@ts-ignore`, `@ts-expect-error`
- 工具、Hook、智能体使用工厂模式 (createXXX)
- kebab-case 文件命名，barrel 导出，没有大杂烩（catch-all）文件

背景: oh-my-opencode 的发布前审查，它是一个拥有 1268 个 TypeScript 文件、16 万行代码的 OpenCode 插件。自 v{PUBLISHED} 以来的更改即将发布。

diff 基础是: git diff v{PUBLISHED}..HEAD

完全遵循 /review-work 技能流程 — 启动所有 5 个审查智能体并收集结果。不要跳过 5 个智能体中的任何一个。
""")
```

### 第 3 层：Oracle 发布综合评估（1 个智能体）

Oracle 获取全貌 — 所有提交、完整的 diff 统计信息以及更改的文件列表。它提供最终的发布准备状态评估。

```
task(
  subagent_type="oracle",
  run_in_background=true,
  load_skills=[],
  description="Oracle: 整体发布综合评估及版本升级推荐",
  prompt="""
<review_type>发布综合评估 — 整体评估</review_type>

<project>oh-my-opencode (npm 包)</project>
<published_version>{PUBLISHED}</published_version>
<local_version>{LOCAL}</local_version>

<all_commits>
{自发布版本以来的所有提交 — 哈希、信息、作者、日期}
</all_commits>

<diff_stat>
{DIFF_STAT — 更改的文件、插入的行数、删除的行数}
</diff_stat>

<changed_files>
{CHANGED_FILES — 修改文件路径的完整列表}
</changed_files>

<full_diff>
{FULL_DIFF — 发布版本和 HEAD 之间的完整 git diff}
</full_diff>

<file_contents>
{读取并包含关键更改文件的完整内容 — 重点关注公共 API 层面、配置架构、智能体定义、Hook 注册、工具注册}
</file_contents>

你是 npm 发布前的最后一道关卡。10 个 ultrabrain 智能体正在审查单个更改，5 个 review-work 智能体正在进行全面审查。你的工作是从鸟瞰的角度查看那些重点审查可能遗漏的地方。

综合检查清单：

1. **发布的连贯性**: 这些更改是否讲述了一个连贯的故事？还是说这是一个应该拆分成多个发布的杂乱无章的更改集合？

2. **版本升级**: 基于语义化版本（semver）：
   - PATCH: 仅修复错误，无行为更改
   - MINOR: 新功能，向后兼容的更改
   - MAJOR: 对公共 API、配置格式或行为的破坏性更改
   建议正确的升级级别并给出具体的理由。

3. **破坏性更改审计**: 详尽列出可能破坏现有用户的每一个更改。检查：
   - 配置架构更改（新的必填字段、移除的字段、重命名的字段）
   - 智能体行为更改（不同的提示词、不同的模型路由）
   - Hook 契约更改（新参数、移除的 Hook、重命名的 Hook）
   - 工具接口更改（新的必填参数、不同的返回类型）
   - CLI 更改（新命令、更改的标志、不同的输出）
   - 技能格式更改（SKILL.md 架构更改）

4. **迁移要求**: 如果存在破坏性更改，用户需要什么迁移步骤？是否有自动迁移机制？

5. **依赖项更改**: 添加了新依赖项？移除了依赖项？版本升级？是否有任何供应链风险？

6. **变更日志草稿**: 编写一个草稿变更日志条目，分组方式如下：
   - feat: 新功能
   - fix: 错误修复
   - refactor: 内部更改（无用户影响）
   - breaking: 带有迁移说明的破坏性更改
   - docs: 文档更改

7. **部署风险评估**:
   - SAFE: 常规更改，测试充分，风险低
   - CAUTION: 重大更改但风险可控
   - RISKY: 更改范围大，测试不充分，或存在没有迁移说明的破坏性更改
   - BLOCK: 发现关键问题，不要发布

8. **发布后监控**: 发布后应该监控什么？错误率、特定功能、用户反馈渠道。

输出格式：
<verdict>SAFE / CAUTION / RISKY / BLOCK</verdict>
<recommended_version_bump>PATCH / MINOR / MAJOR</recommended_version_bump>
<version_bump_justification>为什么是这个升级级别</version_bump_justification>
<release_coherence>对更改是否属于同一个发布的评估</release_coherence>
<breaking_changes>
  详尽的列表，如果没有则为 "None"。
  对于每一个：
  - 更改了什么
  - 谁受到影响
  - 迁移步骤
</breaking_changes>
<changelog_draft>
  准备使用的变更日志条目
</changelog_draft>
<deployment_risk>
  整体风险评估及具体顾虑
</deployment_risk>
<monitoring_recommendations>
  发布后要观察什么
</monitoring_recommendations>
<blocking_issues>发布前必须修复的问题。如果是 SAFE 则为空。</blocking_issues>
""")
```

---

## 第 3 阶段：收集结果

当智能体完成（系统通知）时，通过 `background_output(task_id="...")` 收集。

在表格中跟踪完成情况：

| # | 智能体 | 类型 | 状态 | 结论 |
|---|-------|------|--------|---------|
| 1-10 | Ultrabrain: {group_name} | ultrabrain | pending | — |
| 11 | Review-Work 协调者 | unspecified-high | pending | — |
| 12 | 发布综合评估 Oracle | oracle | pending | — |

在所有智能体完成之前，不要提交最终报告。

---

## 第 4 阶段：最终结论

<verdict_logic>

**BLOCK (拦截)** 如果：
- Oracle 的结论是 BLOCK
- 任何 ultrabrain 发现了 CRITICAL (关键) 阻碍问题
- review-work 在任何 MAIN (主) 智能体上失败

**RISKY (高风险)** 如果：
- Oracle 的结论是 RISKY
- 多个 ultrabrain 返回 CAUTION 或 FAIL
- review-work 通过，但有重大发现

**CAUTION (需谨慎)** 如果：
- Oracle 的结论是 CAUTION
- 少数几个 ultrabrain 标记了轻微问题
- review-work 顺利通过

**SAFE (安全)** 如果：
- Oracle 的结论是 SAFE
- 所有 ultrabrain 都通过
- review-work 通过

</verdict_logic>

汇编最终报告：

```markdown
# 发布前审查 — oh-my-opencode

## 发布: v{PUBLISHED} -> v{LOCAL}
**提交:** {COMMIT_COUNT} | **更改的文件:** {FILE_COUNT} | **智能体:** {AGENT_COUNT}

---

## 整体结论: SAFE / CAUTION / RISKY / BLOCK

## 建议的版本升级: PATCH / MINOR / MAJOR
{来自 Oracle 的理由}

---

## 逐项更改分析 (Ultrabrains)

| # | 更改组 | 结论 | 风险 | 破坏性？ | 阻碍问题 |
|---|-------------|---------|------|-----------|-----------------|
| 1 | {name} | PASS/FAIL | SAFE/CAUTION/RISKY | YES/NO | {数量 或 "none"} |
| ... | ... | ... | ... | ... | ... |

### 逐项更改分析中的阻碍问题
{聚合自所有 ultrabrains — 已去重}

---

## 全面审查 (Review-Work)

| # | 审查领域 | 结论 | 置信度 |
|---|------------|---------|------------|
| 1 | 目标与约束验证 | PASS/FAIL | HIGH/MED/LOW |
| 2 | QA 执行 | PASS/FAIL | HIGH/MED/LOW |
| 3 | 代码质量 | PASS/FAIL | HIGH/MED/LOW |
| 4 | 安全性 | PASS/FAIL | 严重程度 |
| 5 | 上下文挖掘 | PASS/FAIL | HIGH/MED/LOW |

### 全面审查中的阻碍问题
{聚合自 review-work}

---

## 发布综合评估 (Oracle)

### 破坏性更改
{来自 Oracle — 详尽的列表或 "None"}

### 变更日志草稿
{来自 Oracle — 准备使用的}

### 部署风险
{来自 Oracle — 具体顾虑}

### 发布后监控
{来自 Oracle — 要观察什么}

---

## 所有阻碍问题 (已优先级排序)
{已去重，合并自所有三层，按严重程度排序}

## 建议
{如果是 BLOCK/RISKY: 确切说明要修复什么，按优先级排序}
{如果是 CAUTION: 值得在发布前考虑的建议}
{如果是 SAFE: 供未来参考的非阻碍性改进}
```

---

## 反模式

| 违规 | 严重程度 |
|-----------|----------|
| 没有等待所有智能体就进行发布 | **CRITICAL** |
| 按顺序而不是并行生成 ultrabrains | CRITICAL |
| 对任何智能体使用 `run_in_background=false` | CRITICAL |
| 跳过 Oracle 综合评估 | HIGH |
| 没有读取 Oracle 的文件内容（它不能自己读取文件） | HIGH |
| 将所有更改分组到 1-2 个 ultrabrains 而不是分布开来 | HIGH |
| 在所有智能体完成之前给出结论 | HIGH |
| 在 ultrabrain 提示词中没有包含 diff | MAJOR |
