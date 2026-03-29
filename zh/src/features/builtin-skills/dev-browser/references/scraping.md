# 数据抓取指南

对于大型数据集（关注者、帖子、搜索结果），**拦截并重放网络请求**，而不是滚动和解析 DOM。这更快、更可靠，并且自动处理分页。

## 为什么不使用滚动？

滚动缓慢、不可靠且浪费时间。API 会返回结构化数据并内置分页功能。始终优先使用 API 重放。

## 从小规模开始，然后扩展

**不要试图一次性自动化所有内容。** 增量式工作：

1. **捕获一个请求** - 验证你是否拦截了正确的端点
2. **检查一个响应** - 在编写提取代码之前了解数据模式 (schema)
3. **提取几个项目** - 确保你的解析逻辑有效
4. **然后扩展** - 仅在基础功能正常工作后才添加分页循环

这可以防止在问题只是一个简单路径（如 `data.user.timeline` 与 `data.user.result.timeline`）时，浪费时间调试复杂的脚本。

## 逐步工作流程

### 1. 捕获请求详细信息

首先，拦截一个请求以了解 URL 结构和必需的请求头：

```typescript
import { connect, waitForPageLoad } from "@/client.js";
import * as fs from "node:fs";

const client = await connect();
const page = await client.page("site");

let capturedRequest = null;
page.on("request", (request) => {
  const url = request.url();
  // 寻找 API 端点（根据目标网站调整模式）
  if (url.includes("/api/") || url.includes("/graphql/")) {
    capturedRequest = {
      url: url,
      headers: request.headers(),
      method: request.method(),
    };
    fs.writeFileSync("tmp/request-details.json", JSON.stringify(capturedRequest, null, 2));
    console.log("捕获到请求:", url.substring(0, 80) + "...");
  }
});

await page.goto("https://example.com/profile");
await waitForPageLoad(page);
await page.waitForTimeout(3000);

await client.disconnect();
```

### 2. 捕获响应以了解模式

保存原始响应以检查数据结构：

```typescript
page.on("response", async (response) => {
  const url = response.url();
  if (url.includes("UserTweets") || url.includes("/api/data")) {
    const json = await response.json();
    fs.writeFileSync("tmp/api-response.json", JSON.stringify(json, null, 2));
    console.log("捕获到响应");
  }
});
```

然后分析结构以找到：

- 数据数组存在的位置（例如 `data.user.result.timeline.instructions[].entries`）
- 分页游标 (cursor) 存在的位置（例如 `cursor-bottom` 条目）
- 你需要提取哪些字段

### 3. 使用分页重放 API

一旦你了解了模式，就可以直接重放请求：

```typescript
import { connect } from "@/client.js";
import * as fs from "node:fs";

const client = await connect();
const page = await client.page("site");

const results = new Map(); // 使用 Map 去重
const headers = JSON.parse(fs.readFileSync("tmp/request-details.json", "utf8")).headers;
const baseUrl = "https://example.com/api/data";

let cursor = null;
let hasMore = true;

while (hasMore) {
  // 构建带有分页游标的 URL
  const params = { count: 20 };
  if (cursor) params.cursor = cursor;
  const url = `${baseUrl}?params=${encodeURIComponent(JSON.stringify(params))}`;

  // 在浏览器上下文中执行 fetch（具有身份验证 Cookie/Headers）
  const response = await page.evaluate(
    async ({ url, headers }) => {
      const res = await fetch(url, { headers });
      return res.json();
    },
    { url, headers }
  );

  // 提取数据和游标（根据你的 API 调整路径）
  const entries = response?.data?.entries || [];
  for (const entry of entries) {
    if (entry.type === "cursor-bottom") {
      cursor = entry.value;
    } else if (entry.id && !results.has(entry.id)) {
      results.set(entry.id, {
        id: entry.id,
        text: entry.content,
        timestamp: entry.created_at,
      });
    }
  }

  console.log(`已获取一页，总计: ${results.size}`);

  // 检查停止条件
  if (!cursor || entries.length === 0) hasMore = false;

  // 速率限制 - 请保持尊重
  await new Promise((r) => setTimeout(r, 500));
}

// 导出结果
const data = Array.from(results.values());
fs.writeFileSync("tmp/results.json", JSON.stringify(data, null, 2));
console.log(`已保存 ${data.length} 个项目`);

await client.disconnect();
```

## 关键模式

| 模式 | 描述 |
| ----------------------- | ------------------------------------------------------ |
| `page.on('request')` | 捕获传出的请求 URL + 请求头 |
| `page.on('response')` | 捕获响应数据以了解模式 |
| `page.evaluate(fetch)` | 在浏览器上下文中重放请求（继承身份验证） |
| `Map` 用于去重 | API 通常会在页面之间返回重叠的数据 |
| 基于游标的分页 | 在响应中寻找 `cursor`、`next_token`、`offset` |

## 提示

- **扩展模式**：`page.context().cookies()` 不起作用 - 改为从拦截的请求中捕获身份验证头
- **速率限制**：在请求之间添加 500 毫秒以上的延迟以避免被屏蔽
- **停止条件**：检查空结果、缺少游标或达到日期/ID 阈值
- **GraphQL API**：URL 参数通常包含 `variables` 和 `features` JSON 对象 - 捕获并重用它们
