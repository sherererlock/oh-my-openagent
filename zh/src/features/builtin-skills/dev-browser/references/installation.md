# Dev Browser 安装指南

本指南涵盖了所有平台的安装步骤：macOS、Linux 和 Windows。

## 前置条件

- [Node.js](https://nodejs.org) v18 或更高版本（附带 npm）
- Git (用于克隆该技能)

## 安装步骤

### 步骤 1: 克隆技能代码

```bash
# 将 dev-browser 克隆到临时位置
git clone https://github.com/sawyerhood/dev-browser /tmp/dev-browser-skill

# 复制到技能目录 (根据需要调整路径)
# 对于 oh-my-opencode: 已经内置捆绑
# 对于手动安装:
mkdir -p ~/.config/opencode/skills
cp -r /tmp/dev-browser-skill/skills/dev-browser ~/.config/opencode/skills/dev-browser

# 清理临时文件
rm -rf /tmp/dev-browser-skill
```

**Windows (PowerShell):**
```powershell
# 将 dev-browser 克隆到临时位置
git clone https://github.com/sawyerhood/dev-browser $env:TEMP\dev-browser-skill

# 复制到技能目录
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.config\opencode\skills"
Copy-Item -Recurse "$env:TEMP\dev-browser-skill\skills\dev-browser" "$env:USERPROFILE\.config\opencode\skills\dev-browser"

# 清理临时文件
Remove-Item -Recurse -Force "$env:TEMP\dev-browser-skill"
```

### 步骤 2: 安装依赖

```bash
cd ~/.config/opencode/skills/dev-browser
npm install
```

**Windows (PowerShell):**
```powershell
cd "$env:USERPROFILE\.config\opencode\skills\dev-browser"
npm install
```

### 步骤 3: 启动服务器

#### 独立模式 (启动新的浏览器实例)

**macOS/Linux:**
```bash
cd ~/.config/opencode/skills/dev-browser
./server.sh &
# 或者以无头模式启动:
./server.sh --headless &
```

**Windows (PowerShell):**
```powershell
cd "$env:USERPROFILE\.config\opencode\skills\dev-browser"
Start-Process -NoNewWindow -FilePath "node" -ArgumentList "server.js"
# 或者以无头模式启动:
Start-Process -NoNewWindow -FilePath "node" -ArgumentList "server.js", "--headless"
```

**Windows (CMD):**
```cmd
cd %USERPROFILE%\.config\opencode\skills\dev-browser
start /B node server.js
```

等待出现 `Ready` 消息后再运行脚本。

#### 扩展模式 (使用现有的 Chrome 浏览器)

**macOS/Linux:**
```bash
cd ~/.config/opencode/skills/dev-browser
npm run start-extension &
```

**Windows (PowerShell):**
```powershell
cd "$env:USERPROFILE\.config\opencode\skills\dev-browser"
Start-Process -NoNewWindow -FilePath "npm" -ArgumentList "run", "start-extension"
```

等待出现 `Extension connected` 消息。

## Chrome 扩展程序设置 (可选)

Chrome 扩展程序允许控制现有的带有所有已登录会话的 Chrome 浏览器。

### 安装

1. 从[最新发布版本](https://github.com/sawyerhood/dev-browser/releases/latest)下载 `extension.zip`
2. 解压到永久位置:
   - **macOS/Linux:** `~/.dev-browser-extension`
   - **Windows:** `%USERPROFILE%\.dev-browser-extension`
3. 打开 Chrome → 访问 `chrome://extensions`
4. 开启右上角的 "开发者模式 (Developer mode)"
5. 点击 "加载已解压的扩展程序 (Load unpacked)" → 选择刚刚解压的文件夹

### 使用方法

1. 点击 Chrome 工具栏中的 Dev Browser 扩展程序图标
2. 切换到 "活跃 (Active)" 状态
3. 启动扩展程序中继服务器 (参见上文)
4. 使用 dev-browser 脚本 - 它们将会控制你现有的 Chrome 浏览器

## 常见问题排查

### 服务器无法启动

**检查 Node.js 版本:**
```bash
node --version  # 应该为 v18+
```

**检查端口是否可用:**
```bash
# macOS/Linux
lsof -i :3000

# Windows
netstat -ano | findstr :3000
```

### Playwright 安装问题

如果 Chromium 安装失败:
```bash
npx playwright install chromium
```

### Windows 特有问题

**执行策略:**
如果 PowerShell 脚本被拦截:
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

**路径问题:**
在路径中使用正斜杠或转义的反斜杠:
```powershell
# 推荐
cd "$env:USERPROFILE/.config/opencode/skills/dev-browser"
# 同样可行
cd "$env:USERPROFILE\.config\opencode\skills\dev-browser"
```

### 扩展程序未连接

1. 确保扩展程序处于 "Active" 状态 (点击图标进行切换)
2. 检查中继服务器是否正在运行 (`npm run start-extension`)
3. 在控制台查找是否有 `Extension connected` 消息
4. 尝试在 `chrome://extensions` 中重新加载扩展程序

## 权限设置

要跳过 Claude Code 中的权限提示，请将以下内容添加到 `~/.claude/settings.json` 中:

```json
{
  "permissions": {
    "allow": ["Skill(dev-browser:dev-browser)", "Bash(npx tsx:*)"]
  }
}
```

## 更新

```bash
cd ~/.config/opencode/skills/dev-browser
git pull
npm install
```

**Windows:**
```powershell
cd "$env:USERPROFILE\.config\opencode\skills\dev-browser"
git pull
npm install
```
