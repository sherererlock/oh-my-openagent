# Oh-My-OpenAgent 功能参考

## 代理

Oh-My-OpenAgent 提供了 11 个专业的 AI 代理。每个代理都有其独特的专业知识、优化的模型和工具权限。

### 核心代理

核心代理的标签页循环通过注入的运行时 order 字段是确定性的。固定的优先级顺序是 Sisyphus（顺序：1），Hephaestus（顺序：2），Prometheus（顺序：3）和 Atlas（顺序：4）。其余的代理跟随在那个稳定的核心排序之后。

| 代理                  | 模型               | 目的                                                                                                                                                                                                                                                                                                                                                             |
| --------------------- | ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Sisyphus**          | `claude-opus-4-6`  | 默认的编排器。计划、委托并使用专业的子代理通过激进的并行执行来执行复杂的任务。具有扩展思考（32k 预算）的待办事项驱动工作流。回退：`glm-5` → `big-pickle`。                                                                                                                                 |
| **Hephaestus**        | `gpt-5.4`          | 合法的工匠。受 AmpCode 深度模式启发的自主深度工作者。在行动前进行彻底研究的目标导向执行。探索代码库模式，端到端完成任务而不会过早停止。以希腊神话中的锻造和工艺之神命名。需要支持 GPT 的提供商。 |
| **Oracle**            | `gpt-5.4`          | 架构决策、代码审查、调试。具有出色逻辑推理和深度分析的只读咨询。受 AmpCode 启发。回退：`gemini-3.1-pro` → `claude-opus-4-6`。                                                                                                                                                                    |
| **Librarian**         | `minimax-m2.7`     | 多仓库分析、文档查找、OSS 实现示例。通过基于证据的回答深入理解代码库。主要的 OpenCode Go 路径使用 MiniMax M2.7。其他提供商目录可能仍会回退到 MiniMax M2.5，然后是 `claude-haiku-4-5` 和 `gpt-5-nano`。                                                                                  |
| **Explore**           | `grok-code-fast-1` | 快速代码库探索和上下文 grep。主要路径保持在 Grok Code Fast 1。现在在提供商目录公开它的地方使用 MiniMax M2.7，而一些 OpenCode 回退路径仍然使用 MiniMax M2.5 以保持目录兼容性。                                                                                                                          |
| **Multimodal-Looker** | `gpt-5.4`          | 视觉内容专家。分析 PDF、图像、图表以提取信息。回退：`k2p5` → `glm-4.6v` → `gpt-5-nano`。                                                                                                                                                                                                                                  |
### 规划代理

| 代理           | 模型              | 目的                                                                                                                                               |
| -------------- | ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Prometheus** | `claude-opus-4-6` | 带有访谈模式的战略规划师。通过迭代提问创建详细的工作计划。回退：`gpt-5.4` → `gemini-3.1-pro`。          |
| **Metis**      | `claude-opus-4-6` | 计划顾问 — 规划前分析。识别隐藏的意图、歧义和 AI 失败点。回退：`gpt-5.4` → `gemini-3.1-pro`。 |
| **Momus**      | `gpt-5.4`         | 计划审查员 — 根据清晰度、可验证性和完整性标准验证计划。回退：`claude-opus-4-6` → `gemini-3.1-pro`。        |

### 编排代理

| 代理                | 模型                   | 目的                                                                                                                                                                                        |
| ------------------- | ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Atlas**           | `claude-sonnet-4-6`    | 待办事项编排器。系统地执行计划的任务，管理待办事项并协调工作。回退：`gpt-5.4` (medium)。                                                     |
| **Sisyphus-Junior** | _(取决于类别)_ | 类别生成的执行器。根据任务类别（visual-engineering, quick, deep 等）自动选择模型。当主代理通过 `task` 工具委托工作时使用。 |

### 调用代理

主代理会自动调用这些代理，但你也可以显式地调用它们：

```
Ask @oracle to review this design and propose an architecture
Ask @librarian how this is implemented - why does the behavior keep changing?
Ask @explore for the policy on this feature
```

### 工具限制

| 代理              | 限制                                                                                    |
| ----------------- | --------------------------------------------------------------------------------------- |
| oracle            | 只读：不能写入、编辑或委托（已阻止：write, edit, task, call_omo_agent） |
| librarian         | 不能写入、编辑或委托（已阻止：write, edit, task, call_omo_agent）            |
| explore           | 不能写入、编辑或委托（已阻止：write, edit, task, call_omo_agent）            |
| multimodal-looker | 允许列表：仅限 `read`                                                                  |
| atlas             | 不能委托（已阻止：task, call_omo_agent）                                         |
| momus             | 不能写入、编辑或委托（已阻止：write, edit, task）                            |

### 后台代理

在后台运行代理并继续工作：

- 让 GPT 调试，同时 Claude 尝试不同的方法
- Gemini 编写前端，同时 Claude 处理后端
- 启动大规模的并行搜索，继续实施，准备好后使用结果

```
# 在后台启动
task(subagent_type="explore", load_skills=[], prompt="查找认证实现", run_in_background=true)

# 继续工作...
# 完成时系统会通知

# 需要时检索结果
background_output(task_id="bg_abc123")
```

#### 使用 Tmux 的可视化多代理

启用 `tmux.enabled` 以在单独的 tmux 窗格中查看后台代理：

```json
{
  "tmux": {
    "enabled": true,
    "layout": "main-vertical"
  }
}
```

在 tmux 中运行时：

- 后台代理在新的窗格中生成
- 实时观察多个代理工作
- 每个窗格实时显示代理输出
- 代理完成时自动清理
- **稳定的代理排序**：核心代理的标签页循环通过注入的运行时 order 字段是确定性的（Sisyphus: 1, Hephaestus: 2, Prometheus: 3, Atlas: 4）

在 `oh-my-opencode.jsonc` 中自定义代理模型、提示词和权限。

## 类别系统

类别是针对特定领域优化的代理配置预设。与其将所有内容委托给单个 AI 代理，不如调用针对任务性质定制的专家要高效得多。

### 什么是类别以及它们为什么重要

- **类别 (Category)**：“这是一种什么样的工作？”（决定模型、温度、提示心态）
- **技能 (Skill)**：“需要什么工具和知识？”（为特定领域注入专门知识、MCP 工具、工作流）

通过组合这两个概念，你可以通过 `task` 生成最佳的代理。

### 内置类别

| 类别                 | 默认模型                        | 用例                                                                                                                        |
| -------------------- | ------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| `visual-engineering` | `google/gemini-3.1-pro`         | 前端、UI/UX、设计、样式、动画                                                                                 |
| `ultrabrain`         | `openai/gpt-5.4` (xhigh)        | 深度逻辑推理、需要广泛分析的复杂架构决策                                         |
| `deep`               | `openai/gpt-5.3-codex` (medium) | 目标导向的自主解决问题。行动前彻底研究。适用于需要深入理解的棘手问题。 |
| `artistry`           | `google/gemini-3.1-pro` (high)  | 高度创意/艺术性的任务，新颖的想法                                                                                 |
| `quick`              | `openai/gpt-5.4-mini`           | 琐碎的任务 - 单文件更改、错别字修复、简单的修改                                                       |
| `unspecified-low`    | `anthropic/claude-sonnet-4-6`   | 不适合其他类别的任务，需要较低的努力                                                                  |
| `unspecified-high`   | `anthropic/claude-opus-4-6` (max) | 不适合其他类别的任务，需要较高的努力                                                               |
| `writing`            | `google/gemini-3-flash`         | 文档、散文、技术写作                                                                                     |

### 用法

在调用 `task` 工具时指定 `category` 参数。

```typescript
task({
  category: "visual-engineering",
  prompt: "向仪表板页面添加一个响应式图表组件",
});
```

### 自定义类别

你可以在你的插件配置文件中定义自定义类别。在重命名过渡期间，`oh-my-openagent.json[c]` 和旧的 `oh-my-opencode.json[c]` 基本名称都会被识别。

#### 类别配置模式

| 字段                | 类型    | 描述                                                                        |
| ------------------- | ------- | --------------------------------------------------------------------------- |
| `description`       | string  | 类别目的的人类可读描述。显示在任务提示中。 |
| `model`             | string  | 要使用的 AI 模型 ID（例如，`anthropic/claude-opus-4-6`）                      |
| `variant`           | string  | 模型变体（例如，`max`, `xhigh`）                                        |
| `temperature`       | number  | 创造力水平 (0.0 ~ 2.0)。越低越具有确定性。                  |
| `top_p`             | number  | Nucleus 采样参数 (0.0 ~ 1.0)                                      |
| `prompt_append`     | string  | 选择此类别时附加到系统提示词的内容           |
| `thinking`          | object  | 思考模型配置 (`{ type: "enabled", budgetTokens: 16000 }`)   |
| `reasoningEffort`   | string  | 推理努力水平 (`low`, `medium`, `high`)                            |
| `textVerbosity`     | string  | 文本冗长度 (`low`, `medium`, `high`)                              |
| `tools`             | object  | 工具使用控制（使用 `{ "tool_name": false }` 禁用）                  |
| `maxTokens`         | number  | 最大响应 token 数                                                |
| `is_unstable_agent` | boolean | 将代理标记为不稳定 - 强制进入后台模式进行监控              |

#### 示例配置

```jsonc
{
  "categories": {
    // 1. 定义新的自定义类别
    "korean-writer": {
      "model": "google/gemini-3-flash",
      "temperature": 0.5,
      "prompt_append": "你是一位韩国技术作家。保持友好和清晰的语气。",
    },

    // 2. 覆盖现有类别（更改模型）
    "visual-engineering": {
      "model": "openai/gpt-5.4",
      "temperature": 0.8,
    },

    // 3. 配置思考模型并限制工具
    "deep-reasoning": {
      "model": "anthropic/claude-opus-4-6",
      "thinking": {
        "type": "enabled",
        "budgetTokens": 32000,
      },
      "tools": {
        "websearch_web_search_exa": false,
      },
    },
  },
}
```

### Sisyphus-Junior 作为委托执行者

当你使用类别时，一个名为 **Sisyphus-Junior** 的特殊代理会执行工作。

- **特点**：不能将任务**重新委托**给其他代理。
- **目的**：防止无限委托循环并确保专注于分配的任务。

## 高级配置

### 重命名兼容性

发布的包和二进制文件仍然是 `oh-my-opencode`。在 `opencode.json` 中，兼容层现在首选插件条目 `oh-my-openagent`，而旧的 `oh-my-opencode` 条目仍然会加载并带有警告。在过渡期间识别插件配置文件（`oh-my-openagent.json[c]` 或旧的 `oh-my-opencode.json[c]`）。运行 `bunx oh-my-opencode doctor` 以检查旧包名警告。

### 回退模型

使用可以混合纯模型字符串和每个模型对象的数组配置每个代理的回退链：

```jsonc
{
  "agents": {
    "sisyphus": {
      "fallback_models": [
        "opencode/glm-5",
        { "model": "openai/gpt-5.4", "variant": "high" },
        { "model": "anthropic/claude-sonnet-4-6", "thinking": { "type": "enabled", "budgetTokens": 64000 } }
      ]
    }
  }
}
```

当模型出错时，运行时可以在配置的回退数组中移动。对象条目让你调整备份模型本身，而不仅仅是交换模型名称。

### 基于文件的提示词

使用 `prompt` 字段中的 `file://` URL 从外部文件加载代理系统提示词，或使用 `prompt_append` 附加额外内容。`prompt_append` 字段也适用于类别。

```jsonc
{
  "agents": {
    "sisyphus": {
      "prompt": "file:///path/to/custom-prompt.md"
    },
    "oracle": {
      "prompt_append": "file:///path/to/additional-context.md"
    }
  },
  "categories": {
    "deep": {
      "prompt_append": "file:///path/to/deep-category-append.md"
    }
  }
}
```

支持将 `~` 扩展为主目录和相对 `file://` 路径。

用于：
- 将提示词与配置分开进行版本控制
- 在项目之间共享提示词
- 保持配置文件简洁
- 在不重复基本提示词的情况下添加类别特定的上下文

文件内容在运行时加载并注入到代理的系统提示词中。

### 会话恢复

系统自动从常见的会话失败中恢复，无需用户干预：

- **缺失工具结果**：重建可恢复的工具状态，跳过无效的工具部分 ID，而不是使整个恢复通道失败
- **思考块违规**：从 API 思考块不匹配中恢复
- **空消息**：当内容缺失时重建消息历史
- **上下文窗口限制**：通过智能压缩优雅地处理 Claude 上下文窗口超限错误
- **JSON 解析错误**：从格式错误的工具输出中恢复

恢复在代理执行期间透明地发生。你看到的是结果，而不是失败。
## 技能

技能提供带有嵌入式 MCP 服务器和详细说明的专用工作流。技能是一种为代理注入特定领域**专门知识 (Context)** 和**工具 (MCP)** 的机制。

### 内置技能

| 技能               | 触发器                                                  | 描述                                                                                                                                                                                                                                                                                                                                  |
| ------------------ | ------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **git-master**     | commit, rebase, squash, "谁写的", "X 是什么时候添加的" | Git 专家。检测提交样式，拆分原子提交，制定变基策略。三个专长：提交架构师（原子提交、依赖排序、样式检测）、变基外科医生（历史重写、冲突解决、分支清理）、历史考古学家（查找何时/何地引入了特定更改）。 |
| **playwright**     | 浏览器任务、测试、截图                     | 通过 Playwright MCP 的浏览器自动化。**必须用于**浏览器验证、浏览、网页抓取、测试和截图。                                                                                                                                                                                                                   |
| **playwright-cli** | Playwright CLI 上的浏览器任务                         | 通过 Playwright CLI 集成的浏览器自动化。当首选直接 CLI 脚本而不是 MCP 时很有用。                                                                                                                                                                                                                            |
| **agent-browser**  | agent-browser 上的浏览器任务                          | 通过 `agent-browser` CLI 的浏览器自动化。涵盖导航、快照、截图、网络检查和脚本交互。                                                                                                                                                                                                     |
| **dev-browser**    | 有状态的浏览器脚本                              | 具有持久页面状态的浏览器自动化，用于迭代工作流和经过身份验证的会话。                                                                                                                                                                                                                                             |
| **frontend-ui-ux** | UI/UX 任务，样式                                    | 由设计师转型的开发者角色。即使没有设计模型也能制作出令人惊叹的 UI/UX。强调大胆的审美方向、独特的排版、有凝聚力的调色板。                                                                                                                                                                   |

#### git-master 核心原则

**默认多次提交**：

```
3+ 个文件 -> 必须是 2+ 次提交
5+ 个文件 -> 必须是 3+ 次提交
10+ 个文件 -> 必须是 5+ 次提交
```

**自动样式检测**：

- 分析最后 30 次提交的语言（韩语/英语）和样式（语义/普通/简短）
- 自动匹配你仓库的提交约定

**用法**：

```
/git-master 提交这些更改
/git-master 变基到 main
/git-master 谁写了这段认证代码？
```

#### frontend-ui-ux 设计过程

- **设计过程**：目的、基调、约束、差异化
- **审美方向**：选择极致 - 粗野主义、极繁主义、复古未来主义、奢华、俏皮
- **排版**：独特的字体，避免通用（Inter, Roboto, Arial）
- **颜色**：有凝聚力的调色板，带有鲜明的强调色，避免白底紫色的 AI 糟粕
- **运动**：高影响力的交错显示、滚动触发、令人惊讶的悬停状态
- **反模式**：通用的字体、可预测的布局、千篇一律的设计

### 浏览器自动化选项

Oh-My-OpenAgent 提供了两个浏览器自动化提供商，可通过 `browser_automation_engine.provider` 配置。

#### 选项 1：Playwright MCP（默认）

```yaml
mcp:
  playwright:
    command: npx
    args: ["@playwright/mcp@latest"]
```

**用法**：

```
/playwright 导航到 example.com 并截图
```

#### 选项 2：Agent Browser CLI (Vercel)

```json
{
  "browser_automation_engine": {
    "provider": "agent-browser"
  }
}
```

**需要安装**：

```bash
bun add -g agent-browser
```

**用法**：

```
使用 agent-browser 导航到 example.com 并提取主标题
```

**功能（两个提供商）**：

- 导航并与网页交互
- 获取截图和 PDF
- 填写表单并点击元素
- 等待网络请求
- 抓取内容

### 自定义技能创建 (SKILL.md)

你可以将自定义技能直接添加到项目根目录中的 `.opencode/skills/` 或主目录中的 `~/.claude/skills/`。

**示例：`.opencode/skills/my-skill/SKILL.md`**

```markdown
---
name: my-skill
description: 我的特殊自定义技能
mcp:
  my-mcp:
    command: npx
    args: ["-y", "my-mcp-server"]
---

# 我的技能提示词

此内容将被注入到代理的系统提示词中。
...
```

**技能加载位置**（按优先级顺序，从高到低）：

- `.opencode/skills/*/SKILL.md` (项目, OpenCode 原生)
- `~/.config/opencode/skills/*/SKILL.md` (用户, OpenCode 原生)
- `.claude/skills/*/SKILL.md` (项目, Claude Code 兼容)
- `.agents/skills/*/SKILL.md` (项目, Agents 约定)
- `~/.agents/skills/*/SKILL.md` (用户, Agents 约定)

较高优先级的同名技能会覆盖较低优先级的。

在配置中通过 `disabled_skills: ["playwright"]` 禁用内置技能。

### 类别 + 技能组合策略

通过组合类别和技能，你可以创建强大的专业代理。

#### 设计师（UI 实现）

- **类别**: `visual-engineering`
- **load_skills**: `["frontend-ui-ux", "playwright"]`
- **效果**: 实现美观的 UI 并直接在浏览器中验证渲染结果。

#### 架构师（设计审查）

- **类别**: `ultrabrain`
- **load_skills**: `[]` (纯推理)
- **效果**: 利用 GPT-5.4 xhigh 推理进行深入的系统架构分析。

#### 维护者（快速修复）

- **类别**: `quick`
- **load_skills**: `["git-master"]`
- **效果**: 使用具有成本效益的模型快速修复代码并生成干净的提交。

### task 提示词指南

在委托时，**清晰和具体**的提示词至关重要。包含这 7 个要素：

1. **任务 (TASK)**：需要做什么？（单一目标）
2. **预期结果 (EXPECTED OUTCOME)**：交付物是什么？
3. **所需技能 (REQUIRED SKILLS)**：应该通过 `load_skills` 加载哪些技能？
4. **所需工具 (REQUIRED TOOLS)**：必须使用哪些工具？（白名单）
5. **必须做 (MUST DO)**：必须做什么（约束）
6. **绝不能做 (MUST NOT DO)**：绝对不能做什么
7. **上下文 (CONTEXT)**：文件路径、现有模式、参考材料

**坏示例**：

> "修复这个"

**好示例**：

> **任务**：修复 `LoginButton.tsx` 中移动端布局崩溃的问题
> **上下文**：`src/components/LoginButton.tsx`，使用 Tailwind CSS
> **必须做**：在 `md:` 断点处更改 flex-direction
> **绝不能做**：修改现有的桌面布局
> **预期**：按钮在移动设备上垂直对齐

## 命令

命令是斜杠触发的工作流，执行预定义的模板。

### 内置命令

| 命令                 | 描述                                                                                       |
| -------------------- | ------------------------------------------------------------------------------------------ |
| `/init-deep`         | 初始化分层的 AGENTS.md 知识库                                           |
| `/ralph-loop`        | 启动自引用开发循环，直到完成                                   |
| `/ulw-loop`          | 启动 ultrawork 循环 - 以 ultrawork 模式继续                                       |
| `/cancel-ralph`      | 取消活动的 Ralph 循环                                                                   |
| `/refactor`          | 具有 LSP、AST-grep、架构分析和 TDD 验证的智能重构    |
| `/start-work`        | 从 Prometheus 计划开始 Sisyphus 工作会话                                           |
| `/stop-continuation` | 停止此会话的所有继续机制（ralph loop, todo continuation, boulder） |
| `/handoff`           | 创建详细的上下文摘要，以便在新会话中继续工作                     |

### /init-deep

**目的**：在整个项目中生成分层的 AGENTS.md 文件

**用法**：

```
/init-deep [--create-new] [--max-depth=N]
```

创建代理自动读取的特定于目录的上下文文件：

```
project/
├── AGENTS.md              # 项目级上下文
├── src/
│   ├── AGENTS.md          # src 级上下文
│   └── components/
│       └── AGENTS.md      # 组件级上下文
```

### /ralph-loop

**目的**：自引用开发循环，运行直到任务完成

**命名来源**：Anthropic 的 Ralph Wiggum 插件

**用法**：

```
/ralph-loop "构建一个带有身份验证的 REST API"
/ralph-loop "重构支付模块" --max-iterations=50
```

**行为**：

- 代理持续朝着目标努力
- 检测 `<promise>DONE</promise>` 以知道何时完成
- 如果代理在未完成的情况下停止，则自动继续
- 在以下情况下结束：检测到完成、达到最大迭代次数（默认 100）或 `/cancel-ralph`

**配置**：`{ "ralph_loop": { "enabled": true, "default_max_iterations": 100 } }`

### /ulw-loop

**目的**：与 ralph-loop 相同，但激活了 ultrawork 模式

一切都在最大强度下运行 - 并行代理、后台任务、激进探索。

### /refactor

**目的**：使用完整工具链的智能重构

**用法**：

```
/refactor <target> [--scope=<file|module|project>] [--strategy=<safe|aggressive>]
```

**功能**：

- 由 LSP 驱动的重命名和导航
- 用于模式匹配的 AST-grep
- 更改前的架构分析
- 更改后的 TDD 验证
- Codemap 生成

### /start-work

**目的**：从 Prometheus 生成的计划开始执行

**用法**：

```
/start-work [plan-name]
```

使用 atlas 代理系统地执行计划的任务。

### /stop-continuation

**目的**：停止此会话的所有继续机制

停止 ralph loop、todo continuation 和 boulder 状态。当你想让代理停止其当前的多步工作流时使用。

### /handoff

**目的**：创建详细的上下文摘要，以便在新会话中继续工作

生成结构化的交接文档，捕获当前状态、已完成的工作、剩余的工作和相关的文件路径——以便在新的会话中无缝继续。

### 自定义命令

从以下位置加载自定义命令：

- `.opencode/command/*.md` (项目, OpenCode 原生)
- `~/.config/opencode/command/*.md` (用户, OpenCode 原生)
- `.claude/commands/*.md` (项目, Claude Code 兼容)
- `~/.config/opencode/commands/*.md` (用户, Claude Code 兼容)

## 工具

### 代码搜索工具

| 工具     | 描述                                                              |
| -------- | ----------------------------------------------------------------- |
| **grep** | 使用正则表达式进行内容搜索。按文件模式过滤。 |
| **glob** | 快速的文件模式匹配。通过名称模式查找文件。          |

### 编辑工具

| 工具     | 描述                                                                                                                                                       |
| -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **edit** | 基于哈希锚点的编辑工具。使用 `LINE#ID` 格式进行精确、安全的修改。在应用更改之前验证内容哈希——零陈旧行错误。 |

### LSP 工具 (面向代理的 IDE 功能)

| 工具                    | 描述                                        |
| ----------------------- | ------------------------------------------- |
| **lsp_diagnostics**     | 在构建前获取错误/警告            |
| **lsp_prepare_rename**  | 验证重命名操作                   |
| **lsp_rename**          | 在工作区中重命名符号              |
| **lsp_goto_definition** | 跳转到符号定义                   |
| **lsp_find_references** | 在工作区中查找所有用法            |
| **lsp_symbols**         | 获取文件大纲或工作区符号搜索 |

### AST-Grep 工具

| 工具                 | 描述                                         |
| -------------------- | -------------------------------------------- |
| **ast_grep_search**  | 具有 AST 感知的代码模式搜索（25 种语言） |
| **ast_grep_replace** | 具有 AST 感知的代码替换                   |

### 委托工具

| 工具                  | 描述                                                                                                                                                                                                                                    |
| --------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **call_omo_agent**    | 生成 explore/librarian 代理。支持 `run_in_background`。                                                                                                                                                                           |
| **task**              | 基于类别的任务委托。支持内置类别如 `visual-engineering`, `ultrabrain`, `deep`, `artistry`, `quick`, `unspecified-low`, `unspecified-high` 和 `writing`，或通过 `subagent_type` 直接指定代理。 |
| **background_output** | 检索后台任务结果                                                                                                                                                                                                        |
| **background_cancel** | 取消正在运行的后台任务                                                                                                                                                                                                         |

### 视觉分析工具

| 工具        | 描述                                                                                                                                                           |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **look_at** | 通过 Multimodal-Looker 代理分析媒体文件（PDF、图像、图表）。从文档中提取特定信息或摘要，描述视觉内容。 |

### 技能工具

| 工具          | 描述                                                                                                   |
| ------------- | ------------------------------------------------------------------------------------------------------ |
| **skill**     | 按名称加载并执行技能或斜杠命令。返回应用了上下文的详细说明。 |
| **skill_mcp** | 调用技能嵌入的 MCP 中的 MCP 服务器操作。                                                 |

### 会话工具

| 工具               | 描述                                     |
| ------------------ | ---------------------------------------- |
| **session_list**   | 列出所有 OpenCode 会话               |
| **session_read**   | 从会话中读取消息和历史记录 |
| **session_search** | 在会话消息中进行全文搜索 |
| **session_info**   | 获取会话元数据和统计信息      |

### 任务管理工具

需要在配置中设置 `experimental.task_system: true`。

| 工具            | 描述                                     |
| --------------- | ---------------------------------------- |
| **task_create** | 创建带有自动生成的 ID 的新任务 |
| **task_get**    | 通过 ID 检索任务                    |
| **task_list**   | 列出所有活动任务                    |
| **task_update** | 更新现有任务                  |

#### 任务系统详情

**关于 Claude Code 对齐的注意事项**：此实现遵循 Claude Code 内部的 Task 工具签名（`TaskCreate`, `TaskUpdate`, `TaskList`, `TaskGet`）和字段命名约定（`subject`, `blockedBy`, `blocks` 等）。然而，Anthropic 尚未发布这些工具的官方文档。这是 Oh My OpenAgent 基于观察到的 Claude Code 行为和内部规范自己的实现。

**任务架构**：

```ts
interface Task {
  id: string; // T-{uuid}
  subject: string; // 祈使句："运行测试"
  description: string;
  status: "pending" | "in_progress" | "completed" | "deleted";
  activeForm?: string; // 现在进行时："正在运行测试"
  blocks: string[]; // 此任务阻止的任务
  blockedBy: string[]; // 阻止此任务的任务
  owner?: string; // 代理名称
  metadata?: Record<string, unknown>;
  threadID: string; // 会话 ID (自动设置)
}
```

**依赖和并行执行**：

```
[构建前端]    ──┐
                      ├──→ [集成测试] ──→ [部署]
[构建后端]     ──┘
```

- `blockedBy` 为空的任务并行运行
- 依赖任务等待直到阻塞任务完成

**示例工作流**：

```ts
TaskCreate({ subject: "构建前端" }); // T-001
TaskCreate({ subject: "构建后端" }); // T-002
TaskCreate({ subject: "运行集成测试", blockedBy: ["T-001", "T-002"] }); // T-003

TaskList();
// T-001 [pending] 构建前端        blockedBy: []
// T-002 [pending] 构建后端         blockedBy: []
// T-003 [pending] 集成测试     blockedBy: [T-001, T-002]

TaskUpdate({ id: "T-001", status: "completed" });
TaskUpdate({ id: "T-002", status: "completed" });
// T-003 现在解除阻塞
```

**存储**：任务作为 JSON 文件存储在 `.sisyphus/tasks/` 中。

**与 TodoWrite 的区别**：

| 特性               | TodoWrite      | 任务系统                   |
| ------------------ | -------------- | -------------------------- |
| 存储               | 会话内存       | 文件系统                   |
| 持久性             | 关闭时丢失     | 重启后保留                 |
| 依赖               | 无             | 完全支持 (`blockedBy`)     |
| 并行执行           | 手动           | 自动优化                   |

**何时使用**：当工作具有多个带有依赖关系的步骤、多个子代理将协作或进度应跨会话保留时，使用任务系统。

### 交互式终端工具

| 工具                 | 描述                                                                                               |
| -------------------- | -------------------------------------------------------------------------------------------------- |
| **interactive_bash** | 用于 TUI 应用程序 (vim, htop, pudb) 的基于 Tmux 的终端。直接传递 tmux 子命令，无需前缀。 |

**用法示例**：

```bash
# 创建新会话
interactive_bash(tmux_command="new-session -d -s dev-app")

# 发送按键到会话
interactive_bash(tmux_command="send-keys -t dev-app 'vim main.py' Enter")

# 捕获窗格输出
interactive_bash(tmux_command="capture-pane -p -t dev-app")
```

**要点**：

- 命令是 tmux 子命令（无 `tmux` 前缀）
- 用于需要持久会话的交互式应用程序
- 一次性命令应使用带有 `&` 的常规 `Bash` 工具

## 钩子

钩子在代理生命周期的关键点（涵盖完整的会话、消息、工具和参数管道）拦截并修改行为。

### 钩子事件

| 事件            | 何时                          | 可以                                               |
| --------------- | ----------------------------- | -------------------------------------------------- |
| **PreToolUse**  | 工具执行前         | 阻止、修改输入、注入上下文                |
| **PostToolUse** | 工具执行后          | 添加警告、修改输出、注入消息       |
| **Message**     | 消息处理期间     | 转换内容、检测关键字、激活模式 |
| **Event**       | 会话生命周期变化时  | 恢复、回退、通知                  |
| **Transform**   | 上下文转换期间 | 注入上下文、验证块                    |
| **Params**      | 设置 API 参数时   | 调整模型设置、努力水平                |

### 内置钩子

#### 上下文 & 注入

| 钩子                            | 事件                     | 描述                                                                                                                                                                                                      |
| ------------------------------- | ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **directory-agents-injector**   | PreToolUse + PostToolUse | 读取文件时自动注入 AGENTS.md。从文件遍历到项目根目录，收集所有 AGENTS.md 文件。对于 OpenCode 1.1.37+ 已弃用 — 当原生 AGENTS.md 注入可用时自动禁用。 |
| **directory-readme-injector**   | PreToolUse + PostToolUse | 自动注入 README.md 作为目录上下文。                                                                                                                                                             |
| **rules-injector**              | PreToolUse + PostToolUse | 条件匹配时从 `.claude/rules/` 注入规则。支持 globs 和 alwaysApply。                                                                                                                |
| **compaction-context-injector** | Event                    | 在会话压缩期间保留关键上下文。                                                                                                                                                     |
| **context-window-monitor**      | Event                    | 监控上下文窗口使用情况并跟踪 token 消耗。                                                                                                                                               |
| **preemptive-compaction**       | Event                    | 在达到 token 限制之前主动压缩会话。                                                                                                                                                |

#### 生产力 & 控制

| 钩子                        | 事件                | 描述                                                                                                                                                        |
| --------------------------- | ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **keyword-detector**        | Message + Transform | 检测关键字并激活模式：`ultrawork`/`ulw`（最高性能），`search`/`find`（并行探索），`analyze`/`investigate`（深入分析）。 |
| **think-mode**              | Params              | 自动检测扩展思考需求。捕获 "think deeply"、"ultrathink" 并调整模型设置。                                                      |
| **ralph-loop**              | Event + Message     | 管理自引用循环继续。                                                                                                                 |
| **start-work**              | Message             | 处理 /start-work 命令执行。                                                                                                                      |
| **auto-slash-command**      | Message             | 自动从提示词中执行斜杠命令。                                                                                                         |
| **stop-continuation-guard** | Event + Message     | 保护停止继续机制。                                                                                                                     |
| **category-skill-reminder** | Event + PostToolUse | 提醒代理有可用于委托的类别技能。                                                                                              |
| **anthropic-effort**        | Params              | 根据上下文调整 Anthropic API 努力水平。                                                                                                        |

#### 质量 & 安全

| 钩子                            | 事件                     | 描述                                                                                      |
| ------------------------------- | ------------------------ | ----------------------------------------------------------------------------------------- |
| **comment-checker**             | PostToolUse              | 提醒代理减少过多的注释。智能地忽略 BDD、指令、文档字符串。 |
| **thinking-block-validator**    | Transform                | 验证思考块以防止 API 错误。                                          |
| **edit-error-recovery**         | PostToolUse + Event      | 从编辑工具故障中恢复。                                                         |
| **write-existing-file-guard**   | PreToolUse               | 防止在未先读取文件的情况下意外覆盖现有文件。              |
| **hashline-read-enhancer**      | PostToolUse              | 使用哈希锚定行标记增强 read 输出，用于 hashline edit 工具。          |
| **hashline-edit-diff-enhancer** | PreToolUse + PostToolUse | 使用差异标记增强 edit 操作，用于 hashline edit 工具。                    |

#### 恢复 & 稳定性

| 钩子                                        | 事件            | 描述                                                                                                                                                                                                                                                        |
| ------------------------------------------- | --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **session-recovery**                        | Event           | 从会话错误中恢复 — 缺失的工具结果、思考块问题、空消息。                                                                                                                                                                 |
| **anthropic-context-window-limit-recovery** | Event           | 优雅地处理 Claude 上下文窗口限制。                                                                                                                                                                                                            |
| **runtime-fallback**                        | Event + Message | 在可重试的 API 错误（例如 429, 503, 529）、提供商密钥配置错误（例如缺少 API 密钥）和自动重试信号（当 `timeout_seconds > 0` 时）时，自动切换到备份模型。具有按模型冷却时间的可配置重试逻辑。 |
| **model-fallback**                          | Event + Message | 当主模型不可用时管理模型回退链。                                                                                                                                                                                             |
| **json-error-recovery**                     | PostToolUse     | 从工具输出中的 JSON 解析错误中恢复。                                                                                                                                                                                                            |

#### 截断 & 上下文管理

| 钩子                      | 事件        | 描述                                                                                                |
| ------------------------- | ----------- | --------------------------------------------------------------------------------------------------- |
| **tool-output-truncator** | PostToolUse | 截断来自 Grep, Glob, LSP, AST-grep 工具的输出。根据上下文窗口动态调整。 |

#### 通知 & 用户体验

| 钩子                         | 事件                | 描述                                                                                               |
| ---------------------------- | ------------------- | -------------------------------------------------------------------------------------------------- |
| **auto-update-checker**      | Event               | 在会话创建时检查新版本，显示带有版本和 Sisyphus 状态的启动 toast 通知。 |
| **background-notification**  | Event               | 当后台代理任务完成时通知。                                                     |
| **session-notification**     | Event               | 当代理空闲时的操作系统通知。在 macOS、Linux、Windows 上工作。                              |
| **agent-usage-reminder**     | PostToolUse + Event | 提醒你利用专业的代理以获得更好的结果。                                     |
| **question-label-truncator** | PreToolUse          | 截断 Question 工具 UI 中过长的问题标签。                                            |

#### 任务管理

| 钩子                             | 事件                | 描述                                                |
| -------------------------------- | ------------------- | --------------------------------------------------- |
| **task-resume-info**             | PostToolUse         | 提供任务恢复信息以保持连续性。    |
| **delegate-task-retry**          | PostToolUse + Event | 重试失败的任务委托调用。               |
| **empty-task-response-detector** | PostToolUse         | 检测委托任务的空响应。       |
| **tasks-todowrite-disabler**     | PreToolUse          | 当任务系统处于活动状态时禁用 TodoWrite 工具。 |

#### 继续

| 钩子                           | 事件  | 描述                                                       |
| ------------------------------ | ----- | ---------------------------------------------------------- |
| **todo-continuation-enforcer** | Event | 强制完成待办事项 — 将空闲的代理拉回工作岗位。 |
| **compaction-todo-preserver**  | Event | 在会话压缩期间保留待办事项状态。            |
| **unstable-agent-babysitter**  | Event | 使用恢复策略处理不稳定的代理行为。  |

#### 集成

| 钩子                         | 事件                | 描述                                                    |
| ---------------------------- | ------------------- | ------------------------------------------------------- |
| **claude-code-hooks**        | 所有                | 从 Claude Code 的 settings.json 执行钩子。        |
| **atlas**                    | 多个                | 待办事项驱动工作会话的主编排逻辑。 |
| **interactive-bash-session** | PostToolUse + Event | 管理交互式 CLI 的 tmux 会话。              |
| **non-interactive-env**      | PreToolUse          | 处理非交互式环境约束。        |

#### 专用

| 钩子                        | 事件       | 描述                                                       |
| --------------------------- | ---------- | ---------------------------------------------------------- |
| **prometheus-md-only**      | PreToolUse | 强制 Prometheus 规划器仅输出 markdown。      |
| **no-sisyphus-gpt**         | Message    | 阻止 Sisyphus 在不兼容的 GPT 模型上运行。 |
| **no-hephaestus-non-gpt**   | Message    | 阻止 Hephaestus 在非 GPT 模型上运行。        |
| **sisyphus-junior-notepad** | PreToolUse | 管理 Sisyphus-Junior 代理的记事本状态。          |

### Claude Code 钩子集成

通过 Claude Code 的 `settings.json` 运行自定义脚本：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [{ "type": "command", "command": "eslint --fix $FILE" }]
      }
    ]
  }
}
```

**钩子位置**：

- `~/.claude/settings.json` (用户)
- `./.claude/settings.json` (项目)
- `./.claude/settings.local.json` (本地，git 忽略)

### 禁用钩子

在配置中禁用特定钩子：

```json
{
  "disabled_hooks": ["comment-checker"]
}
```

## MCP

### 内置 MCP

| MCP           | 描述                                                                                          |
| ------------- | --------------------------------------------------------------------------------------------- |
| **websearch** | 由 Exa AI 提供支持的实时网络搜索                                                        |
| **context7**  | 任何库/框架的官方文档查找                                       |
| **grep_app**  | 跨公共 GitHub 仓库的超快代码搜索。非常适合查找实现示例。 |

### 技能嵌入的 MCP

技能可以带来自己的 MCP 服务器：

```yaml
---
description: 浏览器自动化技能
mcp:
  playwright:
    command: npx
    args: ["-y", "@anthropic-ai/mcp-playwright"]
---
```

`skill_mcp` 工具使用完整的模式发现调用这些操作。

#### 支持 OAuth 的 MCP

技能可以定义受 OAuth 保护的远程 MCP 服务器。支持具有完全 RFC 兼容性（RFC 9728, 8414, 8707, 7591）的 OAuth 2.1：

```yaml
---
description: 我的 API 技能
mcp:
  my-api:
    url: https://api.example.com/mcp
    oauth:
      clientId: ${CLIENT_ID}
      scopes: ["read", "write"]
---
```

当技能 MCP 配置了 `oauth` 时：

- **自动发现**：获取 `/.well-known/oauth-protected-resource` (RFC 9728)，回退到 `/.well-known/oauth-authorization-server` (RFC 8414)
- **动态客户端注册**：向支持 RFC 7591 的服务器自动注册（clientId 变为可选）
- **PKCE**：所有流程的强制要求
- **资源指示器**：根据 RFC 8707 从 MCP URL 自动生成
- **令牌存储**：持久化在 `~/.config/opencode/mcp-oauth.json` (chmod 0600) 中
- **自动刷新**：令牌在 401 时刷新；带有 `WWW-Authenticate` 的 403 触发升压授权
- **动态端口**：OAuth 回调服务器使用自动发现的可用端口

通过 CLI 预先验证身份：

```bash
bunx oh-my-opencode mcp oauth login <server-name> --server-url https://api.example.com
```

## 模型能力

模型能力由 models.dev 支持，具有可刷新的缓存和兼容性诊断。系统结合了捆绑的 models.dev 快照数据、可选的刷新缓存数据、提供商运行时元数据，以及在没有精确元数据时的启发式方法。

### 刷新能力

使用最新的模型信息更新本地缓存：

```bash
bunx oh-my-opencode refresh-model-capabilities
```

配置启动时的自动刷新：

```jsonc
{
  "model_capabilities": {
    "enabled": true,
    "auto_refresh_on_start": true,
    "refresh_timeout_ms": 5000,
    "source_url": "https://models.dev/api.json"
  }
}
```

### 能力诊断

运行 `bunx oh-my-opencode doctor` 以查看能力诊断，包括：
- 代理和类别的有效模型解析
- 配置的模型依赖兼容性回退时的警告
- 覆盖模型解析输出旁边的兼容性详细信息

## 上下文注入

### 目录 AGENTS.md

在读取文件时自动注入 AGENTS.md。从文件目录向项目根目录遍历：

```
project/
├── AGENTS.md              # 首先注入
├── src/
│   ├── AGENTS.md          # 其次注入
│   └── components/
│       ├── AGENTS.md      # 第三注入
│       └── Button.tsx     # 读取此文件会注入所有 3 个
```

### 条件规则

条件匹配时从 `.claude/rules/` 注入规则：

```markdown
---
globs: ["*.ts", "src/**/*.js"]
description: "TypeScript/JavaScript 编码规则"
---

- 接口名称使用 PascalCase
- 函数名称使用 camelCase
```

支持：

- `.md` 和 `.mdc` 文件
- 用于模式匹配的 `globs` 字段
- 无条件规则使用 `alwaysApply: true`
- 从文件向上遍历到项目根目录，外加 `~/.claude/rules/`

## Claude Code 兼容性

完全兼容 Claude Code 配置层。

### 配置加载器

| 类型         | 位置                                                                               |
| ------------ | ---------------------------------------------------------------------------------- |
| **Commands** | `~/.config/opencode/commands/`, `.claude/commands/`                                |
| **Skills**   | `~/.config/opencode/skills/*/SKILL.md`, `.claude/skills/*/SKILL.md`                |
| **Agents**   | `~/.config/opencode/agents/*.md`, `.claude/agents/*.md`                            |
| **MCPs**     | `~/.claude.json`, `~/.config/opencode/.mcp.json`, `.mcp.json`, `.claude/.mcp.json` |

MCP 配置支持环境变量扩展：`${VAR}`。

### 兼容性开关

禁用特定功能：

```json
{
  "claude_code": {
    "mcp": false,
    "commands": false,
    "skills": false,
    "agents": false,
    "hooks": false,
    "plugins": false
  }
}
```

| 开关       | 禁用功能                                                     |
| ---------- | ------------------------------------------------------------ |
| `mcp`      | `.mcp.json` 文件（保留内置 MCP）                      |
| `commands` | 从 Claude Code 路径加载命令                       |
| `skills`   | 从 Claude Code 路径加载技能                         |
| `agents`   | 从 Claude Code 路径加载代理（保留内置代理） |
| `hooks`    | settings.json 钩子                                          |
| `plugins`  | Claude Code 市场插件                              |

禁用特定插件：

```json
{
  "claude_code": {
    "plugins_override": {
      "claude-mem@thedotmack": false
    }
  }
}
```
