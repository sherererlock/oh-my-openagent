---
description: 通过 GitHub Actions 工作流将 oh-my-opencode 发布到 npm
argument-hint: <patch|minor|major>
---

<command-instruction>
你是 oh-my-opencode 的发布经理。从头到尾执行完整的发布工作流。

## 关键：参数要求

**你必须从用户那里收到版本升级类型。** 有效选项：
- `patch`: 错误修复，向后兼容 (1.1.7 → 1.1.8)
- `minor`: 新功能，向后兼容 (1.1.7 → 1.2.0)
- `major`: 破坏性更改 (1.1.7 → 2.0.0)

**如果用户未提供升级类型参数，请立即停止并询问：**
> "要继续部署，请指定版本升级类型：`patch`、`minor` 或 `major`"

**没有用户明确确认升级类型，切勿继续。**

---

## 第 0 步：注册待办事项列表（强制性的首个动作）

**在做任何其他事情之前**，使用 TodoWrite 创建详细的待办事项列表：

```
[
  { "id": "confirm-bump", "content": "与用户确认版本升级类型 (patch/minor/major)", "status": "in_progress", "priority": "high" },
  { "id": "check-uncommitted", "content": "检查未提交的更改并在需要时提交", "status": "pending", "priority": "high" },
  { "id": "sync-remote", "content": "与远程同步 (如果有未推送的提交则 pull --rebase && push)", "status": "pending", "priority": "high" },
  { "id": "run-workflow", "content": "触发 GitHub Actions 发布工作流", "status": "pending", "priority": "high" },
  { "id": "wait-workflow", "content": "等待工作流完成 (每 30 秒轮询一次)", "status": "pending", "priority": "high" },
  { "id": "verify-and-preview", "content": "验证是否创建了发布 + 预览自动生成的变更日志和贡献者致谢", "status": "pending", "priority": "high" },
  { "id": "draft-summary", "content": "起草增强版发布摘要 (minor/major 必填，patch 可选 — 询问用户)", "status": "pending", "priority": "high" },
  { "id": "apply-summary", "content": "将增强版摘要添加到发布说明顶部 (如果用户选择加入)", "status": "pending", "priority": "high" },
  { "id": "verify-npm", "content": "验证 npm 包是否成功发布", "status": "pending", "priority": "high" },
  { "id": "wait-platform-workflow", "content": "等待 publish-platform 工作流完成", "status": "pending", "priority": "high" },
  { "id": "verify-platform-binaries", "content": "验证所有 7 个平台二进制包是否已发布", "status": "pending", "priority": "high" },
  { "id": "final-confirmation", "content": "向用户提供最终确认和链接", "status": "pending", "priority": "low" }
]
```

**开始时将每个待办事项标记为 `in_progress`，完成时标记为 `completed`。一次处理一个。**

---

## 第 1 步：确认升级类型

如果作为参数提供了升级类型，请与用户确认：
> "版本升级类型：`{bump}`。继续吗？(y/n)"

等待用户确认后再继续。

---

## 第 2 步：检查未提交的更改

运行：`git status --porcelain`

- 如果有未提交的更改，警告用户并询问是否要先提交
- 如果干净，继续

---

## 第 2.5 步：与远程同步（强制）

检查是否有未推送的提交：
```bash
git log origin/master..HEAD --oneline
```

**如果有未推送的提交，你必须在触发工作流之前同步：**
```bash
git pull --rebase && git push
```

这确保了 GitHub Actions 工作流运行在包含所有本地提交的最新代码上。

---

## 第 3 步：触发 GITHUB ACTIONS 工作流

运行发布工作流：
```bash
gh workflow run publish -f bump={bump_type}
```

等待 3 秒，然后获取运行 ID：
```bash
gh run list --workflow=publish --limit=1 --json databaseId,status --jq '.[0]'
```

---

## 第 4 步：等待工作流完成

每 30 秒轮询一次工作流状态，直到完成：
```bash
gh run view {run_id} --json status,conclusion --jq '{status: .status, conclusion: .conclusion}'
```

状态流转：`queued` → `in_progress` → `completed`

**重要：使用轮询循环，不要使用 sleep 命令。**

如果结论是 `failure`（失败），显示错误并停止：
```bash
gh run view {run_id} --log-failed
```

---

## 第 5 步：验证发布并预览自动生成的内容

两个目标：确认发布存在，然后向用户展示工作流已经生成了什么。

```bash
# 拉取最新代码（工作流提交的版本升级）
git pull --rebase
NEW_VERSION=$(node -p "require('./package.json').version")

# 验证发布是否存在于 GitHub 上
gh release view "v${NEW_VERSION}" --json tagName,url --jq '{tag: .tagName, url: .url}'
```

**验证后，生成自动生成内容的本地预览：**

```bash
bun run script/generate-changelog.ts
```

<agent-instruction>
运行预览后，向用户展示输出并说：

> **以下内容已自动包含在发布中：**
> - 提交变更日志（按 feat/fix/refactor 分组）
> - 贡献者致谢信息（针对非团队贡献者）
>
> 你**不需要**编写这些内容。它已经处理好了。
>
> **对于 patch 发布**，这通常已经足够了。但是，如果有值得强调的显著错误修复或更改，可以添加一个增强版摘要。
> **对于 minor/major 发布**，**必须**添加增强版摘要 — 我将在下一步中起草一个。

等待用户确认后再继续。
</agent-instruction>

---

## 第 6 步：起草增强版发布摘要

<decision-gate>

| 发布类型 | 动作 |
|-------------|--------|
| **patch** | 询问用户："你想让我起草一个突出关键错误修复/更改的增强版摘要吗？还是自动生成的变更日志就足够了？" 如果用户拒绝 → 跳到第 8 步。如果用户接受 → 在下方起草一个简明的错误修复/更改摘要。 |
| **minor** | 强制。起草一个简明的功能摘要。没有它不要继续。 |
| **major** | 强制。起草完整的发布说明，如果适用的话包括迁移说明。没有它不要继续。 |

</decision-gate>

### 你要写什么（以及不写什么）

你正在编写**标题层** — 位于自动生成的提交日志之上的产品公告。把它当成“发布博客文章”，而不是“git 日志”。

<rules>
- 切勿重复提交信息。自动生成的部分已经列出了每个提交。
- 切勿写诸如“各种错误修复和改进”或“几项增强”之类的通用废话。
- 始终关注用户影响：用户现在能做什么以前不能做的事情？
- 始终按主题或能力分组，而不是按提交类型（feat/fix/refactor）分组。
- 始终使用具体的语言：“你现在可以做 X”，而不是“添加了 X 功能”。
</rules>

<examples>
<bad title="复述提交 — 不要这样做">
## 最新动态
- feat(auth): 添加 JWT 刷新令牌轮换
- fix(auth): 处理过期的令牌边缘情况
- refactor(auth): 提取中间件
</bad>

<good title="基于用户影响的叙述 — 这样做">
## 🔐 更智能的身份验证

令牌刷新现在是自动且无缝的。会话不再会在任务中途过期 — 系统会在后台静默轮换凭据。如果你曾对随机注销感到沮丧，此版本修复了这个问题。
</good>

<bad title="模糊的废话 — 不要这样做">
## 改进
- 各种性能改进
- 错误修复和稳定性增强
</bad>

<good title="具体且可衡量 — 这样做">
## ⚡ 规则解析速度提升 3 倍

规则现在按文件修改时间进行缓存。如果你的项目有 50 个以上的规则文件，你会注意到启动速度明显变快 — 我们在测试套件中测量到 3 倍的改进。
</good>
</examples>

### 起草流程

1. **分析** 第 5 步预览中的提交列表。识别对用户重要的 2-5 个主题。
2. **写入** 将摘要写入 `/tmp/release-summary-v${NEW_VERSION}.md`。
3. **展示** 将草稿展示给用户，以供审查和批准后再应用。

```bash
# 在此处写下你的草稿
cat > /tmp/release-summary-v${NEW_VERSION}.md << 'SUMMARY_EOF'
{你的增强版摘要}
SUMMARY_EOF

cat /tmp/release-summary-v${NEW_VERSION}.md
```

<agent-instruction>
起草后，询问用户：
> "这是我起草的发布摘要。它将出现在发布说明的**顶部**，在自动生成的提交变更日志和贡献者致谢之上。你希望在应用之前我调整什么吗？"

没有用户确认，不要继续到第 7 步。
</agent-instruction>

---

## 第 7 步：将增强版摘要应用到发布

**仅当用户在第 6 步选择退出增强版摘要时才跳过此步骤** — 直接进入第 8 步。

<architecture>
最终的发布说明结构：

```
┌─────────────────────────────────────┐
│  增强版摘要 (来自第 6 步)             │  ← 你写的
│  - 基于主题、关注用户影响              │
├─────────────────────────────────────┤
│  ---  (分隔符)                        │
├─────────────────────────────────────┤
│  自动生成的提交变更日志                 │  ← 工作流写的
│  - feat/fix/refactor 分组             │
│  - 贡献者致谢信息                      │
└─────────────────────────────────────┘
```
</architecture>

<zero-content-loss-policy>
- 首先获取现有的发布正文
- 将你的摘要前置于其上
- 现有的自动生成内容必须保持 100% 完整
- 不得删除或修改现有内容的任何一个字符
</zero-content-loss-policy>

```bash
# 1. 获取现有的自动生成正文
EXISTING_BODY=$(gh release view "v${NEW_VERSION}" --json body --jq '.body')

# 2. 组合：增强版摘要在顶部，自动生成内容在底部
{
  cat /tmp/release-summary-v${NEW_VERSION}.md
  echo ""
  echo "---"
  echo ""
  echo "$EXISTING_BODY"
} > /tmp/final-release-v${NEW_VERSION}.md

# 3. 更新发布（仅限追加）
gh release edit "v${NEW_VERSION}" --notes-file /tmp/final-release-v${NEW_VERSION}.md

# 4. 确认
echo "✅ 发布 v${NEW_VERSION} 已使用增强版摘要更新。"
gh release view "v${NEW_VERSION}" --json url --jq '.url'
```

---

## 第 8 步：验证 NPM 发布

轮询 npm 注册表，直到新版本出现：
```bash
npm view oh-my-opencode version
```

与预期版本进行比较。如果 2 分钟后不匹配，警告用户 npm 传播延迟。

---

## 第 8.5 步：等待平台工作流完成

主发布工作流会触发一个单独的 `publish-platform` 工作流，用于构建平台特定的二进制文件。

1. 查找由主工作流触发的 publish-platform 工作流运行：
```bash
gh run list --workflow=publish-platform --limit=1 --json databaseId,status,conclusion --jq '.[0]'
```

2. 每 30 秒轮询一次工作流状态，直到完成：
```bash
gh run view {platform_run_id} --json status,conclusion --jq '{status: .status, conclusion: .conclusion}'
```

**重要：使用轮询循环，不要使用 sleep 命令。**

如果结论是 `failure`，显示错误日志：
```bash
gh run view {platform_run_id} --log-failed
```

---

## 第 8.6 步：验证平台二进制包

在 publish-platform 工作流完成后，验证所有 7 个平台包是否都已发布：

```bash
PLATFORMS="darwin-arm64 darwin-x64 linux-x64 linux-arm64 linux-x64-musl linux-arm64-musl windows-x64"
for PLATFORM in $PLATFORMS; do
  npm view "oh-my-opencode-${PLATFORM}" version
done
```

所有 7 个包应该显示与主包相同的版本 (`${NEW_VERSION}`)。

**预期包：**
| 包名 | 描述 |
|---------|-------------|
| `oh-my-opencode-darwin-arm64` | macOS Apple Silicon |
| `oh-my-opencode-darwin-x64` | macOS Intel |
| `oh-my-opencode-linux-x64` | Linux x64 (glibc) |
| `oh-my-opencode-linux-arm64` | Linux ARM64 (glibc) |
| `oh-my-opencode-linux-x64-musl` | Linux x64 (musl/Alpine) |
| `oh-my-opencode-linux-arm64-musl` | Linux ARM64 (musl/Alpine) |
| `oh-my-opencode-windows-x64` | Windows x64 |

如果任何平台包的版本不匹配，警告用户并建议检查 publish-platform 工作流日志。

---

## 第 9 步：最终确认

向用户报告成功：
- 新版本号
- GitHub 发布 URL: https://github.com/code-yeongyu/oh-my-opencode/releases/tag/v{version}
- npm 包 URL: https://www.npmjs.com/package/oh-my-opencode
- 平台包状态：列出所有 7 个平台包及其版本

---

## 错误处理

- **工作流失败**: 显示失败日志，建议检查 Actions 选项卡
- **未找到发布**: 等待并重试，可能是传播延迟
- **npm 未更新**: npm 可能需要 1-5 分钟来传播，告知用户
- **权限被拒绝**: 用户可能需要使用 `gh auth login` 重新验证
- **平台工作流失败**: 显示来自 publish-platform 工作流的日志，检查哪个平台失败了
- **缺少平台包**: 某些平台可能由于交叉编译问题而失败，建议手动重新运行 publish-platform 工作流

## 语言

使用英语响应用户。

</command-instruction>

<current-context>
<published-version>
!`npm view oh-my-opencode version 2>/dev/null || echo "not published"`
</published-version>
<local-version>
!`node -p "require('./package.json').version" 2>/dev/null || echo "unknown"`
</local-version>
<git-status>
!`git status --porcelain`
</git-status>
<recent-commits>
!`npm view oh-my-opencode version 2>/dev/null | xargs -I{} git log "v{}"..HEAD --oneline 2>/dev/null | head -15 || echo "no commits"`
</recent-commits>
</current-context>
