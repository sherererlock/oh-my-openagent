# src/cli/config-manager/ — CLI 安装实用工具

**生成时间:** 2026-03-06

## 概览

共 20 个文件。提供用于 `install` 命令的无状态实用函数。处理 OpenCode 配置操作、供应商配置、JSONC 操作、二进制检测以及 npm 注册表查询。没有类 — 只有扁平的实用函数集合。

## 文件目录

| 文件 | 用途 |
|------|---------|
| `add-plugin-to-opencode-config.ts` | 在 `.opencode/opencode.json` 的 plugin 数组中注册 `oh-my-opencode` |
| `add-provider-config.ts` | 将供应商 API 密钥添加到 OpenCode 配置（用户级） |
| `antigravity-provider-configuration.ts` | 处理 Antigravity 供应商设置（特殊情况） |
| `auth-plugins.ts` | 检测每个供应商的认证插件要求（oauth vs 密钥） |
| `bun-install.ts` | 运行 `bun install` / `npm install` 进行插件设置 |
| `config-context.ts` | `ConfigContext` — 跨安装步骤的共享配置状态 |
| `deep-merge-record.ts` | 用于 JSONC 配置对象的深度合并工具 |
| `detect-current-config.ts` | 读取现有 OpenCode 配置，检测已安装的插件 |
| `ensure-config-directory-exists.ts` | 如果缺少 `.opencode/` 目录则创建它 |
| `format-error-with-suggestion.ts` | 使用可操作的建议格式化错误信息 |
| `generate-omo-config.ts` | 根据安装选择生成 `oh-my-opencode.jsonc` |
| `jsonc-provider-editor.ts` | 读写 JSONC 文件并保留注释 |
| `npm-dist-tags.ts` | 从 npm 注册表获取最新版本 (dist-tags) |
| `opencode-binary.ts` | 检测 OpenCode 二进制位置，验证其是否已安装 |
| `opencode-config-format.ts` | OpenCode 配置格式常量和类型保护 |
| `parse-opencode-config-file.ts` | 解析带有降级处理的 opencode.json/opencode.jsonc |
| `plugin-name-with-version.ts` | 解析用于安装的 `oh-my-opencode@X.Y.Z` |
| `write-omo-config.ts` | 将生成的配置写入 `.opencode/oh-my-opencode.jsonc` |

## 使用模式

函数由 `src/cli/install.ts` / `src/cli/tui-installer.ts` 按顺序调用：

```
1. ensure-config-directory-exists
2. detect-current-config (检查已设置的内容)
3. opencode-binary (验证 opencode 是否已安装)
4. npm-dist-tags (获取最新版本)
5. generate-omo-config (根据用户选择构建配置)
6. write-omo-config
7. add-plugin-to-opencode-config
8. add-provider-config (针对每个选定的供应商)
9. bun-install
```

## 注意事项

- 所有函数都是纯函数 / 无状态的（磁盘 I/O 除外）— 没有共享的模块状态
- `jsonc-provider-editor.ts` 使用了保留注释的 JSONC 库 — **切勿**对 JSONC 文件使用 `JSON.parse`
- `opencode-binary.ts` 会搜索 PATH 环境变量 + 常见的安装位置（如 `.local/bin`, `~/.bun/bin` 等）
