---
name: work-with-pr
description: "完整的 PR 生命周期：git worktree → 实现 → 原子提交 → 创建 PR → 验证循环（CI + review-work + Cubic 批准）→ 合并。持续迭代，直到所有关卡通过且 PR 被合并。合并后自动清理 worktree。每当实现工作需要作为 PR 落地时使用。触发词：'create a PR', 'implement and PR', 'work on this and make a PR', 'implement issue', 'land this as a PR', 'work-with-pr', 'PR workflow', 'implement end to end'，即使用户只是说 'implement X' 但上下文暗示了要提交 PR。"
---

# Work With PR — 完整的 PR 生命周期

你正在执行完整的 PR 生命周期：从隔离的 worktree 设置到实现、创建 PR，以及无上限的验证循环，直到 PR 被合并。循环包含三个关卡 — CI、review-work 和 Cubic — 你需要不断修复和推送，直到这三者同时通过。

<architecture>

```
第 0 阶段: 设置         → 在兄弟目录中创建分支 + worktree
第 1 阶段: 实现         → 执行工作，原子提交
第 2 阶段: 创建 PR      → 推送，创建针对 dev 的 PR
第 3 阶段: 验证循环     → 无上限迭代，直到所有关卡通过：
  ├─ 关卡 A: CI         → gh pr checks (bun test, typecheck, build)
  ├─ 关卡 B: review-work → 5 智能体并行审查
  └─ 关卡 C: Cubic      → cubic-dev-ai[bot] "No issues found" (未发现问题)
第 4 阶段: 合并         → 压缩合并，清理 worktree
```

</architecture>

---

## 第 0 阶段：设置

创建一个隔离的 worktree，以便用户的主工作目录保持干净。这很重要，因为用户可能有未提交的工作，检出分支会破坏它。

<setup>

### 1. 解析仓库上下文

```bash
REPO=$(gh repo view --json nameWithOwner -q .nameWithOwner)
REPO_NAME=$(basename "$PWD")
BASE_BRANCH="dev"  # CI 阻止 PR 直接提交到 master
```

### 2. 创建分支

如果用户提供了分支名称，则使用它。否则，从任务中派生：

```bash
# 自动生成: feature/短描述 或 fix/短描述
BRANCH_NAME="feature/$(echo "$TASK_SUMMARY" | tr '[:upper:] ' '[:lower:]-' | head -c 50)"
git fetch origin "$BASE_BRANCH"
git branch "$BRANCH_NAME" "origin/$BASE_BRANCH"
```

### 3. 创建 worktree

将 worktrees 放置为仓库的兄弟目录 — 而不是在其内部。这避免了 git 嵌套仓库问题，并保持工作树干净。

```bash
WORKTREE_PATH="../${REPO_NAME}-wt/${BRANCH_NAME}"
mkdir -p "$(dirname "$WORKTREE_PATH")"
git worktree add "$WORKTREE_PATH" "$BRANCH_NAME"
```

### 4. 设置工作上下文

后续所有工作都在 worktree 内部进行。如果需要，安装依赖项：

```bash
cd "$WORKTREE_PATH"
# 如果是 bun 项目:
[ -f "bun.lock" ] && bun install
```

</setup>

---

## 第 1 阶段：实现

在 worktree 内部进行实际的实现工作。使用此技能的智能体直接执行工作 — 实现本身不需要子智能体委派。

**范围纪律**：对于错误修复，保持最小化。修复错误，为其添加测试，完成。不要重构周围的代码，不要添加配置选项，也不要“改进”没有损坏的东西。验证循环会捕捉到回归问题 — 相信这个过程。

<implementation>

### 提交策略

使用 git-master 技能的原子提交原则。原子提交的原因：如果 CI 在某个更改上失败，你可以隔离并修复它，而无需撤销所有内容。

```
更改 3 个以上文件  → 至少 2 个以上提交
更改 5 个以上文件  → 至少 3 个以上提交
更改 10 个以上文件 → 至少 5 个以上提交
```

每个提交都应该将实现与其测试配对。在提交时加载 `git-master` 技能：

```
task(category="quick", load_skills=["git-master"], prompt="遵循 git-master 约定原子化地提交更改。仓库位于 {WORKTREE_PATH}。")
```

### 推送前的本地验证

在推送之前，运行与 CI 相同的检查。在本地捕捉失败可以节省完整的 CI 往返时间（约 3-5 分钟）：

```bash
bun run typecheck
bun test
bun run build
```

在推送之前修复任何失败。每个修复-提交循环都应该是原子的。

</implementation>

---

## 第 2 阶段：创建 PR

<pr_creation>

### 推送并创建 PR

```bash
git push -u origin "$BRANCH_NAME"
```

使用项目的模板结构创建 PR：

```bash
gh pr create \
  --base "$BASE_BRANCH" \
  --head "$BRANCH_NAME" \
  --title "$PR_TITLE" \
  --body "$(cat <<'EOF'
## 摘要
[1-3 句话描述此 PR 的作用及原因]

## 更改
[主要更改的要点列表]

## 测试
- `bun run typecheck` ✅
- `bun test` ✅
- `bun run build` ✅

## 相关问题
[如果适用，链接到相关 issue]
EOF
)"
```

捕获 PR 编号：

```bash
PR_NUMBER=$(gh pr view --json number -q .number)
```

</pr_creation>

---

## 第 3 阶段：验证循环

这是此技能的核心。PR 准备就绪前，必须全部通过三个关卡。此循环没有迭代上限 — 一直进行直到完成。关卡的排序是有意设计的：CI 最便宜/最快，review-work 最彻底，Cubic 是外部的异步工具。

<verify_loop>

```
while true:
  1. 等待 CI          → 关卡 A
  2. 如果 CI 失败      → 阅读日志，修复，提交，推送，继续
  3. 运行 review-work → 关卡 B
  4. 如果审查失败      → 修复阻碍问题，提交，推送，继续
  5. 检查 Cubic       → 关卡 C
  6. 如果 Cubic 有问题 → 修复问题，提交，推送，继续
  7. 全部通过         → 退出循环
```

### 关卡 A: CI 检查

CI 是最快的反馈循环。等待其完成，然后解析结果。

```bash
# 等待检查开始 (推送后 GitHub 需要片刻时间)
# 然后监视完成情况
gh pr checks "$PR_NUMBER" --watch --fail-fast
```

**失败时**：获取失败运行的日志以了解什么被破坏了：

```bash
# 找到失败的运行
RUN_ID=$(gh run list --branch "$BRANCH_NAME" --status failure --json databaseId --jq '.[0].databaseId')

# 获取失败作业的日志
gh run view "$RUN_ID" --log-failed
```

阅读日志，修复问题，原子提交，推送，然后重新进入循环。

### 关卡 B: review-work

review-work 技能启动 5 个并行的子智能体（目标验证、QA、代码质量、安全性、上下文挖掘）。所有 5 个都必须通过。

在 CI 通过后调用 review-work — 审查无法构建的代码是没有意义的：

```
task(
  category="unspecified-high",
  load_skills=["review-work"],
  run_in_background=false,
  description="对 PR 更改的实现后审查",
  prompt="审查分支 {BRANCH_NAME} 上的实现工作。worktree 位于 {WORKTREE_PATH}。目标: {ORIGINAL_GOAL}。约束: {CONSTRAINTS}。运行命令: bun run dev (或视情况而定)。"
)
```

**失败时**：review-work 会报告具有特定文件和行号的阻碍问题。修复每个阻碍问题，提交，推送，然后从关卡 A 重新进入循环（由于代码更改，CI 必须重新运行）。

### 关卡 C: Cubic 批准

Cubic (`cubic-dev-ai[bot]`) 是一个自动审查机器人，它在 PR 上发表评论。它**不使用** GitHub 的 APPROVED (已批准) 审查状态 — 而是发布带有问题计数和置信度分数的评论。

**批准信号**：最新的 Cubic 评论包含 `**No issues found**` (未发现问题) 和置信度 `**5/5**`。

**问题信号**：评论列出了文件级别详细信息的问题。

```bash
# 获取最新的 Cubic 审查
CUBIC_REVIEW=$(gh api "repos/${REPO}/pulls/${PR_NUMBER}/reviews" \
  --jq '[.[] | select(.user.login == "cubic-dev-ai[bot]")] | last | .body')

# 检查是否获得批准
if echo "$CUBIC_REVIEW" | grep -q "No issues found"; then
  echo "Cubic: APPROVED"
else
  echo "Cubic: ISSUES FOUND"
  echo "$CUBIC_REVIEW"
fi
```

**出现问题时**：Cubic 的审查正文包含结构化的问题描述。解析它们，确定哪些是有效的（有些可能是误报），修复有效的，提交，推送，从关卡 A 重新进入。

Cubic 审查在 PR 更新时自动触发。推送修复后，等待新的审查出现再进行检查。使用带有条件循环的 `gh api` 轮询：

```bash
# 推送后等待新的 Cubic 审查
PUSH_TIME=$(date -u +%Y-%m-%dT%H:%M:%SZ)
while true; do
  LATEST_REVIEW_TIME=$(gh api "repos/${REPO}/pulls/${PR_NUMBER}/reviews" \
    --jq '[.[] | select(.user.login == "cubic-dev-ai[bot]")] | last | .submitted_at')
  if [[ "$LATEST_REVIEW_TIME" > "$PUSH_TIME" ]]; then
    break
  fi
  # 使用 gh api 调用本身作为延迟机制 — 每次调用大约需要 1-2 秒
  # 若需更长等待，使用: timeout 30 gh pr checks "$PR_NUMBER" --watch 2>/dev/null || true
done
```

### 迭代纪律

循环中的每次迭代：
1. **仅**修复由失败关卡识别出的问题
2. 原子提交（每个提交一个逻辑修复）
3. 推送
4. 从关卡 A 重新进入（代码已更改 → 全面重新验证）

避免在修复迭代期间受到诱惑去“改进”不相关的代码。修复循环中的范围蔓延会使调试变得更加困难，并可能引入新的失败。

</verify_loop>

---

## 第 4 阶段：合并与清理

一旦三个关卡全部通过：

<merge_cleanup>

### 合并 PR

```bash
# 压缩合并以保持历史记录干净
gh pr merge "$PR_NUMBER" --squash --delete-branch
```

### 将 .sisyphus 状态同步回主仓库

在删除 worktree 之前，将 `.sisyphus/` 状态复制回去。当 `.sisyphus/` 被 gitignore 忽略时，在 worktree 执行期间写入那里的文件不会被提交或合并 — 它们会在 worktree 被移除时丢失。

```bash
# 将 .sisyphus 状态从 worktree 同步到主仓库（保留任务状态、计划、记事本）
if [ -d "$WORKTREE_PATH/.sisyphus" ]; then
  mkdir -p "$ORIGINAL_DIR/.sisyphus"
  cp -r "$WORKTREE_PATH/.sisyphus/"* "$ORIGINAL_DIR/.sisyphus/" 2>/dev/null || true
fi
```

### 清理 worktree

worktree 已完成其使命 — 将其移除以避免磁盘膨胀：

```bash
cd "$ORIGINAL_DIR"  # 返回原始工作目录
git worktree remove "$WORKTREE_PATH"
# 清理任何陈旧的 worktree 引用
git worktree prune
```

### 报告完成情况

总结发生的事情：

```
## PR 已合并 ✅

- **PR**: #{PR_NUMBER} — {PR_TITLE}
- **分支**: {BRANCH_NAME} → {BASE_BRANCH}
- **迭代**: {N} 次验证循环
- **通过的关卡**: CI ✅ | review-work ✅ | Cubic ✅
- **Worktree**: 已清理
```

</merge_cleanup>

---

## 失败恢复

<failure_recovery>

如果遇到无法恢复的错误（例如，与基础分支合并冲突，基础设施故障）：

1. **不要删除 worktree** — 用户可能希望检查或手动继续
2. 报告发生了什么，尝试了什么，以及目前的情况
3. 包含 worktree 路径，以便用户可以恢复

对于合并冲突：

```bash
cd "$WORKTREE_PATH"
git fetch origin "$BASE_BRANCH"
git rebase "origin/$BASE_BRANCH"
# 解决冲突，然后继续循环
```

</failure_recovery>

---

## 反模式

| 违规 | 失败原因 | 严重程度 |
|-----------|-------------|----------|
| 在主工作树中工作而不是隔离的工作树 | 污染用户的工作目录，可能破坏未提交的工作 | CRITICAL |
| 直接推送到 dev/master | 完全绕过审查 | CRITICAL |
| 在代码更改后跳过 CI 关卡 | review-work 和 Cubic 可能会在陈旧代码上通过 | CRITICAL |
| 在验证循环期间修复不相关的代码 | 范围蔓延导致新的失败 | HIGH |
| 失败时删除 worktree | 用户失去检查/恢复的能力 | HIGH |
| 没有理由地忽略 Cubic 的误报 | 应该评估 Cubic 的问题，而不是盲目驳回 | MEDIUM |
| 巨大的单一提交 | 难以隔离失败，违反 git-master 原则 | MEDIUM |
| 推送前未运行本地检查 | 在明显的失败上浪费 CI 时间 | MEDIUM |
