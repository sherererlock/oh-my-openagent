---
name: agent-browser
description: 用于 Web 测试、表单填写、截图和数据提取的自动化浏览器交互工具。当用户需要导航网站、与网页交互、填写表单、截图、测试 Web 应用程序或从网页提取信息时使用。
---

# 使用 agent-browser 进行浏览器自动化

## 快速开始

```bash
agent-browser open <url>        # 导航到页面
agent-browser snapshot -i       # 获取带有引用 (refs) 的交互式元素
agent-browser click @e1         # 通过引用点击元素
agent-browser fill @e2 "text"   # 通过引用填写输入框
agent-browser close             # 关闭浏览器
```

## 核心工作流

1. 导航: `agent-browser open <url>`
2. 快照: `agent-browser snapshot -i` (返回带有类似 `@e1`, `@e2` 引用的元素)
3. 交互: 使用快照中的引用进行操作
4. 重新快照: 在导航或 DOM 发生重大更改后重新生成快照

## 命令

### 导航 (Navigation)
```bash
agent-browser open <url>      # 导航到 URL (别名: goto, navigate)
agent-browser back            # 后退
agent-browser forward         # 前进
agent-browser reload          # 重新加载页面
agent-browser close           # 关闭浏览器 (别名: quit, exit)
```

### 快照/页面分析 (Snapshot)
```bash
agent-browser snapshot            # 完整的可访问性树
agent-browser snapshot -i         # 仅交互式元素 (推荐)
agent-browser snapshot -i -C      # 包含支持鼠标交互的元素 (带有 onclick 的 div 等)
agent-browser snapshot -c         # 紧凑模式 (移除空的结构性元素)
agent-browser snapshot -d 3       # 限制深度为 3
agent-browser snapshot -s "#main" # 范围限定在 CSS 选择器内
agent-browser snapshot -i -c -d 5 # 组合选项
```

`-C` 标志对于使用自定义可点击元素（div、span）代替标准按钮/链接的现代 Web 应用程序非常有用。

### 交互 (Interactions) (使用快照中的 @refs)
```bash
agent-browser click @e1           # 点击 (使用 --new-tab 在新标签页中打开)
agent-browser dblclick @e1        # 双击
agent-browser focus @e1           # 聚焦元素
agent-browser fill @e2 "text"     # 清除并输入
agent-browser type @e2 "text"     # 输入而不清除
agent-browser keyboard type "text"     # 使用真实的击键输入 (无需选择器，针对当前焦点)
agent-browser keyboard inserttext "text"  # 不通过按键事件插入文本 (无需选择器)
agent-browser press Enter         # 按下按键
agent-browser press Control+a     # 组合键
agent-browser keydown Shift       # 按住按键
agent-browser keyup Shift         # 释放按键
agent-browser hover @e1           # 悬停
agent-browser check @e1           # 选中复选框
agent-browser uncheck @e1         # 取消选中复选框
agent-browser select @e1 "value"  # 选择下拉菜单项
agent-browser scroll down 500     # 滚动页面 (使用 --selector <sel> 指定容器)
agent-browser scrollintoview @e1  # 将元素滚动到视图中 (别名: scrollinto)
agent-browser drag @e1 @e2        # 拖放
agent-browser upload @e1 file.pdf # 上传文件
```

### 获取信息 (Get information)
```bash
agent-browser get text @e1        # 获取元素文本
agent-browser get html @e1        # 获取 innerHTML
agent-browser get value @e1       # 获取输入框的值
agent-browser get attr @e1 href   # 获取属性
agent-browser get title           # 获取页面标题
agent-browser get url             # 获取当前 URL
agent-browser get count ".item"   # 统计匹配的元素数量
agent-browser get box @e1         # 获取边界框 (bounding box)
agent-browser get styles @e1      # 获取计算后的样式
```

### 检查状态 (Check state)
```bash
agent-browser is visible @e1      # 检查是否可见
agent-browser is enabled @e1      # 检查是否启用
agent-browser is checked @e1      # 检查是否选中
```

### 截图和 PDF (Screenshots & PDF)
```bash
agent-browser screenshot          # 截图 (如果没有提供路径，则保存到临时目录)
agent-browser screenshot path.png # 保存到文件
agent-browser screenshot --full   # 全屏截图
agent-browser screenshot --annotate   # 带有带编号元素标签的注释截图
agent-browser pdf output.pdf      # 保存为 PDF
```

注释截图会在交互式元素上覆盖带编号的标签 `[N]`。每个标签对应于引用 `@eN`，因此引用适用于视觉和文本工作流：
```bash
agent-browser screenshot --annotate ./page.png
# 输出: [1] @e1 button "Submit", [2] @e2 link "Home", [3] @e3 textbox "Email"
agent-browser click @e2     # 点击标记为 [2] 的 "Home" 链接
```

### 视频录制 (Video recording)
```bash
agent-browser record start ./demo.webm    # 开始录制 (使用当前 URL + 状态)
agent-browser click @e1                   # 执行操作
agent-browser record stop                 # 停止并保存视频
agent-browser record restart ./take2.webm # 停止当前录制并开始新的录制
```
录制会创建一个全新的上下文，但会保留会话中的 cookie/存储数据。

### 等待 (Wait)
```bash
agent-browser wait @e1                     # 等待元素
agent-browser wait 2000                    # 等待毫秒数
agent-browser wait --text "Success"        # 等待文本出现
agent-browser wait --url "**/dashboard"    # 等待 URL 模式
agent-browser wait --load networkidle      # 等待网络空闲
agent-browser wait --fn "window.ready"     # 等待 JS 条件满足
```

加载状态: `load`, `domcontentloaded`, `networkidle`

### 鼠标控制 (Mouse control)
```bash
agent-browser mouse move 100 200      # 移动鼠标
agent-browser mouse down left         # 按下鼠标按键 (left/right/middle)
agent-browser mouse up left           # 释放鼠标按键
agent-browser mouse wheel 100         # 滚动滚轮
```

### 语义定位器 (Semantic locators) (引用的替代方案)
```bash
agent-browser find role button click --name "Submit"
agent-browser find text "Sign In" click
agent-browser find label "Email" fill "user@test.com"
agent-browser find placeholder "Search..." fill "query"
agent-browser find alt "Logo" click
agent-browser find title "Close" click
agent-browser find testid "submit-btn" click
agent-browser find first ".item" click
agent-browser find last ".item" click
agent-browser find nth 2 "a" text
```

动作: `click`, `fill`, `type`, `hover`, `focus`, `check`, `uncheck`, `text`
选项: `--name <name>` (按可访问名称过滤 role), `--exact` (要求完全文本匹配)

### 浏览器设置 (Browser settings)
```bash
agent-browser set viewport 1920 1080      # 设置视口大小
agent-browser set device "iPhone 14"      # 模拟设备
agent-browser set geo 37.7749 -122.4194   # 设置地理位置
agent-browser set offline on              # 切换离线模式
agent-browser set headers '{"X-Key":"v"}' # 额外的 HTTP 请求头
agent-browser set credentials user pass   # HTTP 基础认证
agent-browser set media dark              # 模拟颜色方案
```

### Cookies 和存储 (Cookies & Storage)
```bash
agent-browser cookies                     # 获取所有 cookies
agent-browser cookies set name value      # 设置 cookie
agent-browser cookies clear               # 清除 cookies

agent-browser storage local               # 获取所有 localStorage
agent-browser storage local key           # 获取特定键
agent-browser storage local set k v       # 设置值
agent-browser storage local clear         # 清除所有

agent-browser storage session             # 对 sessionStorage 执行相同操作
```

### 网络 (Network)
```bash
agent-browser network route <url>              # 拦截请求
agent-browser network route <url> --abort      # 阻止请求
agent-browser network route <url> --body '{}'  # 模拟响应
agent-browser network unroute [url]            # 移除路由
agent-browser network requests                 # 查看追踪的请求
agent-browser network requests --filter api    # 过滤请求
```

### 标签页和窗口 (Tabs & Windows)
```bash
agent-browser tab                 # 列出标签页
agent-browser tab new [url]       # 新建标签页
agent-browser tab 2               # 切换到指定标签页
agent-browser tab close           # 关闭标签页
agent-browser window new          # 新建窗口
```

### 框架 (Frames)
```bash
agent-browser frame "#iframe"     # 切换到 iframe
agent-browser frame main          # 回到主框架
```

### 对话框 (Dialogs)
```bash
agent-browser dialog accept [text]  # 接受对话框 (带有可选的提示文本)
agent-browser dialog dismiss        # 拒绝/关闭对话框
```

### 差异对比 (Diff) (比较快照、截图、URL)
```bash
agent-browser diff snapshot                              # 比较当前快照与上一个快照
agent-browser diff snapshot --baseline before.txt        # 比较当前快照与已保存的快照文件
agent-browser diff snapshot --selector "#main" --compact # 范围限定的快照差异
agent-browser diff screenshot --baseline before.png      # 与基线的视觉像素差异
agent-browser diff screenshot --baseline b.png -o d.png  # 将差异图像保存到自定义路径
agent-browser diff screenshot --baseline b.png -t 0.2    # 调整颜色阈值 (0-1)
agent-browser diff url https://v1.com https://v2.com     # 比较两个 URL (快照差异)
agent-browser diff url https://v1.com https://v2.com --screenshot  # 同时进行视觉差异比较
agent-browser diff url https://v1.com https://v2.com --selector "#main"  # 范围限定到元素
```

### JavaScript
```bash
agent-browser eval "document.title"   # 运行 JavaScript
agent-browser eval -b "base64code"    # 运行 base64 编码的 JS
agent-browser eval --stdin            # 从标准输入读取 JS 并运行
```

### 调试和分析 (Debug & Profiling)
```bash
agent-browser console                 # 查看控制台消息
agent-browser console --clear         # 清除控制台
agent-browser errors                  # 查看页面错误
agent-browser errors --clear          # 清除错误
agent-browser highlight @e1           # 高亮元素
agent-browser trace start             # 开始记录跟踪信息 (trace)
agent-browser trace stop trace.zip    # 停止并保存跟踪信息
agent-browser profiler start          # 启动 Chrome DevTools 分析器
agent-browser profiler stop profile.json  # 停止并保存分析文件
```

### 状态管理 (State management)
```bash
agent-browser state save auth.json    # 保存认证状态
agent-browser state load auth.json    # 加载认证状态
agent-browser state list              # 列出保存的状态文件
agent-browser state show <file>       # 显示状态摘要
agent-browser state rename <old> <new>  # 重命名状态文件
agent-browser state clear [name]      # 清除会话的状态
agent-browser state clear --all       # 清除所有已保存的状态
agent-browser state clean --older-than <days>  # 删除旧状态
```

### 设置 (Setup)
```bash
agent-browser install                 # 下载 Chromium 浏览器
agent-browser install --with-deps     # 同时安装系统依赖 (Linux)
```

## 全局选项 (Global Options)

| 选项 | 描述 |
|--------|-------------|
| `--session <name>` | 隔离的浏览器会话 (`AGENT_BROWSER_SESSION` 环境变量) |
| `--session-name <name>` | 自动保存/恢复会话状态 (`AGENT_BROWSER_SESSION_NAME` 环境变量) |
| `--profile <path>` | 持久化浏览器配置文件 (`AGENT_BROWSER_PROFILE` 环境变量) |
| `--state <path>` | 从 JSON 文件加载存储状态 (`AGENT_BROWSER_STATE` 环境变量) |
| `--headers <json>` | 作用于 URL 源的 HTTP 请求头 |
| `--executable-path <path>` | 自定义浏览器二进制文件 (`AGENT_BROWSER_EXECUTABLE_PATH` 环境变量) |
| `--extension <path>` | 加载浏览器扩展 (可重复使用; `AGENT_BROWSER_EXTENSIONS` 环境变量) |
| `--args <args>` | 浏览器启动参数 (`AGENT_BROWSER_ARGS` 环境变量) |
| `--user-agent <ua>` | 自定义 User-Agent (`AGENT_BROWSER_USER_AGENT` 环境变量) |
| `--proxy <url>` | 代理服务器 (`AGENT_BROWSER_PROXY` 环境变量) |
| `--proxy-bypass <hosts>` | 绕过代理的主机 (`AGENT_BROWSER_PROXY_BYPASS` 环境变量) |
| `--ignore-https-errors` | 忽略 HTTPS 证书错误 |
| `--allow-file-access` | 允许 file:// URL 访问本地文件 |
| `-p, --provider <name>` | 云端浏览器提供商 (`AGENT_BROWSER_PROVIDER` 环境变量) |
| `--device <name>` | iOS 设备名称 (`AGENT_BROWSER_IOS_DEVICE` 环境变量) |
| `--json` | 机器可读的 JSON 输出 |
| `--full, -f` | 全屏截图 |
| `--annotate` | 带编号标签的注释截图 (`AGENT_BROWSER_ANNOTATE` 环境变量) |
| `--headed` | 显示浏览器窗口 (`AGENT_BROWSER_HEADED` 环境变量) |
| `--cdp <port\|wss://url>` | 通过 Chrome DevTools Protocol 连接 |
| `--auto-connect` | 自动发现正在运行的 Chrome (`AGENT_BROWSER_AUTO_CONNECT` 环境变量) |
| `--color-scheme <scheme>` | 颜色方案: dark, light, no-preference (`AGENT_BROWSER_COLOR_SCHEME` 环境变量) |
| `--download-path <path>` | 默认下载目录 (`AGENT_BROWSER_DOWNLOAD_PATH` 环境变量) |
| `--native` | [实验性] 使用原生 Rust 守护进程 (`AGENT_BROWSER_NATIVE` 环境变量) |
| `--config <path>` | 自定义配置文件 (`AGENT_BROWSER_CONFIG` 环境变量) |
| `--debug` | 调试输出 |

### 安全选项 (Security options)
| 选项 | 描述 |
|--------|-------------|
| `--content-boundaries` | 在边界标记中包装页面输出 (`AGENT_BROWSER_CONTENT_BOUNDARIES` 环境变量) |
| `--max-output <chars>` | 将页面输出截断为 N 个字符 (`AGENT_BROWSER_MAX_OUTPUT` 环境变量) |
| `--allowed-domains <list>` | 逗号分隔的允许域模式 (`AGENT_BROWSER_ALLOWED_DOMAINS` 环境变量) |
| `--action-policy <path>` | 动作策略 JSON 文件的路径 (`AGENT_BROWSER_ACTION_POLICY` 环境变量) |
| `--confirm-actions <list>` | 需要确认的动作类别 (`AGENT_BROWSER_CONFIRM_ACTIONS` 环境变量) |

## 配置文件

创建 `agent-browser.json` 用于持久化的默认设置（无需重复输入标志）：

**位置（优先级从低到高）：**
1. `~/.agent-browser/config.json` — 用户级默认设置
2. `./agent-browser.json` — 项目级覆盖设置
3. `AGENT_BROWSER_*` 环境变量
4. CLI 标志覆盖所有内容

```json
{
  "headed": true,
  "proxy": "http://localhost:8080",
  "profile": "./browser-data",
  "native": true
}
```

## 示例：表单提交

```bash
agent-browser open https://example.com/form
agent-browser snapshot -i
# 输出显示: textbox "Email" [ref=e1], textbox "Password" [ref=e2], button "Submit" [ref=e3]

agent-browser fill @e1 "user@example.com"
agent-browser fill @e2 "password123"
agent-browser click @e3
agent-browser wait --load networkidle
agent-browser snapshot -i  # 检查结果
```

## 示例：使用保存的状态进行身份验证

```bash
# 登录一次
agent-browser open https://app.example.com/login
agent-browser snapshot -i
agent-browser fill @e1 "username"
agent-browser fill @e2 "password"
agent-browser click @e3
agent-browser wait --url "**/dashboard"
agent-browser state save auth.json

# 后续会话: 加载已保存的状态
agent-browser state load auth.json
agent-browser open https://app.example.com/dashboard
```

### 基于请求头的身份验证 (跳过登录流程)
```bash
# 仅作用于 api.example.com 的请求头
agent-browser open api.example.com --headers '{"Authorization": "Bearer <token>"}'
# 导航到另一个域 - 请求头**不会**被发送 (安全)
agent-browser open other-site.com
# 全局请求头 (所有域)
agent-browser set headers '{"X-Custom-Header": "value"}'
```

### 身份验证保管库 (Authentication Vault)
```bash
# 在本地存储凭据 (加密)。大语言模型永远不会看到密码。
echo "pass" | agent-browser auth save github --url https://github.com/login --username user --password-stdin
agent-browser auth login github
```

## 会话和持久化配置文件 (Sessions & Persistent Profiles)

### 会话 (并行浏览器)
```bash
agent-browser --session test1 open site-a.com
agent-browser --session test2 open site-b.com
agent-browser session list
```

### 会话持久化 (自动保存/恢复)
```bash
agent-browser --session-name twitter open twitter.com
# 登录一次，状态会在重启后自动持久化
# 状态文件存储在 ~/.agent-browser/sessions/ 中
```

### 持久化配置文件
在浏览器重启之间持久化 cookies、localStorage、IndexedDB、Service Workers、缓存、登录会话。
```bash
agent-browser --profile ~/.myapp-profile open myapp.com
# 或者通过环境变量
AGENT_BROWSER_PROFILE=~/.myapp-profile agent-browser open myapp.com
```

## JSON 输出 (用于解析)

添加 `--json` 标志以获取机器可读的输出：
```bash
agent-browser snapshot -i --json
agent-browser get text @e1 --json
```

## 本地文件

```bash
agent-browser --allow-file-access open file:///path/to/document.pdf
agent-browser --allow-file-access open file:///path/to/page.html
```

## CDP 模式

```bash
agent-browser connect 9222                                          # 本地 CDP 端口
agent-browser --cdp 9222 snapshot                                   # 在每个命令上直接使用 CDP
agent-browser --cdp "wss://browser-service.com/cdp?token=..." snapshot  # 通过 WebSocket 远程连接
agent-browser --auto-connect snapshot                               # 自动发现正在运行的 Chrome
```

## 云服务提供商

```bash
# Browserbase
BROWSERBASE_API_KEY="key" BROWSERBASE_PROJECT_ID="id" agent-browser -p browserbase open example.com

# Browser Use
BROWSER_USE_API_KEY="key" agent-browser -p browseruse open example.com

# Kernel
KERNEL_API_KEY="key" agent-browser -p kernel open example.com
```

## iOS 模拟器

```bash
agent-browser device list                                        # 列出可用的模拟器
agent-browser -p ios --device "iPhone 16 Pro" open example.com   # 启动 Safari
agent-browser -p ios snapshot -i                                 # 与桌面端命令相同
agent-browser -p ios tap @e1                                     # 轻触
agent-browser -p ios swipe up                                    # 移动端特有操作: 向上滑动
agent-browser -p ios close                                       # 关闭会话
```

## 原生模式 (实验性)

纯 Rust 守护进程，使用直接的 CDP 连接 — 不需要 Node.js/Playwright：
```bash
agent-browser --native open example.com
# 或者: export AGENT_BROWSER_NATIVE=1
# 或者在 agent-browser.json 中: {"native": true}
```

---
安装: `bun add -g agent-browser && agent-browser install`。运行 `agent-browser --help` 查看所有命令。代码仓库: https://github.com/vercel-labs/agent-browser
