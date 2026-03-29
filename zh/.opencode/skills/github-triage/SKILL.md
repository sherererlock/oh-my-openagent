---
name: github-triage
description: "用于 issue 和 PR 的只读 GitHub 筛选分类。1 个项目 = 1 个后台任务（类别：quick）。分析所有未结项目，并将基于证据的报告写入 /tmp/{datetime}/。每一个断言都需要 GitHub 永久链接作为证据。绝不在 GitHub 上采取任何操作 - 没有评论，没有合并，没有关闭，没有标签。仅限报告。触发词：'triage', 'triage issues', 'triage PRs', 'github triage'。"
---

# GitHub Triage - 只读分析器

<role>
只读 GitHub 筛选分类协调者。获取打开的 issue/PR，分类，为每个项目生成 1 个后台 `quick` 子智能体。每个子智能体进行分析并编写报告文件。零 GitHub 状态变更。
</role>

## 架构

**1 个 ISSUE/PR = 1 个 `task_create` = 1 个 `quick` 子智能体（后台）。没有例外。**

| 规则 | 值 |
|------|-------|
| 类别 | `quick` |
| 执行 | `run_in_background=true` |
| 并行度 | 所有项目同时进行 |
| 跟踪 | 每个项目使用 `task_create` |
| 输出 | `/tmp/{YYYYMMDD-HHmmss}/issue-{N}.md` 或 `pr-{N}.md` |

---

## 零操作策略（绝对）

<zero_action>
子智能体绝不能运行任何写入或改变 GitHub 状态的命令。

**禁止**（不详尽）：
`gh issue comment`, `gh issue close`, `gh issue edit`, `gh pr comment`, `gh pr merge`, `gh pr review`, `gh pr edit`, `gh api -X POST`, `gh api -X PUT`, `gh api -X PATCH`, `gh api -X DELETE`

**允许**：
- `gh issue view`, `gh pr view`, `gh api`（仅 GET）- 读取 GitHub 数据
- `Grep`, `Read`, `Glob` - 读取代码库
- `Write` - 仅将报告文件写入 `/tmp/`
- `git log`, `git show`, `git blame` - 读取 git 历史（用于寻找修复提交）

**任何对 GitHub 的更改 = 严重违规。**
</zero_action>

---

## 证据规则（强制）

<evidence>
**报告中的每个事实主张都必须包含 GitHub 永久链接作为证据。**

永久链接是指向特定提交中特定行/范围的 URL，例如：
`https://github.com/{owner}/{repo}/blob/{commit_sha}/{path}#L{start}-L{end}`

### 如何生成永久链接

1. 通过 Grep/Read 找到相关文件和行。
2. 获取当前提交 SHA：`git rev-parse HEAD`
3. 构造：`https://github.com/{REPO}/blob/{SHA}/{filepath}#L{line}`（或使用 `#L{start}-L{end}` 表示范围）

### 规则

- **没有永久链接 = 没有主张。** 如果你不能用永久链接支持一个陈述，请声明“未找到证据”。
- 没有永久链接的主张会被显式标记为 `[UNVERIFIED]`（未验证），并且权重为零。
- 指向 `main`/`master`/`dev` 分支的永久链接是不被接受的 - 只能使用提交 SHA。
- 对于错误分析：永久链接到有问题的代码。对于修复验证：永久链接到修复的提交 diff。
</evidence>

---

## 阶段 0：设置

```bash
REPO=$(gh repo view --json nameWithOwner -q .nameWithOwner)
REPORT_DIR="/tmp/$(date +%Y%m%d-%H%M%S)"
mkdir -p "$REPORT_DIR"
COMMIT_SHA=$(git rev-parse HEAD)
```

将 `REPO`、`REPORT_DIR` 和 `COMMIT_SHA` 传递给每个子智能体。

---

---

## 阶段 1：获取所有未结项目（已修正）

**重要：** `body` 和 `comments` 字段可能包含破坏 jq 解析的控制字符。首先获取基本元数据，然后在子智能体中获取每个项目的完整细节。

```bash
# 步骤 1：获取基本元数据（不包含 body/comments 以避免 JSON 解析问题）
ISSUES_LIST=$(gh issue list --repo $REPO --state open --limit 500 \
  --json number,title,labels,author,createdAt)
ISSUE_COUNT=$(echo "$ISSUES_LIST" | jq length)

# 需要时分页
if [ "$ISSUE_COUNT" -eq 500 ]; then
  LAST_DATE=$(echo "$ISSUES_LIST" | jq -r '.[-1].createdAt')
  while true; do
    PAGE=$(gh issue list --repo $REPO --state open --limit 500 \
      --search "created:<$LAST_DATE" \
      --json number,title,labels,author,createdAt)
    PAGE_COUNT=$(echo "$PAGE" | jq length)
    [ "$PAGE_COUNT" -eq 0 ] && break
    ISSUES_LIST=$(echo "$ISSUES_LIST" "$PAGE" | jq -s '.[0] + .[1] | unique_by(.number)')
    ISSUE_COUNT=$(echo "$ISSUES_LIST" | jq length)
    [ "$PAGE_COUNT" -lt 500 ] && break
    LAST_DATE=$(echo "$PAGE" | jq -r '.[-1].createdAt')
  done
fi

# 对 PR 做同样的操作
PRS_LIST=$(gh pr list --repo $REPO --state open --limit 500 \
  --json number,title,labels,author,headRefName,baseRefName,isDraft,createdAt)
PR_COUNT=$(echo "$PRS_LIST" | jq length)

if [ "$PR_COUNT" -eq 500 ]; then
  LAST_DATE=$(echo "$PRS_LIST" | jq -r '.[-1].createdAt')
  while true; do
    PAGE=$(gh pr list --repo $REPO --state open --limit 500 \
      --search "created:<$LAST_DATE" \
      --json number,title,labels,author,headRefName,baseRefName,isDraft,createdAt)
    PAGE_COUNT=$(echo "$PAGE" | jq length)
    [ "$PAGE_COUNT" -eq 0 ] && break
    PRS_LIST=$(echo "$PRS_LIST" "$PAGE" | jq -s '.[0] + .[1] | unique_by(.number)')
    PR_COUNT=$(echo "$PRS_LIST" | jq length)
    [ "$PAGE_COUNT" -lt 500 ] && break
    LAST_DATE=$(echo "$PAGE" | jq -r '.[-1].createdAt')
  done
fi

echo "Total issues: $ISSUE_COUNT, Total PRs: $PR_COUNT"
```

**大型仓库处理：**
如果项目总数超过 50 个，你**必须**处理所有项目。使用上面的分页代码来获取每一个打开的 issue 和 PR。
**不要**进行抽样或限制为 50 个项目 - 处理整个积压工作。

示例：如果有 500 个打开的 issue，生成 500 个子智能体。如果有 1000 个打开的 PR，生成 1000 个子智能体。

**注意：** 后台任务系统会自动排队过多的任务。


---

## 阶段 2：分类

| 类型 | 检测方式 |
|------|-----------|
| `ISSUE_QUESTION` | `[Question]`, `[Discussion]`, `?`, "how to" / "why does" / "is it possible" |
| `ISSUE_BUG` | `[Bug]`, `Bug:`, 错误信息, 堆栈跟踪, 意外行为 |
| `ISSUE_FEATURE` | `[Feature]`, `[RFE]`, `[Enhancement]`, `Feature Request`, `Proposal` |
| `ISSUE_OTHER` | 其他任何内容 |
| `PR_BUGFIX` | 标题以 `fix` 开头, 分支包含 `fix/`/`bugfix/`, 标签 `bug` |
| `PR_OTHER` | 其他任何内容 |

---

## 阶段 3：生成子智能体（单个工具调用）

**关键：使用单独的 `task_create` 工具调用一个接一个地创建任务。绝不能批处理或脚本化。**

对于每个项目，按顺序执行以下步骤：

### 步骤 3.1：创建任务记录
```typescript
task_create(
  subject="Triage: #{number} {title}",
  description="GitHub {issue|PR} triage analysis - {type}",
  metadata={"type": "{ISSUE_QUESTION|ISSUE_BUG|ISSUE_FEATURE|ISSUE_OTHER|PR_BUGFIX|PR_OTHER}", "number": {number}}
)
```

### 步骤 3.2：生成分析子智能体（后台）
```typescript
task(
  category="quick",
  run_in_background=true,
  load_skills=[],
  prompt=SUBAGENT_PROMPT
)
```

**子智能体的绝对规则：**
- **仅分析** - 绝不在 GitHub 上采取行动（不评论，不合并，不关闭）
- **只读** - 工具仅用于读取代码/GitHub 数据
- **只写报告** - 输出仅通过 Write 工具写入 `{REPORT_DIR}/{issue|pr}-{number}.md`
- **需要证据** - 每个主张都必须有 GitHub 永久链接作为证据

```
对于每个项目：
  1. task_create(subject="Triage: #{number} {title}")
  2. task(category="quick", run_in_background=true, load_skills=[], prompt=SUBAGENT_PROMPT)
  3. 存储映射：item_number -> { task_id, background_task_id }
```

---

## 子智能体提示词

### 通用前言（包含在所有子智能体提示词中）

```
上下文：
- 仓库: {REPO}
- 报告目录: {REPORT_DIR}
- 当前提交 SHA: {COMMIT_SHA}

永久链接格式：
每个事实主张都必须包含一个永久链接：https://github.com/{REPO}/blob/{COMMIT_SHA}/{filepath}#L{start}-L{end}
没有永久链接 = 没有主张。将无法验证的主张标记为 [UNVERIFIED]。
如果需要获取当前 SHA：git rev-parse HEAD

绝对规则（违反任何一条 = 严重失败）：
- 绝不能运行 gh issue comment, gh issue close, gh issue edit
- 绝不能运行 gh pr comment, gh pr merge, gh pr review, gh pr edit
- 绝不能运行任何带有 -X POST, -X PUT, -X PATCH, -X DELETE 的 gh 命令
- 绝不能运行 git checkout, git fetch, git pull, git switch, git worktree
- 你唯一可写的输出：通过 Write 工具写入 {REPORT_DIR}/{issue|pr}-{number}.md
```


---

### ISSUE_QUESTION

```
你正在分析 {REPO} 的 issue #{number}。

项目：
- Issue #{number}: {title}
- 作者: {author}
- 正文: {body}
- 评论: {comments_summary}

任务：
1. 理解问题。
2. 搜索代码库 (Grep, Read) 寻找答案。
3. 对于每个发现，构造一个永久链接：https://github.com/{REPO}/blob/{COMMIT_SHA}/{path}#L{N}
4. 将报告写入 {REPORT_DIR}/issue-{number}.md

报告格式（将此作为文件内容写入）：

# Issue #{number}: {title}
**类型:** 问题 | **作者:** {author} | **创建时间:** {createdAt}

## 问题
[1-2 句话摘要]

## 发现
[包含永久链接证据的每个发现。例如：]
- 配置在 [`src/config/loader.ts#L42-L58`](https://github.com/{REPO}/blob/{SHA}/src/config/loader.ts#L42-L58) 解析

## 建议答案
[包含代码引用和永久链接的答案草稿]

## 置信度: [HIGH (高) | MEDIUM (中) | LOW (低)]
[原因。如果是低：缺少什么]

## 推荐操作
[维护者应该做什么]

---
记住：没有永久链接 = 没有主张。每个代码引用都需要一个永久链接。
```

---

### ISSUE_BUG

```
你正在分析 {REPO} 的错误报告 #{number}。

项目：
- Issue #{number}: {title}
- 作者: {author}
- 正文: {body}
- 评论: {comments_summary}

任务：
1. 理解：预期行为、实际行为、重现步骤。
2. 搜索代码库以查找相关代码。追踪逻辑。
3. 决定结论：CONFIRMED_BUG（确认的错误）、NOT_A_BUG（不是错误）、ALREADY_FIXED（已修复）或 UNCLEAR（不清楚）。
4. 对于 ALREADY_FIXED：使用 git log/git blame 找到修复提交。包括提交 SHA 和更改内容。
5. 对于每个发现，构造一个永久链接。
6. 将报告写入 {REPORT_DIR}/issue-{number}.md

寻找 "ALREADY_FIXED"（已修复）提交：
- 使用 `git log --all --oneline -- {file}` 查找相关文件的近期更改
- 使用 `git log --all --grep="fix" --grep="{keyword}" --all-match --oneline` 搜索提交信息
- 使用 `git blame {file}` 查找谁最后修改了相关行
- 使用 `git show {commit_sha}` 验证修复
- 构造提交永久链接：https://github.com/{REPO}/commit/{fix_commit_sha}

报告格式（将此作为文件内容写入）：

# Issue #{number}: {title}
**类型:** 错误报告 | **作者:** {author} | **创建时间:** {createdAt}

## 错误摘要
**预期:** [用户期望的内容]
**实际:** [实际发生的内容]
**重现:** [步骤，如果提供的话]

## 结论: [CONFIRMED_BUG | NOT_A_BUG | ALREADY_FIXED | UNCLEAR]

## 分析

### 证据
[每一条证据都带有永久链接。没有永久链接 = 标记 [UNVERIFIED]]

### 根本原因 (如果是 CONFIRMED_BUG)
[哪个文件，哪个函数，哪里出错了]
- 有问题的代码：[`{path}#L{N}`](permalink)

### 为什么不是错误 (如果是 NOT_A_BUG)
[带有永久链接的严格证明，证明当前行为是正确的]

### 修复细节 (如果是 ALREADY_FIXED)
- **修复提交:** [`{short_sha}`](https://github.com/{REPO}/commit/{full_sha})
- **修复日期:** {date}
- **更改内容:** [包含 diff 永久链接的描述]
- **修复者:** {author}

### 阻碍 (如果是 UNCLEAR)
[是什么阻碍了确定结论，接下来要调查什么]

## 严重程度: [LOW (低) | MEDIUM (中) | HIGH (高) | CRITICAL (严重)]

## 受影响的文件
[带有永久链接的列表]

## 建议修复 (如果是 CONFIRMED_BUG)
[具体方法："在 {file}#L{N} 中，将 X 更改为 Y，因为 Z"]

## 推荐操作
[维护者应该做什么]

---
关键：没有永久链接的主张毫无价值。如果你找不到证据，明确说明，而不是做出未经证实的声明。
```

---

### ISSUE_FEATURE

```
你正在分析 {REPO} 的功能请求 #{number}。

项目：
- Issue #{number}: {title}
- 作者: {author}
- 正文: {body}
- 评论: {comments_summary}

任务：
1. 理解请求。
2. 搜索代码库以查找现有（部分/完全）的实现。
3. 评估可行性。
4. 将报告写入 {REPORT_DIR}/issue-{number}.md

报告格式（将此作为文件内容写入）：

# Issue #{number}: {title}
**类型:** 功能请求 | **作者:** {author} | **创建时间:** {createdAt}

## 请求摘要
[用户想要什么]

## 现有实现: [YES_FULLY (完全) | YES_PARTIALLY (部分) | NO (无)]
[如果存在：在哪里，带有实现的永久链接]

## 可行性: [EASY (容易) | MODERATE (中等) | HARD (困难) | ARCHITECTURAL_CHANGE (架构更改)]

## 相关文件
[带有永久链接]

## 实现说明
[方法，陷阱，依赖项]

## 推荐操作
[维护者应该做什么]
```

---

### ISSUE_OTHER

```
你正在分析 {REPO} 的 issue #{number}。

项目：
- Issue #{number}: {title}
- 作者: {author}
- 正文: {body}
- 评论: {comments_summary}

任务：评估并将报告写入 {REPORT_DIR}/issue-{number}.md

报告格式（将此作为文件内容写入）：

# Issue #{number}: {title}
**类型:** [QUESTION | BUG | FEATURE | DISCUSSION | META | STALE]
**作者:** {author} | **创建时间:** {createdAt}

## 摘要
[1-2 句话]

## 需要注意: [YES | NO]
## 建议标签: [如果有]
## 推荐操作: [维护者应该做什么]
```

---

### PR_BUGFIX

```
你正在审查 {REPO} 的 PR #{number}。

项目：
- PR #{number}: {title}
- 作者: {author}
- 基础分支: {baseRefName} <- 头部: {headRefName}
- 草稿: {isDraft} | 可合并: {mergeable}
- 审查: {reviewDecision} | CI: {statusCheckRollup_summary}
- 正文: {body}

任务：
1. 获取 PR 详情 (只读): gh pr view {number} --repo {REPO} --json files,reviews,comments,statusCheckRollup,reviewDecision
2. 读取 diff: gh api repos/{REPO}/pulls/{number}/files
3. 搜索代码库以验证修复的正确性。
4. 将报告写入 {REPORT_DIR}/pr-{number}.md

报告格式（将此作为文件内容写入）：

# PR #{number}: {title}
**类型:** Bugfix (错误修复) | **作者:** {author}
**基础:** {baseRefName} <- {headRefName} | **草稿:** {isDraft}

## 修复摘要
[什么错误，如何修复 - 带有更改代码的永久链接]

## 代码审查

### 正确性
[修复正确吗？根本原因解决了没？带有永久链接的证据]

### 副作用
[风险更改，破坏性更改 - 如果有请附带永久链接]

### 代码质量
[风格，模式，测试覆盖率]

## 合并准备状态

| 检查项 | 状态 |
|-------|--------|
| CI | [PASS / FAIL / PENDING] |
| 审查 | [APPROVED / CHANGES_REQUESTED / PENDING / NONE] |
| 可合并 | [YES / NO / CONFLICTED] |
| 草稿 | [YES / NO] |
| 正确性 | [VERIFIED / CONCERNS / UNCLEAR] |
| 风险 | [NONE / LOW / MEDIUM / HIGH] |

## 更改的文件
[带有简短描述的列表]

## 推荐操作: [MERGE (合并) | REQUEST_CHANGES (请求更改) | NEEDS_REVIEW (需要审查) | WAIT (等待)]
[带有证据的推理]

---
绝不合并。绝不评论。绝不审查。仅写入文件。
```

---

### PR_OTHER

```
你正在审查 {REPO} 的 PR #{number}。

项目：
- PR #{number}: {title}
- 作者: {author}
- 基础分支: {baseRefName} <- 头部: {headRefName}
- 草稿: {isDraft} | 可合并: {mergeable}
- 审查: {reviewDecision} | CI: {statusCheckRollup_summary}
- 正文: {body}

任务：
1. 获取 PR 详情 (只读): gh pr view {number} --repo {REPO} --json files,reviews,comments,statusCheckRollup,reviewDecision
2. 读取 diff: gh api repos/{REPO}/pulls/{number}/files
3. 将报告写入 {REPORT_DIR}/pr-{number}.md

报告格式（将此作为文件内容写入）：

# PR #{number}: {title}
**类型:** [FEATURE | REFACTOR | DOCS | CHORE | TEST | OTHER]
**作者:** {author}
**基础:** {baseRefName} <- {headRefName} | **草稿:** {isDraft}

## 摘要
[2-3 句话，带有关键更改的永久链接]

## 状态

| 检查项 | 状态 |
|-------|--------|
| CI | [PASS / FAIL / PENDING] |
| 审查 | [APPROVED / CHANGES_REQUESTED / PENDING / NONE] |
| 可合并 | [YES / NO / CONFLICTED] |
| 风险 | [LOW / MEDIUM / HIGH] |
| 对齐度 | [YES / NO / UNCLEAR] |

## 更改的文件
[数量和关键文件]

## 阻碍
[如果有]

## 推荐操作: [MERGE (合并) | REQUEST_CHANGES (请求更改) | NEEDS_REVIEW (需要审查) | CLOSE (关闭) | WAIT (等待)]
[推理]

---
绝不合并。绝不评论。绝不审查。仅写入文件。
```

---

## 阶段 4：收集与更新

轮询每个任务的 `background_output()`。当每个任务完成时：
1. 解析报告。
2. `task_update(id=task_id, status="completed", description=REPORT_SUMMARY)`
3. 立即流式传输给用户。

---

## 阶段 5：最终摘要

写入 `{REPORT_DIR}/SUMMARY.md` 并展示给用户：

```markdown
# GitHub Triage 报告 - {REPO}

**日期:** {date} | **提交:** {COMMIT_SHA}
**处理项目:** {total}
**报告目录:** {REPORT_DIR}

## Issues ({issue_count})
| 类别 | 数量 |
|----------|-------|
| 确认的错误 | {n} |
| 已修复的错误 | {n} |
| 不是错误 | {n} |
| 需要调查 | {n} |
| 问题已分析 | {n} |
| 功能已评估 | {n} |
| 其他 | {n} |

## PRs ({pr_count})
| 类别 | 数量 |
|----------|-------|
| 错误修复已审查 | {n} |
| 其他 PR 已审查 | {n} |

## 需要注意的项目
[每个项目：编号，标题，结论，单行摘要，报告文件的链接]

## 报告文件
[所有生成的文件及路径]
```

---

## 反模式

| 违规 | 严重程度 |
|-----------|----------|
| 任何 GitHub 状态变更 (comment/close/merge/review/label/edit) | **CRITICAL** |
| 没有永久链接的主张 | **CRITICAL** |
| 使用 `quick` 以外的类别 | CRITICAL |
| 将多个项目批处理到一个任务中 | CRITICAL |
| `run_in_background=false` | CRITICAL |
| 在 PR 分支上执行 `git checkout` | CRITICAL |
| 没有代码库证据的猜测 | HIGH |
| 没有将报告写入 `{REPORT_DIR}` | HIGH |
| 在永久链接中使用分支名而不是提交 SHA | HIGH |
