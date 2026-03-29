---
name: dev-browser
description: 具有持久化页面状态的浏览器自动化。当用户要求导航网站、填写表单、截图、提取网络数据、测试 Web 应用或自动化浏览器工作流时使用。触发短语包括“转到 [url]”、“点击”、“填写表单”、“截图”、“抓取”、“自动化”、“测试网站”、“登录”或任何浏览器交互请求。
---

# 开发者浏览器技能 (Dev Browser Skill)

在脚本执行之间保持页面状态的浏览器自动化。编写专注的小型脚本来逐步完成任务。一旦你验证了工作流的一部分，并且有重复的工作要做，你就可以编写一个脚本来在一次执行中完成这些重复工作。

## 选择你的方法

- **本地/提供源码的网站**：首先阅读源代码，直接编写选择器
- **未知的页面布局**：使用 `getAISnapshot()` 发现元素，使用 `selectSnapshotRef()` 与其交互
- **视觉反馈**：通过截图查看用户看到的内容

## 设置

> **安装**：有关包含 Windows 支持的详细设置说明，请参阅 [references/installation.md](references/installation.md)。

提供两种模式。如果不清楚使用哪一种，请询问用户。

### 独立模式（默认）

启动一个新的 Chromium 浏览器以进行全新的自动化会话。

```bash
./skills/dev-browser/server.sh &
```

如果用户请求，添加 `--headless` 标志。**在运行脚本之前等待 `Ready` 消息。**

### 扩展模式

连接到用户现有的 Chrome 浏览器。在以下情况使用此模式：

- 用户已经登录网站，并希望你在非本地开发的已认证体验后执行操作。
- 用户要求你使用扩展

**重要提示**：核心流程仍然相同。你在他们的浏览器内创建命名页面。

**启动中继服务器：**

```bash
cd skills/dev-browser && npm i && npm run start-extension &
```

等待控制台显示 `Waiting for extension to connect...` 接着是 `Extension connected`。这表示客户端已连接，浏览器已准备好受控。
**工作流程：**

1. 脚本调用 `client.page("name")`，就像正常模式一样创建新页面 / 连接到现有页面。
2. 自动化在用户的实际浏览器会话上运行

如果扩展尚未连接，请告诉用户启动并激活它。下载链接：https://github.com/SawyerHood/dev-browser/releases

## 编写脚本

> **从 `skills/dev-browser/` 目录运行所有脚本。** `@/` 导入别名需要此目录的配置。

使用 Heredoc 内联执行脚本：

```bash
cd skills/dev-browser && npx tsx <<'EOF'
import { connect, waitForPageLoad } from "@/client.js";

const client = await connect();
// 创建具有自定义视口大小的页面（可选）
const page = await client.page("example", { viewport: { width: 1920, height: 1080 } });

await page.goto("https://example.com");
await waitForPageLoad(page);

console.log({ title: await page.title(), url: page.url() });
await client.disconnect();
EOF
```

**仅当**脚本需要重用、很复杂或用户明确要求时，才写入 `tmp/` 文件。

### 关键原则

1. **小型脚本**：每个脚本只做一件事（导航、点击、填写、检查）
2. **评估状态**：在最后记录/返回状态以决定后续步骤
3. **描述性的页面名称**：使用 `"checkout"`、`"login"`，而不是 `"main"`
4. **断开连接以退出**：`await client.disconnect()` - 页面在服务器上保持持久化
5. **在 evaluate 中使用纯 JS**：`page.evaluate()` 在浏览器中运行 - 没有 TypeScript 语法

## 工作流循环

对于复杂任务，请遵循此模式：

1. **编写一个脚本**来执行一项操作
2. **运行它**并观察输出
3. **评估** - 成功了吗？当前状态是什么？
4. **决定** - 任务完成了吗，还是我们需要另一个脚本？
5. **重复**直到任务完成

### 浏览器上下文中没有 TypeScript

传递给 `page.evaluate()` 的代码在浏览器中运行，浏览器不理解 TypeScript：

```typescript
// ✅ 正确：纯 JavaScript
const text = await page.evaluate(() => {
  return document.body.innerText;
});

// ❌ 错误：TypeScript 语法在运行时将失败
const text = await page.evaluate(() => {
  const el: HTMLElement = document.body; // 类型注解在浏览器中会中断！
  return el.innerText;
});
```

## 抓取数据

对于抓取大型数据集，拦截并重放网络请求，而不是滚动 DOM。有关涵盖请求捕获、模式发现和分页 API 重放的完整指南，请参阅 [references/scraping.md](references/scraping.md)。

## 客户端 API

```typescript
const client = await connect();

// 获取或创建命名页面（视口仅适用于新页面）
const page = await client.page("name");
const pageWithSize = await client.page("name", { viewport: { width: 1920, height: 1080 } });

const pages = await client.list(); // 列出所有页面名称
await client.close("name"); // 关闭页面
await client.disconnect(); // 断开连接（页面持久存在）

// ARIA Snapshot 方法
const snapshot = await client.getAISnapshot("name"); // 获取可访问性树
const element = await client.selectSnapshotRef("name", "e5"); // 通过 ref 获取元素
```

`page` 对象是一个标准的 Playwright Page。

## 等待

```typescript
import { waitForPageLoad } from "@/client.js";

await waitForPageLoad(page); // 导航后
await page.waitForSelector(".results"); // 等待特定元素
await page.waitForURL("**/success"); // 等待特定 URL
```

## 检查页面状态

### 截图

```typescript
await page.screenshot({ path: "tmp/screenshot.png" });
await page.screenshot({ path: "tmp/full.png", fullPage: true });
```

### ARIA Snapshot (元素发现)

使用 `getAISnapshot()` 发现页面元素。返回 YAML 格式的可访问性树：

```yaml
- banner:
  - link "Hacker News" [ref=e1]
  - navigation:
    - link "new" [ref=e2]
- main:
  - list:
    - listitem:
      - link "Article Title" [ref=e8]
      - link "328 comments" [ref=e9]
- contentinfo:
  - textbox [ref=e10]
    - /placeholder: "Search"
```

**解释 refs:**

- `[ref=eN]` - 用于交互的元素引用（仅限可见的、可点击的元素）
- `[checked]`、`[disabled]`、`[expanded]` - 元素状态
- `[level=N]` - 标题级别
- `/url:`、`/placeholder:` - 元素属性

**与 refs 交互:**

```typescript
const snapshot = await client.getAISnapshot("hackernews");
console.log(snapshot); // 找到你需要的 ref

const element = await client.selectSnapshotRef("hackernews", "e2");
await element.click();
```

## 错误恢复

失败后页面状态仍会持久存在。使用以下方式调试：

```bash
cd skills/dev-browser && npx tsx <<'EOF'
import { connect } from "@/client.js";

const client = await connect();
const page = await client.page("hackernews");

await page.screenshot({ path: "tmp/debug.png" });
console.log({
  url: page.url(),
  title: await page.title(),
  bodyText: await page.textContent("body").then((t) => t?.slice(0, 200)),
});

await client.disconnect();
EOF
```
