# 模型设置兼容性解析器实施计划

> **对于代理工作者：** 必需：使用 superpowers:subagent-driven-development（如果子代理可用）或 superpowers:executing-plans 来实施此计划。步骤使用复选框 (`- [ ]`) 语法进行跟踪。

**目标：** 集中处理 `variant` 和 `reasoningEffort` 的兼容性，以便已选择的模型接收到针对该确切模型的最佳有效设置。

**架构：** 在 `src/shared/` 中引入一个纯粹的共享解析器，它计算兼容的设置并记录降级/删除。首先将其集成在 `chat.params` 中，然后将特定于 Claude 的努力量逻辑保留为薄薄的一层，而不是一个特殊情况的策略所有者。

**技术栈：** TypeScript，Bun test，现有的共享模型规范化/实用工具，OpenCode 插件 `chat.params` 路径。

---

### 任务 1：创建纯粹的兼容性解析器

**文件：**
- 创建: `src/shared/model-settings-compatibility.ts`
- 创建: `src/shared/model-settings-compatibility.test.ts`
- 修改: `src/shared/index.ts`

- [ ] **步骤 1：为精确保留行为编写失败的测试**
- [ ] **步骤 2：为降级行为编写失败的测试（在需要时将 `max` -> `high`, `xhigh` -> `high`）**
- [ ] **步骤 3：为不支持的值的删除编写失败的测试**
- [ ] **步骤 4：为模型系列的区分编写失败的测试（Opus vs Sonnet/Haiku，GPT 系列变体）**
- [ ] **步骤 5：使用显式的能力阶梯实现纯粹的解析器**
- [ ] **步骤 6：从 `src/shared/index.ts` 导出解析器**
- [ ] **步骤 7：运行 `bun test src/shared/model-settings-compatibility.test.ts`**
- [ ] **步骤 8：提交 (Commit)**

### 任务 2：将解析器集成到 chat.params 中

**文件：**
- 修改: `src/plugin/chat-params.ts`
- 修改: `src/plugin/chat-params.test.ts`

- [ ] **步骤 1：编写失败的测试，表明 `chat.params` 将解析器的输出应用于运行时设置**
- [ ] **步骤 2：确保测试涵盖 `variant` 和 `reasoningEffort` 的决策**
- [ ] **步骤 3：更新 `chat-params.ts`，使其在特定于钩子的调整之前调用共享解析器**
- [ ] **步骤 4：保留现有的提示词参数存储合并行为**
- [ ] **步骤 5：运行 `bun test src/plugin/chat-params.test.ts`**
- [ ] **步骤 6：提交 (Commit)**

### 任务 3：围绕解析器重新界定 anthropic-effort 的范围

**文件：**
- 修改: `src/hooks/anthropic-effort/hook.ts`
- 修改: `src/hooks/anthropic-effort/index.test.ts`

- [ ] **步骤 1：编写失败的测试，编纂集中化后预期保留的特定于 Anthropic 的行为**
- [ ] **步骤 2：在仍然需要的地方，将 `anthropic-effort` 缩减为特定于 Claude/Anthropic 的努力量注入**
- [ ] **步骤 3：如果现在由共享解析器拥有它，则从钩子中删除重复的兼容性策略**
- [ ] **步骤 4：运行 `bun test src/hooks/anthropic-effort/index.test.ts`**
- [ ] **步骤 5：提交 (Commit)**

### 任务 4：在实际请求路径中添加集成/回归覆盖率

**文件：**
- 修改: `src/plugin/chat-params.test.ts`
- 修改: `src/hooks/anthropic-effort/index.test.ts`
- 仅在附近套件中需要的地方添加测试

- [ ] **步骤 1：为带有 `variant=max` 的非 Opus Claude 添加回归测试，解析为兼容设置而无需临时的仅路径逻辑**
- [ ] **步骤 2：为 GPT 风格的 `reasoningEffort` 兼容性添加回归测试**
- [ ] **步骤 3：添加回归测试，表明支持的值保持不变**
- [ ] **步骤 4：运行重点测试集**
- [ ] **步骤 5：提交 (Commit)**

### 任务 5：验证完整的质量标准

**文件：**
- 没有预期的代码更改

- [ ] **步骤 1：运行 `bun run typecheck`**
- [ ] **步骤 2：为涉及的文件运行重点套件**
- [ ] **步骤 3：如果干净，运行 `bun test`**
- [ ] **步骤 4：审查差异以防意外的范围蔓延**
- [ ] **步骤 5：提交任何最终的清理**

### 任务 6：准备 PR 元数据

**文件：**
- 除非进一步更新文档，否则不需要更改仓库文件

- [ ] **步骤 1：编写人类摘要，解释这是设置兼容性，而不是模型回退**
- [ ] **步骤 2：记录范围：第 1 阶段仅涵盖 `variant` 和 `reasoningEffort`**
- [ ] **步骤 3：记录明确的非目标：在第 1 阶段中不进行模型切换，不进行自动放大**
- [ ] **步骤 4：请求审查**
