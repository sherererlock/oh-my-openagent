# CLI 参考

发布的 `oh-my-opencode` CLI 的完整参考。在重命名过渡期间，OpenCode 插件注册现在首选 `opencode.json` 中的 `oh-my-openagent`。

## 基本用法

```bash
# 显示帮助
bunx oh-my-opencode

# 或使用 npx
npx oh-my-opencode
```

## 命令

| 命令                          | 描述                                                   |
| ----------------------------- | ------------------------------------------------------ |
| `install`                     | 交互式设置向导                                         |
| `doctor`                      | 环境诊断和健康检查                                     |
| `run`                         | 带有任务完成强制执行的 OpenCode 会话运行器             |
| `get-local-version`           | 显示本地版本信息并检查更新                             |
| `refresh-model-capabilities`  | 刷新基于 models.dev 的缓存模型能力                     |
| `version`                     | 显示版本信息                                           |
| `mcp oauth`                   | MCP OAuth 身份验证管理                                 |

---

## install

用于初始 Oh My OpenCode 设置的交互式安装工具。提供基于 `@clack/prompts` 的 TUI。

### 用法

```bash
bunx oh-my-opencode install
```

### 安装过程

1. **订阅选择**：选择你实际拥有的提供商和订阅
2. **插件注册**：在 OpenCode 设置中注册 `oh-my-openagent`，或在兼容窗口期间升级旧的 `oh-my-opencode` 条目
3. **配置文件创建**：将生成的 OmO 配置写入活动 OpenCode 配置目录中的 `oh-my-opencode.json`
4. **身份验证提示**：显示你选择的提供商的 `opencode auth login` 步骤，除非设置了 `--skip-auth`

### 选项

| 选项 | 描述 |
| ------ | ----------- |
| `--no-tui` | 在无 TUI 的非交互模式下运行 |
| `--claude <no\|yes\|max20>` | Claude 订阅模式 |
| `--openai <no\|yes>` | OpenAI / ChatGPT 订阅 |
| `--gemini <no\|yes>` | Gemini 集成 |
| `--copilot <no\|yes>` | GitHub Copilot 订阅 |
| `--opencode-zen <no\|yes>` | OpenCode Zen 访问权限 |
| `--zai-coding-plan <no\|yes>` | Z.ai Coding Plan 订阅 |
| `--kimi-for-coding <no\|yes>` | Kimi for Coding 订阅 |
| `--opencode-go <no\|yes>` | OpenCode Go 订阅 |
| `--skip-auth` | 跳过身份验证设置提示 |

---

## doctor

诊断你的环境以确保 Oh My OpenCode 正常运行。当前的检查分为系统、配置、工具和模型。

doctor 命令会检测常见问题，包括：
- `opencode.json` 中的旧插件条目引用（当仍在使用 `oh-my-opencode` 而不是 `oh-my-openagent` 时发出警告）
- 配置文件有效性和 JSONC 解析错误
- 模型解析和回退链验证
- 缺失或配置错误的 MCP 服务器

### 用法

```bash
bunx oh-my-opencode doctor
```

### 诊断类别

| 类别              | 检查项                                                                               |
| ----------------- | ------------------------------------------------------------------------------------ |
| **System**        | OpenCode 二进制文件、版本 (>= 1.0.150)、插件注册、旧包名警告                         |
| **Config**        | 配置文件有效性、JSONC 解析、Zod 模式验证                                             |
| **Tools**         | AST-Grep、LSP 服务器、GitHub CLI、MCP 服务器                                         |
| **Models**        | 模型能力缓存、模型解析、代理/类别覆盖、可用性                                        |

### 选项

| 选项         | 描述                                      |
| ------------ | ----------------------------------------- |
| `--status`   | 显示紧凑的系统仪表板                      |
| `--verbose`  | 显示详细的诊断信息                        |
| `--json`     | 以 JSON 格式输出结果                      |

### 示例输出

```
oh-my-opencode doctor

┌──────────────────────────────────────────────────┐
│  Oh-My-OpenAgent Doctor                           │
└──────────────────────────────────────────────────┘

System
  ✓ OpenCode version: 1.0.155 (>= 1.0.150)
  ✓ Plugin registered in opencode.json

Config
  ✓ oh-my-opencode.jsonc is valid
  ✓ Model resolution: all agents have valid fallback chains
  ⚠ categories.visual-engineering: using default model

Tools
  ✓ AST-Grep available
  ✓ LSP servers configured

Models
  ✓ 11 agents, 8 categories, 0 overrides
  ⚠ Some configured models rely on compatibility fallback

Summary: 10 passed, 1 warning, 0 failed
```
---

## run

运行带有 todo/后台任务完成强制执行的 opencode。与 'opencode run' 不同，此命令会等待直到所有 todo 都完成或取消，并且所有子会话（后台任务）都处于空闲状态。

### 用法

```bash
bunx oh-my-opencode run <message>
```

### 选项

| 选项                  | 描述                                                                |
| --------------------- | ------------------------------------------------------------------- |
| `-a, --agent <name>`  | 要使用的代理 (默认：来自 CLI/env/config，回退：Sisyphus)            |
| `-m, --model <provider/model>` | 模型覆盖 (例如 anthropic/claude-sonnet-4)                  |
| `-d, --directory <path>` | 工作目录                                                         |
| `-p, --port <port>`   | 服务器端口 (如果端口已在使用则附加)                                 |
| `--attach <url>`      | 附加到现有的 opencode 服务器 URL                                    |
| `--on-complete <command>` | 完成后运行的 Shell 命令                                         |
| `--json`              | 将结构化 JSON 结果输出到 stdout                                     |
| `--no-timestamp`      | 在运行输出中禁用时间戳前缀                                          |
| `--verbose`           | 显示完整的事件流 (默认：仅消息/工具)                                |
| `--session-id <id>`   | 恢复现有会话而不是创建新会话                                        |

---

## get-local-version

显示当前安装的版本并检查更新。

### 用法

```bash
bunx oh-my-opencode get-local-version
```

### 选项

| 选项              | 描述                                           |
| ----------------- | ---------------------------------------------- |
| `-d, --directory` | 检查配置的工作目录                             |
| `--json`          | 以 JSON 格式输出，用于脚本                     |

### 输出

显示：
- 当前安装的版本
- npm 上最新的可用版本
- 你是否是最新的
- 特殊模式（本地开发、固定版本）

---

## version

显示版本信息。

### 用法

```bash
bunx oh-my-opencode version
```

---

## mcp oauth

管理远程 MCP 服务器的 OAuth 2.1 身份验证。

### 用法

```bash
# 登录到受 OAuth 保护的 MCP 服务器
bunx oh-my-opencode mcp oauth login <server-name> --server-url https://api.example.com

# 使用显式客户端 ID 和范围登录
bunx oh-my-opencode mcp oauth login my-api --server-url https://api.example.com --client-id my-client --scopes "read,write"

# 删除存储的 OAuth 令牌
bunx oh-my-opencode mcp oauth logout <server-name>

# 检查 OAuth 令牌状态
bunx oh-my-opencode mcp oauth status [server-name]
```

### 选项

| 选项                 | 描述                                                                      |
| -------------------- | ------------------------------------------------------------------------- |
| `--server-url <url>` | MCP 服务器 URL (登录必需)                                                 |
| `--client-id <id>`   | OAuth 客户端 ID (如果服务器支持动态客户端注册则可选)                      |
| `--scopes <scopes>`  | 逗号分隔的 OAuth 范围                                                     |

### 令牌存储

令牌存储在 `~/.config/opencode/mcp-oauth.json` 中，权限为 `0600`（仅所有者读/写）。键格式：`{serverHost}/{resource}`。

---

## 配置文件

运行时首先加载用户配置作为基本配置，然后将项目配置合并到其上：

1. **项目级**: `.opencode/oh-my-openagent.jsonc`, `.opencode/oh-my-openagent.json`, `.opencode/oh-my-opencode.jsonc`, 或 `.opencode/oh-my-opencode.json`
2. **用户级**: `~/.config/opencode/oh-my-openagent.jsonc`, `~/.config/opencode/oh-my-openagent.json`, `~/.config/opencode/oh-my-opencode.jsonc`, 或 `~/.config/opencode/oh-my-opencode.json`

**命名说明**: 发布的包和二进制文件仍然是 `oh-my-opencode`。在 `opencode.json` 中，兼容层现在首选插件条目 `oh-my-openagent`。插件配置加载识别 `oh-my-openagent.*` 和旧的 `oh-my-opencode.*` 基本名称。如果同一个目录中存在这两个基本名称，目前以旧的 `oh-my-opencode.*` 文件为准。

### 文件名兼容性

支持 `.jsonc` 和 `.json` 扩展名。首选 JSONC（带注释的 JSON），因为它允许：
- 注释（`//` 和 `/* */` 样式）
- 数组和对象中的尾随逗号

如果同一个目录中同时存在 `.jsonc` 和 `.json`，则以 `.jsonc` 文件为准。

### JSONC 支持

配置文件支持 **JSONC（带注释的 JSON）** 格式。你可以使用注释和尾随逗号。

```jsonc
{
  // 代理配置
  "sisyphus_agent": {
    "disabled": false,
    "planner_enabled": true,
  },

  /* 类别定制 */
  "categories": {
    "visual-engineering": {
      "model": "google/gemini-3.1-pro",
    },
  },
}
```

---

## 故障排除

### "OpenCode version too old" 错误

```bash
# 更新 OpenCode
npm install -g opencode@latest
# 或
bun install -g opencode@latest
```

### "Plugin not registered" 错误

```bash
# 重新安装插件
bunx oh-my-opencode install
```

### Doctor 检查失败

```bash
# 使用详细信息进行诊断
bunx oh-my-opencode doctor --verbose

# 显示紧凑的系统仪表板
bunx oh-my-opencode doctor --status

# 用于脚本的 JSON 输出
bunx oh-my-opencode doctor --json
```

### "Using legacy package name" 警告

如果 doctor 在 `opencode.json` 中找到旧的插件条目 `oh-my-opencode`，它会发出警告。将插件数组更新为规范的 `oh-my-openagent` 条目：

```bash
# 在用户配置中替换旧的插件条目
jq '.plugin = (.plugin // [] | map(if . == "oh-my-opencode" then "oh-my-openagent" else . end))' \
  ~/.config/opencode/opencode.json > /tmp/opencode.json && mv /tmp/opencode.json ~/.config/opencode/opencode.json
```
---

## refresh-model-capabilities

从 models.dev 刷新缓存的模型能力快照。这会更新用于能力解析和兼容性诊断的本地缓存。

### 用法

```bash
bunx oh-my-opencode refresh-model-capabilities
```

### 选项

| 选项              | 描述                                                |
| ----------------- | --------------------------------------------------- |
| `-d, --directory` | 读取 oh-my-opencode 配置的工作目录                  |
| `--source-url <url>` | 覆盖 models.dev 源 URL                              |
| `--json`          | 以 JSON 格式输出刷新摘要                            |

### 配置

在你的插件配置中配置自动刷新行为：

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

---

## 非交互模式

使用 JSON 输出进行 CI 或脚本化诊断。

```bash
# 在 CI 环境中运行 doctor
bunx oh-my-opencode doctor --json

# 将结果保存到文件
bunx oh-my-opencode doctor --json > doctor-report.json
```

---

## 开发者信息

### CLI 结构

```
src/cli/
├── cli-program.ts        # 基于 Commander.js 的主入口
├── install.ts            # 基于 @clack/prompts 的 TUI 安装程序
├── config-manager/       # JSONC 解析，多源配置管理
│   └── *.ts
├── doctor/               # 健康检查系统
│   ├── index.ts          # Doctor 命令入口
│   └── checks/           # 17+ 个独立的检查模块
├── run/                  # 会话运行器
│   └── *.ts
└── mcp-oauth/            # OAuth 管理命令
    └── *.ts
```

### 添加新的 Doctor 检查

创建 `src/cli/doctor/checks/my-check.ts`:

```typescript
import type { DoctorCheck } from "../types";

export const myCheck: DoctorCheck = {
  name: "my-check",
  category: "environment",
  check: async () => {
    // 检查逻辑
    const isOk = await someValidation();

    return {
      status: isOk ? "pass" : "fail",
      message: isOk ? "一切看起来都很好" : "出了点问题",
    };
  },
};
```

在 `src/cli/doctor/checks/index.ts` 中注册：

```typescript
export { myCheck } from "./my-check";
```
