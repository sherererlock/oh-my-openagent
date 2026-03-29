# src/tools/lsp/ — LSP 工具实现

**生成日期:** 2026-03-06

## 概览

33 个文件。完整的 LSP (语言服务器协议, Language Server Protocol) 客户端栈作为 6 个工具公开。自定义实现，负责管理服务器进程、打开文件和转发请求 — **不**委托给 OpenCode 内置的 LSP。

## 工具公开

| 工具 | 文件 | 它的作用 |
|------|------|--------------|
| `lsp_goto_definition` | `goto-definition-tool.ts` | 跳转到符号定义 |
| `lsp_find_references` | `find-references-tool.ts` | 符号的所有用法 |
| `lsp_symbols` | `symbols-tool.ts` | 文档大纲或工作区符号搜索 |
| `lsp_diagnostics` | `diagnostics-tool.ts` | 来自语言服务器的错误/警告 |
| `lsp_prepare_rename` | `rename-tools.ts` | 在应用之前验证重命名 |
| `lsp_rename` | `rename-tools.ts` | 跨工作区应用安全的重命名 |

所有 6 个都是直接的 `ToolDefinition` 对象（不是工厂函数） — 直接在 `tool-registry.ts` 中注册。

## 架构

```
tools.ts (6 个 ToolDefinition 导出)
  ↓ 使用
LspClientWrapper (lsp-client-wrapper.ts)
  ↓ 包装
LSPClient (lsp-client.ts) 扩展自 LSPClientConnection (lsp-client-connection.ts)
  ↓ 通过以下方式通信
LSPClientTransport (lsp-client-transport.ts)
  ↓ 对话目标
LSPProcess (lsp-process.ts) — 生成服务器二进制文件
```

## 关键文件

| 文件 | 用途 |
|------|---------|
| `lsp-client-wrapper.ts` | 高级入口：解析服务器，打开文件，运行请求 |
| `lsp-client.ts` | `LSPClient` — 文件跟踪，文档同步 (`didOpen`/`didChange`) |
| `lsp-client-connection.ts` | JSON-RPC 请求/响应/通知层 |
| `lsp-client-transport.ts` | stdin/stdout 字节流帧 |
| `lsp-process.ts` | 衍生 (Spawn) + 清理 LSP 服务器进程 |
| `lsp-manager-process-cleanup.ts` | 退出时回收孤儿 LSP 进程 |
| `lsp-manager-temp-directory-cleanup.ts` | 清理某些服务器使用的临时目录 |
| `server-definitions.ts` | 40+ 个内置服务器，与 OpenCode 的 `server.ts` 同步 |
| `server-config-loader.ts` | 从 `.opencode/lsp.json` 加载自定义服务器配置 |
| `server-resolution.ts` | 解析哪个服务器处理某种文件扩展名 |
| `server-installation.ts` | 检测缺失的二进制文件，提供安装提示 |
| `language-mappings.ts` | 扩展名 → 语言 ID 映射 |
| `lsp-formatters.ts` | 将 LSP 响应格式化为人类可读的字符串 |
| `workspace-edit.ts` | 将 `WorkspaceEdit` 结果应用到磁盘（用于重命名） |
| `types.ts` | `LSPServerConfig`、`Position`、`Range`、`Location`、`Diagnostic` 等 |

## 服务器解析

```
文件 → 扩展名 (.ts) → language-mappings → 服务器 ID (typescript)
  → server-resolution: 检查用户配置 (.opencode/lsp.json) → 回退到 server-definitions.ts
  → server-installation: 验证二进制文件是否存在 (如果不存在则发出带有安装提示的警告)
  → LSPProcess.spawn(command[])
```

## 注意事项

- 在任何 LSP 请求之前，必须通过 `didOpen` 打开文件 — `LSPClient.openFile()` 处理此操作
- 在发送请求之前，`didOpen` 之后会有 1 秒的延迟用于服务器初始化
- `lsp_servers` 工具已被移除 — 它与 OpenCode 内置的 `LspServers` 工具重复
- 与 OpenCode 的 `server.ts` 同步 — 添加服务器时，请先检查上游
