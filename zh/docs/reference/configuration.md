# 配置参考

Oh My OpenCode 插件配置的完整参考。在重命名过渡期间，运行时识别 `oh-my-openagent.json[c]` 和旧的 `oh-my-opencode.json[c]` 文件。

---

## 目录

- [快速入门](#快速入门)
  - [文件位置](#文件位置)
  - [快速入门示例](#快速入门示例)
- [核心概念](#核心概念)
  - [代理](#代理)
  - [类别](#类别)
  - [模型解析](#模型解析)
- [任务系统](#任务系统)
  - [后台任务](#后台任务)
  - [Sisyphus 代理](#sisyphus-代理)
  - [Sisyphus 任务](#sisyphus-任务)
- [功能](#功能)
  - [技能](#技能)
  - [钩子](#钩子)
  - [命令](#命令)
  - [浏览器自动化](#浏览器自动化)
  - [Tmux 集成](#tmux-集成)
  - [Git Master](#git-master)
  - [注释检查器](#注释检查器)
  - [通知](#通知)
  - [MCP](#mcps)
  - [LSP](#lsp)
- [高级](#高级)
  - [运行时回退](#运行时回退)
  - [模型能力](#模型能力)
  - [哈希行编辑](#哈希行编辑)
  - [实验性](#实验性)
- [参考](#参考)
  - [环境变量](#环境变量)
  - [特定于提供商](#特定于提供商)

---

## 快速入门

### 文件位置

首先加载用户配置，然后项目配置覆盖它。在每个目录中，兼容层识别重命名和旧的基本名称。

1. 项目配置：`.opencode/oh-my-openagent.json[c]` 或 `.opencode/oh-my-opencode.json[c]`
2. 用户配置（首选 `.jsonc` 而不是 `.json`）：

| 平台        | 路径候选                                                                                     |
| ----------- | -------------------------------------------------------------------------------------------- |
| macOS/Linux | `~/.config/opencode/oh-my-openagent.json[c]`, `~/.config/opencode/oh-my-opencode.json[c]`    |
| Windows     | `%APPDATA%\opencode\oh-my-openagent.json[c]`, `%APPDATA%\opencode\oh-my-opencode.json[c]`    |

**重命名兼容性：** 发布的包和 CLI 二进制文件仍为 `oh-my-opencode`。OpenCode 插件注册首选 `oh-my-openagent`，而旧的 `oh-my-opencode` 条目和配置基本名称在过渡期间仍会加载。配置检测会在检查 `oh-my-openagent` 之前检查 `oh-my-opencode`，因此如果同一个目录中存在这两个插件配置基本名称，目前以旧的 `oh-my-opencode.*` 文件为准。
JSONC 支持 `// 行注释`、`/* 块注释 */` 和尾随逗号。

启用模式自动完成：

```json
{
  "$schema": "https://raw.githubusercontent.com/code-yeongyu/oh-my-openagent/dev/assets/oh-my-opencode.schema.json"
}
```

运行 `bunx oh-my-opencode install` 进行引导设置。运行 `opencode models` 列出可用模型。

### 快速入门示例

这是一个实用的起始配置：

```jsonc
{
  "$schema": "https://raw.githubusercontent.com/code-yeongyu/oh-my-openagent/dev/assets/oh-my-opencode.schema.json",

  "agents": {
    // 主编排器：Claude Opus 或 Kimi K2.5 效果最好
    "sisyphus": {
      "model": "kimi-for-coding/k2p5",
      "ultrawork": { "model": "anthropic/claude-opus-4-6", "variant": "max" },
    },

    // 研究代理：便宜快速的模型就可以了
    "librarian": { "model": "google/gemini-3-flash" },
    "explore": { "model": "github-copilot/grok-code-fast-1" },

    // 架构咨询：GPT-5.4 或 Claude Opus
    "oracle": { "model": "openai/gpt-5.4", "variant": "high" },

    // Prometheus 继承 sisyphus 模型；只需添加提示词指导
    "prometheus": {
      "prompt_append": "大量利用 deep & quick 代理，始终并行。",
    },
  },

  "categories": {
    // quick - 琐碎的任务
    "quick": { "model": "opencode/gpt-5-nano" },

    // unspecified-low - 中等任务
    "unspecified-low": { "model": "anthropic/claude-sonnet-4-6" },

    // unspecified-high - 复杂工作
    "unspecified-high": { "model": "anthropic/claude-opus-4-6", "variant": "max" },

    // writing - 文档/散文
    "writing": { "model": "google/gemini-3-flash" },

    // visual-engineering - Gemini 主导视觉任务
    "visual-engineering": {
      "model": "google/gemini-3.1-pro",
      "variant": "high",
    },

    // 用于 git 操作的自定义类别
    "git": {
      "model": "opencode/gpt-5-nano",
      "description": "所有 git 操作",
      "prompt_append": "专注于原子提交、清晰的消息和安全的操作。",
    },
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

  "experimental": { "aggressive_truncation": true, "task_system": true },
  "tmux": { "enabled": false },
}
```

---

## 核心概念

### 代理

覆盖内置代理设置。可用代理：`sisyphus`, `hephaestus`, `prometheus`, `oracle`, `librarian`, `explore`, `multimodal-looker`, `metis`, `momus`, `atlas`。

```json
{
  "agents": {
    "explore": { "model": "anthropic/claude-haiku-4-5", "temperature": 0.5 },
    "multimodal-looker": { "disable": true }
  }
}
```

完全禁用代理：`{ "disabled_agents": ["oracle", "multimodal-looker"] }`

核心代理会接收一个注入的运行时 `order` 字段，以便在 UI 中进行确定性的 Tab 循环：Sisyphus = 1，Hephaestus = 2，Prometheus = 3，Atlas = 4。这不是用户可配置的配置键。

#### 代理选项

| 选项              | 类型           | 描述                                                            |
| ----------------- | -------------- | --------------------------------------------------------------- |
| `model`           | string         | 模型覆盖 (`provider/model`)                                     |
| `fallback_models` | string\|array  | API 错误时的回退模型。支持字符串或混合了字符串和带有每个模型设置的对象条目的数组 |
| `temperature`     | number         | 采样温度                                                        |
| `top_p`           | number         | Top-p 采样                                                      |
| `prompt`          | string         | 替换系统提示词。支持 `file://` URI                              |
| `prompt_append`   | string         | 附加到系统提示词。支持 `file://` URI                            |
| `tools`           | array          | 允许的工具列表                                                  |
| `disable`         | boolean        | 禁用此代理                                                      |
| `mode`            | string         | 代理模式                                                        |
| `color`           | string         | UI 颜色                                                         |
| `permission`      | object         | 每个工具的权限（见下文）                                        |
| `category`        | string         | 从类别继承模型                                                  |
| `variant`         | string         | 模型变体：`max`, `high`, `medium`, `low`, `xhigh`。规范化为支持的值 |
| `maxTokens`       | number         | 最大响应 token 数                                               |
| `thinking`        | object         | Anthropic 扩展思考                                              |
| `reasoningEffort` | string         | OpenAI 推理：`none`, `minimal`, `low`, `medium`, `high`, `xhigh`。规范化为支持的值 |
| `textVerbosity`   | string         | 文本冗长度：`low`, `medium`, `high`                             |
| `providerOptions` | object         | 特定于提供商的选项                                              |

#### Anthropic 扩展思考

```json
{
  "agents": {
    "oracle": { "thinking": { "type": "enabled", "budgetTokens": 200000 } }
  }
}
```

#### 代理权限

控制代理可以使用哪些工具：

```json
{
  "agents": {
    "explore": {
      "permission": {
        "edit": "deny",
        "bash": "ask",
        "webfetch": "allow"
      }
    }
  }
}
```

| 权限                 | 值                                                                          |
| -------------------- | --------------------------------------------------------------------------- |
| `edit`               | `ask` / `allow` / `deny`                                                    |
| `bash`               | `ask` / `allow` / `deny` 或按命令: `{ "git": "allow", "rm": "deny" }`       |
| `webfetch`           | `ask` / `allow` / `deny`                                                    |
| `doom_loop`          | `ask` / `allow` / `deny`                                                    |
| `external_directory` | `ask` / `allow` / `deny`                                                    |


#### 带有每个模型设置的回退模型

`fallback_models` 接受单个模型字符串或数组。数组条目可以是纯字符串或具有单独模型设置的对象：

```jsonc
{
  "agents": {
    "sisyphus": {
      "model": "anthropic/claude-opus-4-6",
      "fallback_models": [
        // 简单的字符串回退
        "openai/gpt-5.4",
        // 带有每个模型设置的对象
        {
          "model": "google/gemini-3.1-pro",
          "variant": "high",
          "temperature": 0.2
        },
        {
          "model": "anthropic/claude-sonnet-4-6",
          "thinking": { "type": "enabled", "budgetTokens": 64000 }
        }
      ]
    }
  }
}
```

对象条目支持：`model`, `variant`, `reasoningEffort`, `temperature`, `top_p`, `maxTokens`, `thinking`。

#### 提示词的文件 URI

`prompt` 和 `prompt_append` 都支持通过 `file://` URI 从文件加载内容。类别级别的 `prompt_append` 支持相同的 URI 形式。

```jsonc
{
  "agents": {
    "sisyphus": {
      "prompt_append": "file:///absolute/path/to/prompt.txt"
    },
    "oracle": {
      "prompt": "file://./relative/to/project/prompt.md"
    },
    "explore": {
      "prompt_append": "file://~/home/dir/prompt.txt"
    }
  },
  "categories": {
    "custom": {
      "model": "anthropic/claude-sonnet-4-6",
      "prompt_append": "file://./category-context.md"
    }
  }
}
```

路径可以是绝对的（`file:///abs/path`）、相对于项目根目录（`file://./rel/path`）或相对于主目录（`file://~/home/path`）。如果无法解码、解析或读取文件 URI，OmO 会在提示词中插入警告占位符，而不是硬失败。

### 类别

`task()` 工具使用的特定领域模型委托。当 Sisyphus 委托工作时，它选择一个类别，而不是模型名称。

#### 内置类别

| 类别                 | 默认模型                        | 描述                                           |
| -------------------- | ------------------------------- | ---------------------------------------------- |
| `visual-engineering` | `google/gemini-3.1-pro` (high)  | 前端、UI/UX、设计、动画                        |
| `ultrabrain`         | `openai/gpt-5.4` (xhigh)        | 深度逻辑推理、复杂的架构                       |
| `deep`               | `openai/gpt-5.3-codex` (medium) | 自主解决问题、彻底的研究                       |
| `artistry`           | `google/gemini-3.1-pro` (high)  | 创造性/非常规的方法                            |
| `quick`              | `openai/gpt-5.4-mini`           | 琐碎的任务、错别字修复、单文件更改             |
| `unspecified-low`    | `anthropic/claude-sonnet-4-6`   | 一般任务，低努力量                             |
| `unspecified-high`   | `anthropic/claude-opus-4-6` (max) | 一般任务，高努力量                           |
| `writing`            | `google/gemini-3-flash`         | 文档、散文、技术写作                           |

> **注意**：内置默认值仅在配置中存在该类别时才适用。否则将使用系统默认模型。

#### 类别选项

| 选项                | 类型          | 默认值  | 描述                                                                |
| ------------------- | ------------- | ------- | ------------------------------------------------------------------- |
| `model`             | string        | -       | 模型覆盖                                                            |
| `fallback_models`   | string\|array | -       | API 错误时的回退模型。支持字符串或混合了字符串和对象条目的数组      |
| `temperature`       | number        | -       | 采样温度                                                            |
| `top_p`             | number        | -       | Top-p 采样                                                          |
| `maxTokens`         | number        | -       | 最大响应 token 数                                                   |
| `thinking`          | object        | -       | Anthropic 扩展思考                                                  |
| `reasoningEffort`   | string        | -       | OpenAI 推理努力。不支持的值会被规范化                               |
| `textVerbosity`     | string        | -       | 文本冗长度                                                          |
| `tools`             | array         | -       | 允许的工具                                                          |
| `prompt_append`     | string        | -       | 附加到系统提示词                                                    |
| `variant`           | string        | -       | 模型变体。不支持的值会被规范化                                      |
| `description`       | string        | -       | 显示在 `task()` 工具提示词中                                        |
| `is_unstable_agent` | boolean       | `false` | 强制后台模式 + 监控。对于 Gemini 模型自动启用。                     |

禁用类别：`{ "disabled_categories": ["ultrabrain"] }`

### 模型解析

运行时优先级：

1. **UI 选择的模型** - 在 OpenCode UI 中选择的模型，适用于主要代理
2. **用户覆盖** - 在配置中设置的模型 → 按原样使用。即使在冷缓存上，显式用户配置也优先于硬编码的回退链
3. **类别默认值** - 从分配的类别配置继承的模型
4. **用户 `fallback_models`** - 在内置回退链之前尝试用户配置的回退列表
5. **提供商回退链** - 来自 OmO 源的内置提供商/模型链
6. **系统默认值** - OpenCode 配置的默认模型

#### 模型设置兼容性

模型设置根据模型能力进行兼容性规范化，而不是硬失败。

规范化的字段：

- `variant` - 降级到最接近的支持值
- `reasoningEffort` - 降级到最接近的支持值，如果不支持则移除
- `temperature` - 如果模型元数据不支持则移除
- `top_p` - 如果模型元数据不支持则移除
- `maxTokens` - 限制在模型报告的最大输出限制内
- `thinking` - 如果目标模型不支持思考则移除

示例：
- Claude 模型不支持 `reasoningEffort` - 自动移除
- GPT-4.1 不支持推理 - `reasoningEffort` 被移除
- o 系列模型支持 `none` 到 `high` - `xhigh` 降级为 `high`
- GPT-5 支持 `none`, `minimal`, `low`, `medium`, `high`, `xhigh` - 全部通过

能力数据首先来自提供商运行时元数据。OmO 还附带了基于 models.dev 的捆绑能力数据，支持可刷新的本地 models.dev 缓存，并在没有确切元数据时回退到启发式系列检测和别名规则。`bunx oh-my-opencode doctor` 显示能力诊断，并在配置的模型依赖于兼容性回退时发出警告。


#### 代理提供商链

| 代理                  | 默认模型            | 提供商优先级                                                                 |
| --------------------- | ------------------- | ---------------------------------------------------------------------------- |
| **Sisyphus**          | `claude-opus-4-6`   | `claude-opus-4-6 (max)` → `kimi-k2.5` 通过 OpenCode Go / Kimi 提供商 → `gpt-5.4 (medium)` → `glm-5` → `big-pickle` |
| **Hephaestus**        | `gpt-5.4`           | `gpt-5.4 (medium)`                                                           |
| **oracle**            | `gpt-5.4`           | `gpt-5.4 (high)` → `gemini-3.1-pro (high)` → `claude-opus-4-6 (max)` → `glm-5` |
| **librarian**         | `minimax-m2.7`      | `opencode-go/minimax-m2.7` → `opencode/minimax-m2.5` → `claude-haiku-4-5` → `gpt-5-nano` |
| **explore**           | `grok-code-fast-1`  | `grok-code-fast-1` → `opencode-go/minimax-m2.7` → `opencode/minimax-m2.5` → `claude-haiku-4-5` → `gpt-5-nano` |
| **multimodal-looker** | `gpt-5.4`           | `gpt-5.4 (medium)` → `kimi-k2.5` → `glm-4.6v` → `gpt-5-nano`                |
| **Prometheus**        | `claude-opus-4-6`   | `claude-opus-4-6 (max)` → `gpt-5.4 (high)` → `glm-5` → `gemini-3.1-pro`     |
| **Metis**             | `claude-opus-4-6`   | `claude-opus-4-6 (max)` → `gpt-5.4 (high)` → `glm-5` → `k2p5`               |
| **Momus**             | `gpt-5.4`           | `gpt-5.4 (xhigh)` → `claude-opus-4-6 (max)` → `gemini-3.1-pro (high)` → `glm-5` |
| **Atlas**             | `claude-sonnet-4-6` | `claude-sonnet-4-6` → `kimi-k2.5` → `gpt-5.4 (medium)` → `minimax-m2.7`     |

#### 类别提供商链

| 类别                   | 默认模型            | 提供商优先级                                                   |
| ---------------------- | ------------------- | -------------------------------------------------------------- |
| **visual-engineering** | `gemini-3.1-pro`    | `gemini-3.1-pro` → `glm-5` → `claude-opus-4-6`                 |
| **ultrabrain**         | `gpt-5.4`           | `gpt-5.4` → `gemini-3.1-pro` → `claude-opus-4-6`               |
| **deep**               | `gpt-5.3-codex`     | `gpt-5.3-codex` → `claude-opus-4-6` → `gemini-3.1-pro`         |
| **artistry**           | `gemini-3.1-pro`    | `gemini-3.1-pro` → `claude-opus-4-6` → `gpt-5.4`               |
| **quick**              | `gpt-5.4-mini`      | `gpt-5.4-mini` → `claude-haiku-4-5` → `gemini-3-flash` → `minimax-m2.7` → `gpt-5-nano` |
| **unspecified-low**    | `claude-sonnet-4-6` | `claude-sonnet-4-6` → `gpt-5.3-codex` → `gemini-3-flash` → `minimax-m2.7` |
| **unspecified-high**   | `claude-opus-4-6`   | `claude-opus-4-6` → `gpt-5.4 (high)` → `glm-5` → `k2p5` → `kimi-k2.5` |
| **writing**            | `gemini-3-flash`    | `gemini-3-flash` → `claude-sonnet-4-6` → `minimax-m2.7`        |

运行 `bunx oh-my-opencode doctor --verbose` 以查看您的配置的有效模型解析。

---

## 任务系统

### 后台任务

控制并行代理执行和并发限制。

```json
{
  "background_task": {
    "defaultConcurrency": 5,
    "staleTimeoutMs": 180000,
    "providerConcurrency": { "anthropic": 3, "openai": 5, "google": 10 },
    "modelConcurrency": { "anthropic/claude-opus-4-6": 2 }
  }
}
```

| 选项                  | 默认值   | 描述                                                                  |
| --------------------- | -------- | --------------------------------------------------------------------- |
| `defaultConcurrency`  | -        | 最大并发任务数（所有提供商）                                          |
| `staleTimeoutMs`      | `180000` | 中断无活动任务（最小值：60000）                                       |
| `providerConcurrency` | -        | 每个提供商的限制（键 = 提供商名称）                                   |
| `modelConcurrency`    | -        | 每个模型的限制（键 = `provider/model`）。覆盖提供商限制。             |

优先级：`modelConcurrency` > `providerConcurrency` > `defaultConcurrency`

### Sisyphus 代理

配置主编排系统。

```json
{
  "sisyphus_agent": {
    "disabled": false,
    "default_builder_enabled": false,
    "planner_enabled": true,
    "replace_plan": true
  }
}
```

| 选项                      | 默认值  | 描述                                                            |
| ------------------------- | ------- | --------------------------------------------------------------- |
| `disabled`                | `false` | 禁用所有 Sisyphus 编排，恢复原始的 build/plan                   |
| `default_builder_enabled` | `false` | 启用 OpenCode-Builder 代理（默认关闭）                          |
| `planner_enabled`         | `true`  | 启用 Prometheus (Planner) 代理                                  |
| `replace_plan`            | `true`  | 将默认的 plan 代理降级为子代理模式                              |

也可以在 `agents` 下使用它们的名称自定义 Sisyphus 代理：`Sisyphus`, `OpenCode-Builder`, `Prometheus (Planner)`, `Metis (Plan Consultant)`。

### Sisyphus 任务

启用 Sisyphus 任务系统以进行跨会话的任务跟踪。

```json
{
  "sisyphus": {
    "tasks": {
      "enabled": false,
      "storage_path": ".sisyphus/tasks",
      "claude_code_compat": false
    }
  }
}
```

| 选项                 | 默认值            | 描述                                       |
| -------------------- | ----------------- | ------------------------------------------ |
| `enabled`            | `false`           | 启用 Sisyphus 任务系统                     |
| `storage_path`       | `.sisyphus/tasks` | 存储路径（相对于项目根目录）               |
| `claude_code_compat` | `false`           | 启用 Claude Code 路径兼容模式              |

---

## 功能

### 技能

技能带来了特定领域的专业知识和嵌入式 MCP。

内置技能：`playwright`, `playwright-cli`, `agent-browser`, `dev-browser`, `git-master`, `frontend-ui-ux`

禁用内置技能：`{ "disabled_skills": ["playwright"] }`

#### 技能配置

```json
{
  "skills": {
    "sources": [
      { "path": "./my-skills", "recursive": true },
      "https://example.com/skill.yaml"
    ],
    "enable": ["my-skill"],
    "disable": ["other-skill"],
    "my-skill": {
      "description": "它的作用",
      "template": "自定义提示词模板",
      "from": "source-file.ts",
      "model": "custom/model",
      "agent": "custom-agent",
      "subtask": true,
      "argument-hint": "使用提示",
      "license": "MIT",
      "compatibility": ">= 3.0.0",
      "metadata": { "author": "Your Name" },
      "allowed-tools": ["read", "bash"]
    }
  }
}
```

| `sources` 选项   | 默认值  | 描述                            |
| ---------------- | ------- | ------------------------------- |
| `path`           | -       | 本地路径或远程 URL              |
| `recursive`      | `false` | 递归进入子目录                  |
| `glob`           | -       | 用于文件选择的 Glob 模式        |

### 钩子

通过 `disabled_hooks` 禁用内置钩子：

```json
{ "disabled_hooks": ["comment-checker"] }
```

可用钩子：`todo-continuation-enforcer`, `context-window-monitor`, `session-recovery`, `session-notification`, `comment-checker`, `grep-output-truncator`, `tool-output-truncator`, `directory-agents-injector`, `directory-readme-injector`, `empty-task-response-detector`, `think-mode`, `anthropic-context-window-limit-recovery`, `rules-injector`, `background-notification`, `auto-update-checker`, `startup-toast`, `keyword-detector`, `agent-usage-reminder`, `non-interactive-env`, `interactive-bash-session`, `compaction-context-injector`, `thinking-block-validator`, `claude-code-hooks`, `ralph-loop`, `preemptive-compaction`, `auto-slash-command`, `sisyphus-junior-notepad`, `no-sisyphus-gpt`, `start-work`, `runtime-fallback`

**注意：**

- `directory-agents-injector` - 在 OpenCode 1.1.37+ 上自动禁用（原生 AGENTS.md 支持）
- `no-sisyphus-gpt` - **不要禁用**。它阻止 Sisyphus 在不兼容的 GPT 模型上运行，同时允许专门的 GPT-5.4 提示词路径。
- `startup-toast` 是 `auto-update-checker` 的子功能。通过将 `startup-toast` 添加到 `disabled_hooks` 来仅禁用 Toast 通知。
- `session-recovery` - 自动从可恢复的会话错误（缺失的工具结果、不可用的工具、思考块违规）中恢复。在恢复期间显示 Toast 通知。启用 `experimental.auto_resume` 以在恢复后自动重试。

### 命令

通过 `disabled_commands` 禁用内置命令：

```json
{ "disabled_commands": ["init-deep", "start-work"] }
```

可用命令：`init-deep`, `ralph-loop`, `ulw-loop`, `cancel-ralph`, `refactor`, `start-work`, `stop-continuation`, `handoff`

### 浏览器自动化

| 提供商                 | 接口      | 安装                                                |
| ---------------------- | --------- | --------------------------------------------------- |
| `playwright` (默认)    | MCP 工具  | 通过 npx 自动安装                                   |
| `agent-browser`        | Bash CLI  | `bun add -g agent-browser && agent-browser install` |

切换提供商：

```json
{ "browser_automation_engine": { "provider": "agent-browser" } }
```

### Tmux 集成

在单独的 tmux 窗格中运行后台子代理。需要在带有 `opencode --port <port>` 的 tmux 中运行。

```json
{
  "tmux": {
    "enabled": true,
    "layout": "main-vertical",
    "main_pane_size": 60,
    "main_pane_min_width": 120,
    "agent_pane_min_width": 40
  }
}
```

| 选项                   | 默认值          | 描述                                                                                |
| ---------------------- | --------------- | ----------------------------------------------------------------------------------- |
| `enabled`              | `false`         | 启用 tmux 窗格生成                                                                  |
| `layout`               | `main-vertical` | `main-vertical` / `main-horizontal` / `tiled` / `even-horizontal` / `even-vertical` |
| `main_pane_size`       | `60`            | 主窗格百分比 (20–80)                                                                |
| `main_pane_min_width`  | `120`           | 主窗格最小列数                                                                      |
| `agent_pane_min_width` | `40`            | 代理窗格最小列数                                                                    |

### Git Master

配置 git 提交行为：

```json
{ "git_master": { "commit_footer": true, "include_co_authored_by": true } }
```

### 注释检查器

自定义注释质量检查器：

```json
{
  "comment_checker": {
    "custom_prompt": "你的消息。使用 {{comments}} 占位符。"
  }
}
```

### 通知

强制启用会话通知：

```json
{ "notification": { "force_enable": true } }
```

`force_enable` (`false`) - 即使检测到外部通知插件也强制使用 session-notification。

### MCP

内置 MCP（默认启用）：`websearch` (Exa AI), `context7` (库文档), `grep_app` (GitHub 代码搜索)。

```json
{ "disabled_mcps": ["websearch", "context7", "grep_app"] }
```

### LSP

配置语言服务器协议集成：

```json
{
  "lsp": {
    "typescript-language-server": {
      "command": ["typescript-language-server", "--stdio"],
      "extensions": [".ts", ".tsx"],
      "priority": 10,
      "env": { "NODE_OPTIONS": "--max-old-space-size=4096" },
      "initialization": {
        "preferences": { "includeInlayParameterNameHints": "all" }
      }
    },
    "pylsp": { "disabled": true }
  }
}
```

| 选项             | 类型    | 描述                                 |
| ---------------- | ------- | ------------------------------------ |
| `command`        | array   | 启动 LSP 服务器的命令                |
| `extensions`     | array   | 文件扩展名（例如 `[".ts"]`）         |
| `priority`       | number  | 多个服务器匹配时的优先级             |
| `env`            | object  | 环境变量                             |
| `initialization` | object  | 传递给服务器的初始化选项             |
| `disabled`       | boolean | 禁用此服务器                         |

---

## 高级

### 运行时回退

在 API 错误时自动切换到备份模型。

**简单配置**（使用默认值启用/禁用）：

```json
{ "runtime_fallback": true }
{ "runtime_fallback": false }
```

**高级配置**（完全控制）：

```json
{
  "runtime_fallback": {
    "enabled": true,
    "retry_on_errors": [400, 429, 503, 529],
    "max_fallback_attempts": 3,
    "cooldown_seconds": 60,
    "timeout_seconds": 30,
    "notify_on_fallback": true
  }
}
```

| 选项                    | 默认值              | 描述                                                                                                                           |
| ----------------------- | ------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| `enabled`               | `false`             | 启用运行时回退                                                                                                                 |
| `retry_on_errors`       | `[400,429,503,529]` | 触发回退的 HTTP 状态码。也处理分类的提供商密钥错误。                                                                           |
| `max_fallback_attempts` | `3`                 | 每个会话的最大回退尝试次数 (1–20)                                                                                              |
| `cooldown_seconds`      | `60`                | 重试失败模型之前的秒数                                                                                                         |
| `timeout_seconds`       | `30`                | 强制进行下一次回退之前的秒数。**设置为 `0` 可禁用基于超时的升级和提供商重试消息检测。**                                        |
| `notify_on_fallback`    | `true`              | 模型切换时的 Toast 通知                                                                                                        |

为每个代理或类别定义 `fallback_models`：

```json
{
  "agents": {
    "sisyphus": {
      "model": "anthropic/claude-opus-4-6",
      "fallback_models": [
        "openai/gpt-5.4",
        {
          "model": "google/gemini-3.1-pro",
          "variant": "high"
        }
      ]
    }
  }
}
```

`fallback_models` 还支持对象样式的条目，以便你可以将设置附加到特定的回退模型：

```json
{
  "agents": {
    "sisyphus": {
      "model": "anthropic/claude-opus-4-6",
      "fallback_models": [
        "openai/gpt-5.4",
        {
          "model": "anthropic/claude-sonnet-4-6",
          "variant": "high",
          "thinking": { "type": "enabled", "budgetTokens": 12000 }
        },
        {
          "model": "openai/gpt-5.3-codex",
          "reasoningEffort": "high",
          "temperature": 0.2,
          "top_p": 0.95,
          "maxTokens": 8192
        }
      ]
    }
  }
}
```

允许混合数组，因此字符串条目和对象条目可以一起出现在同一个回退链中。

#### 对象样式的 `fallback_models`

对象条目使用以下形状：

| 字段            | 类型   | 描述                                                                                 |
| --------------- | ------ | ------------------------------------------------------------------------------------ |
| `model`         | string | 回退模型 ID。当 OmO 可以继承当前/默认提供商时，提供商前缀是可选的。                  |
| `variant`       | string | 此回退条目的显式变体覆盖。                                                           |
| `reasoningEffort`| string | 此回退条目的显式 OpenAI 推理努力覆盖。                                               |
| `temperature`   | number | 如果此回退模型变为活动状态，则应用的温度。                                           |
| `top_p`         | number | 如果此回退模型变为活动状态，则应用的 Top-p。                                         |
| `maxTokens`     | number | 如果此回退模型变为活动状态，则应用的最大响应 token 数。                              |
| `thinking`      | object | 如果此回退模型变为活动状态，则应用的 Anthropic 思考配置。                            |

每个模型的设置**仅用于回退**。只有在实际选择了该特定回退模型时，它们才会被提升，因此当主模型成功解析时，它们不会覆盖你的主模型设置。

`thinking` 使用与普通代理/类别选项相同的形状：

| 字段           | 类型   | 描述                                |
| -------------- | ------ | ----------------------------------- |
| `type`         | string | `enabled` 或 `disabled`             |
| `budgetTokens` | number | 可选的 Anthropic 思考预算           |

当 OmO 可以从当前/默认提供商推断出提供商前缀时，对象条目也可以省略提供商前缀。如果你在 `model` 中提供了内联变体语法并提供了一个显式的 `variant` 字段，则显式的 `variant` 字段胜出。

#### 完整示例

**1. 简单的字符串链**

当你只需要一个有序的回退链时使用字符串：

```json
{
  "agents": {
    "atlas": {
      "model": "anthropic/claude-sonnet-4-6",
      "fallback_models": [
        "anthropic/claude-haiku-4-5",
        "openai/gpt-5.4",
        "google/gemini-3.1-pro"
      ]
    }
  }
}
```

**2. 同一提供商简写**

如果主模型已经建立了提供商，回退条目可以省略前缀：

```json
{
  "agents": {
    "atlas": {
      "model": "openai/gpt-5.4",
      "fallback_models": [
        "gpt-5.4-mini",
        {
          "model": "gpt-5.3-codex",
          "reasoningEffort": "medium",
          "maxTokens": 4096
        }
      ]
    }
  }
}
```

在这个例子中，OmO 将 `gpt-5.4-mini` 和 `gpt-5.3-codex` 视为 OpenAI 回退条目，因为当前/默认提供商已经是 `openai`。

**3. 混合跨提供商链**

当只有某些回退模型需要特殊设置时，混合使用字符串条目和对象条目：

```json
{
  "agents": {
    "sisyphus": {
      "model": "anthropic/claude-opus-4-6",
      "fallback_models": [
        "openai/gpt-5.4",
        {
          "model": "anthropic/claude-sonnet-4-6",
          "variant": "high",
          "thinking": { "type": "enabled", "budgetTokens": 12000 }
        },
        {
          "model": "google/gemini-3.1-pro",
          "variant": "high"
        }
      ]
    }
  }
}
```

**4. 类别级别的回退链**

`fallback_models` 在 `categories` 下的工作方式相同：

```json
{
  "categories": {
    "deep": {
      "model": "openai/gpt-5.3-codex",
      "fallback_models": [
        {
          "model": "openai/gpt-5.4",
          "reasoningEffort": "xhigh",
          "maxTokens": 12000
        },
        {
          "model": "anthropic/claude-opus-4-6",
          "variant": "max",
          "temperature": 0.2
        },
        "google/gemini-3.1-pro(high)"
      ]
    }
  }
}
```

**5. 包含所有支持字段的完整对象条目**

这展示了在一个地方所有支持的对象样式参数：

```json
{
  "agents": {
    "oracle": {
      "model": "openai/gpt-5.4",
      "fallback_models": [
        {
          "model": "openai/gpt-5.3-codex(low)",
          "variant": "xhigh",
          "reasoningEffort": "high",
          "temperature": 0.3,
          "top_p": 0.9,
          "maxTokens": 8192,
          "thinking": {
            "type": "disabled"
          }
        }
      ]
    }
  }
}
```

在这个例子中，显式的 `"variant": "xhigh"` 覆盖了 `"model"` 中的内联 `(low)` 后缀。

这最后一个例子是一个**完整的形状参考**。在实际配置中，请首选适合提供商的设置：

- 对于 OpenAI 推理模型，使用 `reasoningEffort`
- 对于具有思考能力的 Anthropic 模型，使用 `thinking`
- 仅当该回退模型支持时，才使用 `variant`, `temperature`, `top_p` 和 `maxTokens`

### 模型能力

OmO 可以在启动时刷新本地的 models.dev 能力快照。此缓存由 `model_capabilities` 控制。

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

| 选项                    | 默认行为                                     | 描述                                      |
| ----------------------- | -------------------------------------------- | ----------------------------------------- |
| `enabled`               | 启用，除非显式设置为 `false`                 | 模型能力刷新行为的主开关                  |
| `auto_refresh_on_start` | 在启动时刷新，除非显式设置为 `false`         | 在启动检查期间刷新本地 models.dev 缓存    |
| `refresh_timeout_ms`    | `5000`                                       | 启动刷新尝试的超时时间                    |
| `source_url`            | `https://models.dev/api.json`                | 覆盖 models.dev 源 URL                    |

注意：

- 启动刷新通过自动更新检查器钩子运行。
- 可以通过 `bunx oh-my-opencode refresh-model-capabilities` 手动刷新。
- 当 OmO 解析能力以进行兼容性检查时，提供商运行时元数据仍然优先。

### 哈希行编辑

用使用 `LINE#ID` 引用的基于哈希的版本替换内置的 `Edit` 工具，以防止陈旧的行编辑。默认情况下禁用。

```json
{ "hashline_edit": true }
```

启用后，两个配套钩子将处于活动状态：`hashline-read-enhancer`（注释 Read 输出）和 `hashline-edit-diff-enhancer`（显示差异）。通过设置 `hashline_edit: true` 选择加入。如果需要，可以通过 `disabled_hooks` 单独禁用配套钩子。

### 实验性

```json
{
  "experimental": {
    "truncate_all_tool_outputs": false,
    "aggressive_truncation": false,
    "auto_resume": false,
    "disable_omo_env": false,
    "task_system": false,
    "dynamic_context_pruning": {
      "enabled": false,
      "notification": "detailed",
      "turn_protection": { "enabled": true, "turns": 3 },
      "protected_tools": [
        "task",
        "todowrite",
        "todoread",
        "lsp_rename",
        "session_read",
        "session_write",
        "session_search"
      ],
      "strategies": {
        "deduplication": { "enabled": true },
        "supersede_writes": { "enabled": true, "aggressive": false },
        "purge_errors": { "enabled": true, "turns": 5 }
      }
    }
  }
}
```

| 选项                                     | 默认值     | 描述                                                                                 |
| ---------------------------------------- | ---------- | ------------------------------------------------------------------------------------ |
| `truncate_all_tool_outputs`              | `false`    | 截断所有工具输出（不仅仅是白名单内的）                                               |
| `aggressive_truncation`                  | `false`    | 超过 token 限制时积极截断                                                            |
| `auto_resume`                            | `false`    | 思考块恢复后自动恢复                                                                 |
| `disable_omo_env`                        | `false`    | 禁用自动注入的 `<omo-env>` 块（日期/时间/语言环境）。提高缓存命中率。                |
| `task_system`                            | `false`    | 启用 Sisyphus 任务系统                                                               |
| `dynamic_context_pruning.enabled`        | `false`    | 自动修剪旧的工具输出以管理上下文窗口                                                 |
| `dynamic_context_pruning.notification`   | `detailed` | 修剪通知：`off` / `minimal` / `detailed`                                             |
| `turn_protection.turns`                  | `3`        | 保护免受修剪的最近回合数 (1–10)                                                      |
| `strategies.deduplication`               | `true`     | 删除重复的工具调用                                                                   |
| `strategies.supersede_writes`            | `true`     | 当文件随后被读取时，修剪写入输入                                                     |
| `strategies.supersede_writes.aggressive` | `false`    | 如果存在**任何**后续读取，则修剪任何写入                                             |
| `strategies.purge_errors.turns`          | `5`        | 修剪出错工具输入之前的回合数                                                         |

---

## 参考

### 环境变量

| 变量                  | 描述                                                              |
| --------------------- | ----------------------------------------------------------------- |
| `OPENCODE_CONFIG_DIR` | 覆盖 OpenCode 配置目录（对配置文件隔离很有用）                    |

### 特定于提供商

#### Google Auth

安装 [`opencode-antigravity-auth`](https://github.com/NoeFabris/opencode-antigravity-auth) 用于 Google Gemini。提供多帐户负载均衡、双配额和基于变体的思考。

#### Ollama

**必须**禁用流式传输以避免 JSON 解析错误：

```json
{
  "agents": {
    "explore": { "model": "ollama/qwen3-coder", "stream": false }
  }
}
```

常用模型：`ollama/qwen3-coder`, `ollama/ministral-3:14b`, `ollama/lfm2.5-thinking`

有关 `JSON Parse error: Unexpected EOF` 问题，请参阅 [Ollama 故障排除](../troubleshooting/ollama.md)。
