# 代理-模型匹配指南

> **致代理和用户**：为什么每个代理都需要一个特定的模型——以及如何在不破坏任何东西的情况下进行定制。

## 核心见解：模型就是开发者

把 AI 模型想象成团队中的开发者。每个人都有不同的大脑、不同的个性、不同的优势。**一个模型不仅仅是“更聪明”或“更笨”。它的思维方式不同。** 给 Claude 和 GPT 同样的指令，它们会以根本不同的方式解释它。

这不是一个 bug。这是整个系统的基础。

Oh My OpenAgent 为每个代理分配了一个与其*工作风格*相匹配的模型——就像组建一个团队，让每个人都在适合其个性的角色中一样。

### Sisyphus：善于交际的领导者

Sisyphus 是那种认识所有人、无处不在、通过沟通和协调完成工作的开发者。与其他代理交谈，了解整个代码库的上下文，智能地委派工作，并且代码也写得很好。但是深度的、纯技术性的问题？他会稍微挣扎一下。

**这就是为什么 Sisyphus 使用 Claude / Kimi / GLM。** 这些模型擅长：

- 遵循复杂的多步指令（Sisyphus 的提示词大约 1,100 行）
- 在许多工具调用之间保持对话流畅
- 理解细微的委派和编排模式
- 产生结构良好、富有沟通性的输出

将 Sisyphus 与较旧的 GPT 模型一起使用，就像把你最好的项目经理——那个协调每个人、主持站会、保持整个团队一致的人——独自关在一个房间里调试竞争条件。完全不合适。GPT-5.4 现在有专门的 Sisyphus 提示词路径，但 GPT 仍然不是编排器的默认推荐。

### Hephaestus：深度专家

Hephaestus 是那种整天待在房间里写代码的开发者。话不多。可能看起来有点社交尴尬。但给他们一个艰深的技术问题，三个小时后他们就会带着别人找不到的解决方案出现。

**这就是为什么 Hephaestus 使用 GPT-5.4。** GPT-5.4 就是为此而生的：

- 没有手把手指导的深度自主探索
- 跨复杂代码库的多文件推理
- 原则驱动的执行（给出一个目标，而不是一个配方）
- 长时间独立工作

将 Hephaestus 与 GLM 或 Kimi 一起使用，就像把你最善于沟通、最会社交的开发者分配去独自坐着，除了深度的技术工作什么也不做。他们最终会完成的，但他们不会发光——你浪费了恰好使他们有价值的技能。

### 总结

每个代理的提示词都经过调整以匹配其模型的个性。**当你改变模型时，你就改变了大脑——同样的指令会被完全不同地理解。** 模型匹配不是关于“更好”或“更坏”。它是关于合适。

---

## Claude 和 GPT 的思维方式有何不同

这对于理解为什么有些代理支持这两种模型系列而另一些不支持很重要。

**Claude** 响应**机制驱动**的提示词——详细的清单、模板、分步过程。规则越多 = 依从性越高。你可以写一个带有嵌套工作流的 1,100 行提示词，Claude 会遵循每一步。

**GPT**（尤其是 5.2+）响应**原则驱动**的提示词——简洁的原则、XML 结构、明确的决策标准。规则越多 = 矛盾面越大 = 偏离越多。当你陈述目标并让它自己找出机制时，GPT 工作得最好。

真实示例：Prometheus 的 Claude 提示词分布在 7 个文件中，大约 1,100 行。GPT 提示词通过约 121 行的 3 个原则实现了相同的行为。相同的结果，完全不同的方法。

支持这两个系列（Prometheus, Atlas）的代理在运行时自动检测你的模型，并通过 `isGptModel()` 切换提示词。你不需要去想它。

---

## 代理档案

### 沟通者 → Claude / Kimi / GLM

这些代理拥有针对 Claude 优化的提示词——冗长、详细、机制驱动。它们需要能够可靠遵循复杂、多层指令的模型。

| 代理         | 角色              | 回退链                                 | 备注                                                                                             |
| ------------ | ----------------- | -------------------------------------- | ------------------------------------------------------------------------------------------------- |
| **Sisyphus** | 主编排器          | Claude Opus → opencode-go/kimi-k2.5 → K2P5 → Kimi K2.5 → GPT-5.4 → GLM-5 → Big Pickle | Claude 系列优先。GPT-5.4 有专用的提示词支持。Kimi 可通过多个提供商获取。 |
| **Metis**    | 计划差距分析器    | Claude Opus → GPT-5.4 → opencode-go/glm-5 → K2P5 | 首选 Claude。GPT-5.4 作为 GLM-5 回退之前的次选。                                     |

### 双提示词代理 → 首选 Claude，支持 GPT

这些代理为 Claude 和 GPT 系列提供单独的提示词。它们在运行时自动检测你的模型并切换。

| 代理           | 角色              | 回退链                                 | 备注                                                                 |
| -------------- | ----------------- | -------------------------------------- | -------------------------------------------------------------------- |
| **Prometheus** | 战略规划师        | Claude Opus → GPT-5.4 → opencode-go/glm-5 → Gemini 3.1 Pro | 访谈模式规划。GPT 提示词紧凑且由原则驱动。 |
| **Atlas**      | Todo 编排器       | Claude Sonnet → opencode-go/kimi-k2.5 → GPT-5.4 | Claude 优先，opencode-go 作为中间项，GPT-5.4 作为最后手段。   |

### 深度专家 → GPT

这些代理专为 GPT 的原则驱动风格而建。它们的提示词假设自主的、目标导向的执行。不要覆盖为 Claude。

| 代理           | 角色                    | 回退链                                 | 备注                                             |
| -------------- | ----------------------- | -------------------------------------- | ------------------------------------------------ |
| **Hephaestus** | 自主深度工作者          | GPT-5.4 (medium)                      | 需要 GPT 访问权限。工匠。 |
| **Oracle**     | 架构顾问                | GPT-5.4 → Gemini 3.1 Pro → Claude Opus → opencode-go/glm-5 | 只读的高智商咨询。                  |
| **Momus**      | 无情的审查员            | GPT-5.4 → Claude Opus → Gemini 3.1 Pro → opencode-go/glm-5 | 验证和计划审查。GPT-5.4 使用 xhigh 变体。 |

### 实用工具运行者 → 速度胜于智能

这些代理执行 grep、搜索和检索。它们有意使用最快、最便宜的可用模型。**不要将它们“升级”到 Opus**——那是在雇佣高级工程师来归档文书工作。

| 代理                  | 角色               | 回退链                                         | 备注                                                  |
| --------------------- | ------------------ | ---------------------------------------------- | ----------------------------------------------------- |
| **Explore**           | 快速代码库 grep    | Grok Code Fast → opencode-go/minimax-m2.7 → opencode/minimax-m2.5 → Haiku → GPT-5-Nano | 速度就是一切。并行发射 10 个。在提供商目录公开时使用 opencode-go/minimax-m2.7，回退到 opencode/minimax-m2.5。 |
| **Librarian**         | 文档/代码搜索      | opencode-go/minimax-m2.7 → opencode/minimax-m2.5 → Haiku → GPT-5-Nano                  | 文档检索不需要深度推理。在提供商目录公开时使用 opencode-go/minimax-m2.7，回退到 opencode/minimax-m2.5。 |
| **Multimodal Looker** | 视觉/截图          | GPT-5.4 → opencode-go/kimi-k2.5 → GLM-4.6v → GPT-5-Nano                                 | 使用第一个可用的支持多模态的回退。 |
| **Sisyphus-Junior**   | 类别执行器         | Claude Sonnet → opencode-go/kimi-k2.5 → GPT-5.4 → MiniMax M2.7 → Big Pickle              | 处理委托的类别任务。Sonnet 级别的默认值。 |

---

## 模型系列

### Claude 系列

善于沟通、遵循指令、结构化输出。最适合需要遵循复杂多步提示词的代理。

| 模型                  | 优势                                                                         |
| --------------------- | ---------------------------------------------------------------------------- |
| **Claude Opus 4.6**   | 整体最佳。对复杂提示词的依从性最高。Sisyphus 的默认值。 |
| **Claude Sonnet 4.6** | 更快、更便宜。日常任务的良好平衡。                            |
| **Claude Haiku 4.5**  | 快速且便宜。适合快速任务和实用工具工作。                       |
| **Kimi K2.5**         | 行为与 Claude 非常相似。低成本下的全能好手。           |
| **GLM 5**             | 类似 Claude 的行为。对于编排任务很可靠。                         |

### GPT 系列

原则驱动、显式推理、深度技术能力。最适合在复杂问题上自主工作的代理。

| 模型              | 优势                                                                                            |
| ----------------- | ----------------------------------------------------------------------------------------------- |
| **GPT-5.3 Codex** | 深度编码强国。自主探索。仍可用于深度类别和显式覆盖。 |
| **GPT-5.4**       | 高智商、战略推理。Oracle、Momus 的默认值，也是 Prometheus / Atlas 的关键回退。对 Momus 使用 xhigh 变体。 |
| **GPT-5.4 Mini**  | 快速 + 强大的推理。适合轻量级自主任务。quick 类别的默认值。 |
| **GPT-5-Nano**    | 超级便宜、快速。适合简单的实用工具任务。                                               |

### 其他模型

| 模型                 | 优势                                                                                                         |
| -------------------- | ------------------------------------------------------------------------------------------------------------ |
| **Gemini 3.1 Pro**   | 擅长视觉/前端任务。不同的推理风格。`visual-engineering` 和 `artistry` 的默认值。 |
| **Gemini 3 Flash**   | 快速。适合文档搜索和轻量级任务。                                                                   |
| **Grok Code Fast 1** | 极快的代码 grep。Explore 代理的默认值。                                                           |
| **MiniMax M2.7**     | 快速且聪明。在提供商目录公开较新的 MiniMax 产品线时使用，特别是通过 OpenCode Go。 |
| **MiniMax M2.5**     | 传统的 OpenCode 目录条目，仍在一些回退链中用于兼容性。 |

### OpenCode Go

高级订阅层（$10/月），通过 OpenCode 的基础设施提供对中国前沿模型的可靠访问。

**可用模型：**

| 模型                     | 用例                                                                  |
| ------------------------ | --------------------------------------------------------------------- |
| **opencode-go/kimi-k2.5** | 支持视觉，类似 Claude 的推理。被 Sisyphus, Atlas, Sisyphus-Junior, Multimodal Looker 使用。 |
| **opencode-go/glm-5**     | 仅文本的编排模型。被 Oracle, Prometheus, Metis, Momus 使用。                           |
| **opencode-go/minimax-m2.7** | 超便宜，快速响应。被 Librarian, Explore, Atlas, 和 Sisyphus-Junior 用于实用工具工作。 |

**何时被使用：**

OpenCode Go 模型作为中间选项出现在回退链中。它们弥合了高级 Claude 访问和免费层替代方案之间的差距。系统会在回退到特定于提供商的更便宜的条目（如 MiniMax 或 Big Pickle）之前尝试 OpenCode Go 模型，然后在适用的情况下回退到 GPT 替代方案。

**仅限 Go 的场景：**

某些模型标识符（如 `k2p5`（付费 Kimi K2.5）和 `glm-5`）可能仅在某些地区的 OpenCode Go 订阅中可用。当使用这些短标识符配置时，系统会首先通过 opencode-go 提供商解析它们。

### 关于免费层回退

你可能会在源代码或日志中看到类似 `kimi-k2.5-free`、`minimax-m2.7`、`minimax-m2.5` 或 `big-pickle`（GLM 4.6）的模型名称。这些是回退链中特定于提供商或针对速度优化的条目。确切的 MiniMax 模型可能因提供商目录而异。

你不需要配置它们。系统包含它们，以便在你没有所有付费订阅时能优雅地降级。如果你有付费版本，付费版本始终是首选。

---

## 任务类别

当代理委派工作时，它们不选择模型名称——它们选择一个**类别**。类别会自动映射到正确的模型。

| 类别                 | 何时使用                   | 回退链                                       |
| -------------------- | -------------------------- | -------------------------------------------- |
| `visual-engineering` | 前端、UI、CSS、设计        | Gemini 3.1 Pro → GLM 5 → Claude Opus → opencode-go/glm-5 → K2P5 |
| `ultrabrain`         | 需要最大推理能力           | GPT-5.4 → Gemini 3.1 Pro → Claude Opus → opencode-go/glm-5 |
| `deep`               | 深度编码、复杂逻辑         | GPT-5.3 Codex → Claude Opus → Gemini 3.1 Pro |
| `artistry`           | 创意的、新颖的方法         | Gemini 3.1 Pro → Claude Opus → GPT-5.4       |
| `quick`              | 简单、快速的任务           | GPT-5.4 Mini → Claude Haiku → Gemini Flash → opencode-go/minimax-m2.7 → GPT-5-Nano |
| `unspecified-high`   | 一般的复杂工作             | Claude Opus → GPT-5.4 → GLM 5 → K2P5 → opencode-go/glm-5 → Kimi K2.5 |
| `unspecified-low`    | 一般的标准工作             | Claude Sonnet → GPT-5.3 Codex → opencode-go/kimi-k2.5 → Gemini Flash |
| `writing`            | 文本、文档、散文           | Gemini Flash → opencode-go/kimi-k2.5 → Claude Sonnet |

请参阅 [编排系统指南](./orchestration.md) 了解代理如何将任务调度到类别。

---

## 定制化

### 配置示例

```jsonc
{
  "$schema": "https://raw.githubusercontent.com/code-yeongyu/oh-my-openagent/dev/assets/oh-my-opencode.schema.json",

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

    // Prometheus 继承 sisyphus 模型；只需添加提示词指导
    "prometheus": {
      "prompt_append": "大量利用 deep & quick 代理，始终并行。",
    },
  },

  "categories": {
    "quick": { "model": "opencode/gpt-5-nano" },
    "unspecified-low": { "model": "anthropic/claude-sonnet-4-6" },
    "unspecified-high": { "model": "anthropic/claude-opus-4-6", "variant": "max" },
    "visual-engineering": {
      "model": "google/gemini-3.1-pro",
      "variant": "high",
    },
    "writing": { "model": "google/gemini-3-flash" },
  },

  // 限制昂贵的提供商；让便宜的自由运行
  "background_task": {
    "providerConcurrency": {
      "anthropic": 3,
      "openai": 3,
      "opencode": 10,
      "zai-coding-plan": 10,
    },
    "modelConcurrency": {
      "anthropic/claude-opus-4-6": 2,
      "opencode/gpt-5-nano": 20,
    },
  },
}
```

运行 `opencode models` 查看可用模型，运行 `opencode auth login` 对提供商进行身份验证。

### 安全的 vs 危险的覆盖

**安全的** — 相同个性类型：

- Sisyphus: Opus → Sonnet, Kimi K2.5, GLM 5 (所有沟通型模型)
- Prometheus: Opus → GPT-5.4 (自动切换到 GPT 提示词)
- Atlas: Claude Sonnet 4.6 → GPT-5.4 (自动切换到 GPT 提示词)

**危险的** — 个性不匹配：

- Sisyphus → 较旧的 GPT 模型：**仍然不合适。GPT-5.4 是唯一专用的 GPT 提示词路径。**
- Hephaestus → Claude：**为 Codex 的自主风格而建。Claude 无法复制这一点。**
- Explore → Opus：**巨大的成本浪费。Explore 需要速度，而不是智能。**
- Librarian → Opus：**同上。文档搜索不需要 Opus 级别的推理。**

### 模型解析如何工作

每个代理都有一个回退链。系统按优先级顺序尝试模型，直到找到一个可通过你连接的提供商获取的模型。你不需要为每个模型配置提供商。只需进行身份验证（`opencode auth login`），系统就会找出哪些模型可用以及在何处可用。

核心代理的标签页循环通过注入的运行时 `order` 字段来确定。固定的优先级顺序是 Sisyphus（顺序：1）、Hephaestus（顺序：2）、Prometheus（顺序：3）和 Atlas（顺序：4），然后是剩余的代理。

你的显式配置始终胜出。如果你为代理设置了特定模型，即使解析数据处于冷状态，该选择也会优先。

Variant 和 `reasoningEffort` 覆盖会被规范化为模型支持的值，因此跨提供商的覆盖会优雅降级，而不是硬失败。

模型能力以 models.dev 为后盾，具有可刷新的缓存和能力诊断。使用 `bunx oh-my-opencode refresh-model-capabilities` 更新缓存，或配置 `model_capabilities.auto_refresh_on_start` 以在启动时刷新。

要查看你的代理实际将使用哪些模型，请运行 `bunx oh-my-opencode doctor`。这将根据你当前的身份验证和配置显示有效的模型解析。

```
代理请求 → 用户覆盖 (如果配置了) → 回退链 → 系统默认
```

### 基于文件的提示词

你可以使用 `prompt` 字段中的 `file://` URL 从外部文件加载代理系统提示词，或使用 `prompt_append` 附加额外内容。`prompt_append` 字段也适用于类别。

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

文件内容在运行时加载并注入到代理的系统提示词中。支持 `~` 扩展为家目录和相对的 `file://` 路径。

---

## 另请参阅

- [安装指南](./installation.md) — 设置和身份验证
- [编排系统指南](./orchestration.md) — 代理如何将任务调度到类别
- [配置参考](../reference/configuration.md) — 完整的配置选项
- [`src/shared/model-requirements.ts`](../../src/shared/model-requirements.ts) — 回退链的事实来源
