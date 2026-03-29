# 什么是 Oh My OpenAgent？

Oh My OpenAgent 是 OpenCode 的多模型代理编排线束。它将单个 AI 代理转变为实际交付代码的协调开发团队。

不锁定于 Claude。不锁定于 OpenAI。不锁定于任何人。

只有更好的结果，更便宜的模型，真正的编排。

---

## 快速入门

### 安装

将此粘贴到你的 LLM 代理会话中：

```
Install and configure oh-my-openagent by following the instructions here:
https://raw.githubusercontent.com/code-yeongyu/oh-my-openagent/refs/heads/dev/docs/guide/installation.md
```

或阅读完整的 [安装指南](./installation.md) 了解手动设置、提供商身份验证和故障排除。

### 你的第一个任务

安装后，只需输入：

```
ultrawork
```

就是这样。代理会找出一切——探索你的代码库，研究模式，实现功能，使用诊断进行验证。继续工作直到完成。

想要更多控制？按 **Tab** 键进入 [Prometheus 模式](./orchestration.md) 进行基于面试的规划，然后运行 `/start-work` 进行全面编排。

---

## 理念：打破束缚

我们过去称其为“打了类固醇的 Claude Code”。那是错的。

这不是为了让 Claude Code 更好。这是为了打破“一个模型，一个提供商，一种工作方式就足够了”的想法。Anthropic 希望你被锁定。OpenAI 希望你被锁定。每个人都希望你被锁定。

Oh My OpenAgent 不玩这个游戏。它在模型之间进行编排，为正确的工作选择正确的大脑。Claude 用于编排。GPT 用于深度推理。Gemini 用于前端。GPT-5.4 Mini 用于快速任务。全部自动协同工作。

---

## 它是如何工作的：代理编排

Oh My OpenAgent 不是让一个代理做所有事情，而是使用**根据任务类型相互委托的专门代理**。

**架构：**

```
用户请求
    ↓
[意图门] — 对你实际想要什么进行分类
    ↓
[Sisyphus] — 主编排器，计划和委托
    ↓
    ├─→ [Prometheus] — 战略规划（访谈模式）
    ├─→ [Atlas] — Todo 编排和执行
    ├─→ [Oracle] — 架构咨询
    ├─→ [Librarian] — 文档/代码搜索
    ├─→ [Explore] — 快速代码库 grep
    └─→ [基于类别的代理] — 按任务类型专业化
```

当 Sisyphus 委托给子代理时，它不选择模型名称。它选择一个**类别** — `visual-engineering`, `ultrabrain`, `quick`, `deep`。类别自动映射到正确的模型。你什么都不用碰。

深入了解代理如何协作，请参阅 [编排系统指南](./orchestration.md)。

---

## 认识代理们

### Sisyphus：纪律代理

以希腊神话命名。他每天都在推石头。从不停止。从不放弃。

Sisyphus 是你的主编排器。他计划，委托给专家，并通过激进的并行执行推动任务完成。他不会半途而废。他不会分心。他完成任务。

**推荐模型：**

- **Claude Opus 4.6** — 整体最佳体验。Sisyphus 是使用针对 Claude 优化的提示词构建的。
- **Claude Sonnet 4.6** — 能力和成本的良好平衡。
- **Kimi K2.5** — 很好的类似 Claude 的替代方案。许多用户专门运行这个组合。
- **GLM 5** — 稳健的选项，特别是通过 Z.ai。

Sisyphus 仍然在 Claude 系列模型、Kimi 和 GLM 上表现最好。GPT-5.4 现在有专门的提示词路径，但较旧的 GPT 模型仍然不适合，应该路由到 Hephaestus。

### Hephaestus：合法的工匠

带有故意的讽刺意味命名。Anthropic 因为这个项目阻止 OpenCode 使用他们的 API。所以团队建立了一个自主的 GPT 原生代理作为替代。

Hephaestus 运行在 GPT-5.4 上。给他一个目标，而不是一个配方。他探索代码库，研究模式，并端到端地执行，无需手把手指导。他是合法的工匠，因为他生于必需，而非特权。

当你需要深度架构推理、跨多个文件的复杂调试或跨领域知识综合时，使用 Hephaestus。当工作需要 GPT-5.4 的特定优势时，显式切换到他。

**为什么这比原版 Codex CLI 更好：**

- **多模型编排。** 纯 Codex 是单模型的。OmO 自动将不同的任务路由到不同的模型。GPT 用于深度推理。Gemini 用于前端。GPT-5.4 Mini 用于速度。为正确的工作选择正确的大脑。
- **后台代理。** 并行触发 5+ 个代理。这是 Codex 无法做到的。当一个代理编写代码时，另一个研究模式，另一个检查文档。就像真正的开发团队。
- **类别系统。** 任务按意图路由，而不是模型名称。`visual-engineering` 获得 Gemini。`ultrabrain` 获得 GPT-5.4。`quick` 获得 GPT-5.4 Mini。无需手动处理。
- **积累智慧。** 子代理从以前的结果中学习。任务 1 中发现的约定传递给任务 5。早期犯的错误不会重犯。系统随着工作变得更聪明。

### Prometheus：战略规划师

Prometheus 像真正的工程师一样面试你。提出澄清问题。识别范围和歧义。在触碰哪怕一行代码之前构建详细的计划。

按 **Tab** 键进入 Prometheus 模式，或在 Sisyphus 中输入 `@plan "你的任务"`。

### Atlas：指挥

Atlas 执行 Prometheus 的计划。将任务分配给专业的子代理。跨任务积累学习。独立验证完成情况。

运行 `/start-work` 在你最新的计划上激活 Atlas。

### Oracle：顾问

只读的高智商顾问，用于架构决策和复杂调试。当面临不熟悉的模式、安全问题或多系统权衡时，请教 Oracle。

### 配角

- **Metis** — 差距分析器。在计划最终确定之前发现 Prometheus 遗漏的内容。
- **Momus** — 无情的审查员。根据清晰度、验证和上下文标准验证计划。
- **Explore** — 快速代码库 grep。使用侧重速度的模型进行模式发现。
- **Librarian** — 文档和 OSS 代码搜索。保持对库 API 和最佳实践的最新了解。
- **Multimodal Looker** — 视觉和截图分析。

---

## 工作模式

### Ultrawork 模式：致懒人

输入 `ultrawork` 或直接 `ulw`。就是这样。

代理会找出一切。探索你的代码库。研究模式。实现功能。使用诊断进行验证。继续工作直到完成。

这是“只管去做”模式。全自动。你不需要深入思考，因为代理为你深入思考。

### Prometheus 模式：致精确者

按 **Tab** 键进入 Prometheus 模式。

Prometheus 像真正的工程师一样面试你。提出澄清问题。识别范围和歧义。在触碰哪怕一行代码之前构建详细的计划。

然后运行 `/start-work`，Atlas 接管。任务被分配给专业的子代理。每次完成都独立验证。跨任务积累学习。进度跨会话跟踪。

将 Prometheus 用于多天的项目、关键的生产更改、复杂的重构，或当你想要记录的决策线索时。

---

## 代理模型匹配

不同的代理最适合不同的模型。Oh My OpenAgent 自动分配最佳模型，但你可以自定义一切。

### 默认配置

模型在安装时自动配置。交互式安装程序会询问你有哪些提供商，然后为每个代理和类别生成最佳的模型分配。

在运行时，回退链确保即使你首选的提供商宕机，工作也能继续。每个代理都有一个提供商优先级链。系统按顺序尝试提供商，直到找到可用的模型。

### 自定义模型配置

你可以在你的配置中覆盖特定的代理或类别：

```jsonc
{
  "$schema": "https://raw.githubusercontent.com/code-yeongyu/oh-my-openagent/dev/assets/oh-my-openagent.schema.json",

  "agents": {
    // 主编排器：Claude Opus 或 Kimi K2.5 效果最好
    "sisyphus": {
      "model": "kimi-for-coding/k2p5",
      "ultrawork": { "model": "anthropic/claude-opus-4-6", "variant": "max" },
    },

    // 研究代理：更便宜的模型也可以
    "librarian": { "model": "google/gemini-3-flash" },
    "explore": { "model": "github-copilot/grok-code-fast-1" },

    // 架构咨询：GPT 或 Claude Opus
    "oracle": { "model": "openai/gpt-5.4", "variant": "high" },
  },

  "categories": {
    // 前端工作：Gemini 主导视觉任务
    "visual-engineering": {
      "model": "google/gemini-3.1-pro",
      "variant": "high",
    },

    // 一般的高努力工作
    "unspecified-high": { "model": "anthropic/claude-opus-4-6", "variant": "max" },

    // 快速任务：使用 GPT-5.4-mini (快速且便宜)
    "quick": { "model": "openai/gpt-5.4-mini" },

    // 深度推理：GPT-5.4
    "ultrabrain": { "model": "openai/gpt-5.4", "variant": "xhigh" },
  },
}
```

### 模型系列

**类 Claude 模型**（遵循指令、结构化输出）：

- Claude Opus 4.6, Claude Sonnet 4.6, Claude Haiku 4.5
- Kimi K2.5 — 行为与 Claude 非常相似
- GLM 5 — 类 Claude 行为，适合广泛的任务

**GPT 模型**（显式推理、原则驱动）：

- GPT-5.4 — 深度编码强国，Hephaestus 需要，Oracle 的默认值
- GPT-5-Nano — 超便宜、快速的实用工具任务

**行为不同的模型**：

- Gemini 3.1 Pro — 擅长视觉/前端任务
- MiniMax M2.7 / M2.7-highspeed — 快速且聪明的实用工具任务
- Grok Code Fast 1 — 针对代码 grep/搜索进行了优化

有关哪些模型最适合每个代理、安全与危险的覆盖以及提供商优先级链的完整详细信息，请参阅 [代理-模型匹配指南](./agent-model-matching.md)。

---

## 为什么它比纯 Claude Code 更好

Claude Code 很好。但它是单一代理运行单一模型单独完成所有事情。

Oh My OpenAgent 将其转变为一个协调的团队：

**并行执行。** Claude Code 一次处理一件事。OmO 并行启动后台代理——研究、实现和验证同时进行。就像有 5 个工程师而不是 1 个。

**基于哈希的编辑。** 当模型无法准确重现行时，Claude Code 的编辑工具会失败。OmO 的 `LINE#ID` 内容哈希在应用之前验证每次编辑。Grok Code Fast 1 的成功率仅通过这一更改就从 6.7% 提高到了 68.3%。

**意图门。** Claude Code 接收你的提示词并运行。OmO 首先对你的真实意图进行分类——研究、实现、调查、修复——然后进行相应的路由。更少的误解，更好的结果。

**LSP + AST 工具。** 工作区级重命名、跳转到定义、查找引用、构建前诊断、AST 感知的代码重写。原版 Claude Code 不具备的 IDE 精度。

**带有嵌入式 MCP 的技能。** 每项技能都带来了自己的 MCP 服务器，其作用域限于任务。上下文窗口保持清洁，而不是因为每个工具而膨胀。

**纪律执行。** Todo 执行器将空闲的代理拉回工作岗位。注释检查器剥离 AI 糟粕。Ralph Loop 一直持续到 100% 完成。系统不允许代理偷懒。

**根本优势。** 模型有不同的气质。Claude 思考得很深。GPT 进行架构推理。Gemini 可视化。Haiku 移动得很快。单模型工具迫使你为所有任务选择一种个性。Oh My OpenAgent 利用了它们所有，按任务类型路由。这不是临时的黑客手段——随着模型进一步专业化，这是唯一有意义的架构。多模型编排和单模型限制之间的差距每个月都在扩大。我们押注于那个未来。

---

## 意图门

在处理任何请求之前，Sisyphus 会对你的真实意图进行分类。

你是在要求研究吗？实现？调查？修复？意图门弄清楚你实际想要什么，而不仅仅是你输入的字面单词。这意味着代理理解上下文、细微差别以及你请求背后的真正目标。

Claude Code 没有这个。它接收你的提示词并运行。Oh My OpenAgent 先思考，后行动。

---

## 下一步是什么

- **[安装指南](./installation.md)** — 完整的设置说明、提供商身份验证和故障排除
- **[编排指南](./orchestration.md)** — 深入了解代理协作、使用 Prometheus 进行规划以及使用 Atlas 进行执行
- **[代理-模型匹配指南](./agent-model-matching.md)** — 哪些模型最适合每个代理以及如何自定义
- **[配置参考](../reference/configuration.md)** — 带有示例的完整配置选项
- **[功能参考](../reference/features.md)** — 完整的功能文档
- **[宣言](../manifesto.md)** — 项目背后的理念

---

**准备好开始了吗？** 输入 `ultrawork`，看看一个协调的 AI 团队能做什么。
