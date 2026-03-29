---
description: 比较 HEAD 与最新发布的 npm 版本，并列出所有未发布的更改
---

<command-instruction>
立即输出分析结果。不要提问。不要写前言。

## 关键要求：不要仅仅复制提交信息！

对于每个提交，你必须：
1. 阅读实际的 diff 以了解发生了什么更改
2. 用通俗的语言描述真实的更改
3. 解释为什么这个更改很重要（如果不明显的话）

## 步骤：
1. 运行 `git diff v{published-version}..HEAD` 查看实际的更改
2. 按类型（feat/fix/refactor/docs）分组，并附上真实的描述
3. 注意是否有破坏性更改（如果有）
4. 推荐版本升级类型（major/minor/patch）

## 输出格式：
- feat: "添加了做 Y 的 X"（不要仅仅写 "添加 X 功能"）
- fix: "修复了发生 X 的错误，现在 Y"（不要仅仅写 "修复 X 错误"）
- refactor: "将 X 从 A 更改为 B，现在支持 C"（不要仅仅写 "重命名 X"）
</command-instruction>

<version-context>
<published-version>
!`npm view oh-my-opencode version 2>/dev/null || echo "not published"`
</published-version>
<local-version>
!`node -p "require('./package.json').version" 2>/dev/null || echo "unknown"`
</local-version>
<latest-tag>
!`git tag --sort=-v:refname | head -1 2>/dev/null || echo "no tags"`
</latest-tag>
</version-context>

<git-context>
<commits-since-release>
!`npm view oh-my-opencode version 2>/dev/null | xargs -I{} git log "v{}"..HEAD --oneline 2>/dev/null || echo "no commits since release"`
</commits-since-release>
<diff-stat>
!`npm view oh-my-opencode version 2>/dev/null | xargs -I{} git diff "v{}"..HEAD --stat 2>/dev/null || echo "no diff available"`
</diff-stat>
<files-changed-summary>
!`npm view oh-my-opencode version 2>/dev/null | xargs -I{} git diff "v{}"..HEAD --stat 2>/dev/null | tail -1 || echo ""`
</files-changed-summary>
</git-context>

<output-format>
## 未发布更改 (v{published} → HEAD)

### feat
| 作用域 | 更改内容 |
|-------|--------------|
| X | 实际更改的描述 |

### fix
| 作用域 | 更改内容 |
|-------|--------------|
| X | 实际更改的描述 |

### refactor
| 作用域 | 更改内容 |
|-------|--------------|
| X | 实际更改的描述 |

### docs
| 作用域 | 更改内容 |
|-------|--------------|
| X | 实际更改的描述 |

### 破坏性更改
无 或 列表

### 更改的文件
{diff-stat}

### 建议的版本升级
- **建议**: patch|minor|major
- **原因**: 建议的理由
</output-format>

<oracle-safety-review>
## Oracle 部署安全审查（仅当用户明确请求时）

**触发关键字**: "safe to deploy"（可以部署吗）, "can I deploy"（我可以部署吗）, "is it safe"（安全吗）, "review"（审查）, "check"（检查）, "oracle"

当用户请求中包含以上任何关键字时：

### 1. 预验证
```bash
bun run typecheck
bun test
```
- 失败时 → 立即报告 "❌ 无法部署"，不要调用 Oracle

### 2. 调用 Oracle 提示词

收集以下信息并传递给 Oracle：

```
## 部署安全审查请求

### 更改摘要
{上方分析的更改表}

### 关键 diff（按功能组织）
{每个 feat/fix/refactor 的核心代码更改 - 仅关键部分，而非完整 diff}

### 验证结果
- 类型检查: ✅/❌
- 测试: {pass}/{total} (✅/❌)

### 审查项目
1. **回归风险**: 是否有可能会影响现有功能的更改？
2. **副作用**: 是否有可能会发生意外副作用的地方？
3. **破坏性更改**: 是否有影响外部用户的更改？
4. **边缘情况**: 是否有遗漏的边缘情况？
5. **部署建议**: SAFE（安全） / CAUTION（需谨慎） / UNSAFE（不安全）

### 请求
请深入分析上述更改并提供你对部署安全性的判断。
如果存在风险，请结合具体场景进行解释。
如果可以，建议部署后需要监控的关键字。
```

### 3. Oracle 响应后的输出格式

## 🔍 Oracle 部署安全审查结果

### 裁决: ✅ SAFE（安全） / ⚠️ CAUTION（需谨慎） / ❌ UNSAFE（不安全）

### 风险分析
| 领域 | 风险等级 | 描述 |
|------|------------|-------------|
| ... | 🟢/🟡/🔴 | ... |

### 建议
- ...

### 部署后监控关键字
- ...

### 结论
{Oracle 的最终判断}
</oracle-safety-review>
