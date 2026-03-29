# src/features/mcp-oauth/ — MCP 服务器的 OAuth 2.0 + PKCE + DCR

**生成日期：** 2026-03-06

## 概览

18 个文件。用于需要身份验证的 MCP 服务器的完整 OAuth 2.0 授权流程。实现了 PKCE (RFC 7636)、动态客户端注册 (DCR, RFC 7591) 和资源指示器 (RFC 8707)。由 `bunx oh-my-opencode mcp-oauth login` 使用。

## 授权流程

```
1. discovery.ts → 获取 /.well-known/oauth-authorization-server
2. dcr.ts → 动态客户端注册 (如果服务器支持)
3. oauth-authorization-flow.ts → 生成 PKCE 验证器/挑战
4. callback-server.ts → 在随机端口上的本地 HTTP 服务器，用于重定向
5. 打开浏览器 → 授权 URL
6. callback-server.ts → 接收 code + state
7. provider.ts → 用 code 交换令牌 (使用 PKCE 验证器)
8. storage.ts → 将令牌持久化到 ~/.config/opencode/mcp-oauth/
9. step-up.ts → 如果初始令牌权限不足，处理升级身份验证
```

## 关键文件

| 文件 | 目的 |
|------|---------|
| `oauth-authorization-flow.ts` | PKCE 助手：`generateCodeVerifier()`, `generateCodeChallenge()`, `buildAuthorizationUrl()` |
| `callback-server.ts` | 本地 HTTP 重定向服务器 — 监听 OAuth 回调 |
| `provider.ts` | `OAuthProvider` — 令牌交换、刷新、撤销 |
| `discovery.ts` | 从 well-known 端点获取并解析 OAuth 服务器元数据 |
| `dcr.ts` | 动态客户端注册 — 将此应用注册到 OAuth 服务器 |
| `resource-indicator.ts` | RFC 8707 资源指示器处理 |
| `step-up.ts` | 处理升级身份验证挑战 |
| `storage.ts` | 将令牌持久化到 `~/.config/opencode/mcp-oauth/{server-hash}.json` |
| `schema.ts` | OAuth 服务器元数据、令牌响应、DCR 的 Zod 模式 |

## PKCE 实现

- Code verifier (代码验证器): 32 字节随机数 → base64url (无填充)
- Code challenge (代码挑战): SHA-256(verifier) → base64url
- 方法: `S256`

## 令牌存储

位置：`~/.config/opencode/mcp-oauth/` — 每个 MCP 服务器一个 JSON 文件 (由服务器 URL 哈希作为键)。
字段：`access_token`, `refresh_token`, `expires_at`, `client_id`。

## CLI 命令

```bash
bunx oh-my-opencode mcp-oauth login <server-url>   # 完整的 PKCE 流程
bunx oh-my-opencode mcp-oauth logout <server-url>  # 撤销并删除令牌
bunx oh-my-opencode mcp-oauth status               # 列出已存储的令牌
```
