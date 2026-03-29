# 模型设置兼容性解析器设计

## 目标

引入一个中央解析器，该解析器接受一个已经选择的模型和一组所需的模型设置，然后返回针对该确切模型的最佳兼容配置。

这明确地独立于模型回退。

## 问题

目前，关于 `variant` 和 `reasoningEffort` 兼容性的逻辑散布在多个地方：
- `hooks/anthropic-effort`
- `plugin/chat-params`
- agent/category/fallback 配置层
- delegate/background 提示词管道

这造成了不一致的行为：
- 有些路径会限制不支持的级别
- 有些路径让它们原样通过
- 有些路径会默默丢弃它们
- 有些路径使用了不能普遍适用的、特定于模型系列的假设

结果是，即使选择的模型本身是有效的，请求行为也很脆弱。

## 范围

第 1 阶段仅涵盖：
- `variant`
- `reasoningEffort`

第 1 阶段不包括的范围：
- 模型回退本身
- `thinking`
- `maxTokens`
- `temperature`
- `top_p`
- 设置的自动向上重新映射

## 预期行为

给定一个固定的模型和所需的设置：
1. 如果所需的值受支持，则保留它。
2. 如果不受支持，则降级到最接近的、受支持的较低值。
3. 如果不存在兼容的值，则丢弃该字段。
4. 不要切换模型。
5. 在第 1 阶段中，不要自动升级设置。

## 架构

添加一个中央模块：
- `src/shared/model-settings-compatibility.ts`

核心 API：

```ts
type DesiredModelSettings = {
  variant?: string
  reasoningEffort?: string
}

type ModelSettingsCompatibilityInput = {
  providerID: string
  modelID: string
  desired: DesiredModelSettings
}

type ModelSettingsCompatibilityChange = {
  field: "variant" | "reasoningEffort"
  from: string
  to?: string
  reason: string
}

type ModelSettingsCompatibilityResult = {
  variant?: string
  reasoningEffort?: string
  changes: ModelSettingsCompatibilityChange[]
}
```

## 兼容性模型

第 1 阶段应该在**平台提供可靠能力数据的地方以元数据优先**，并且只有在缺少该元数据时才回退到基于系列的规则。

### Variant 兼容性

首选的真实来源：
- OpenCode/提供商模型元数据 (`variants`)

元数据不可用时的回退：
- 基于系列的阶梯

回退阶梯示例：
- Claude Opus 系列：`low`, `medium`, `high`, `max`
- Claude Sonnet/Haiku 系列：`low`, `medium`, `high`
- OpenAI GPT 系列：仅在缺少元数据时保守地使用系列回退
- 未知系列：保守地丢弃不支持的值

### Reasoning effort 兼容性

当前第 1 阶段的真实来源：
- 保守的模型/提供商系列启发式方法

原因：
- 当前可用的 OpenCode SDK/提供商元数据公开了模型 `variants`，但没有公开关于 `reasoningEffort` 级别的等效的每模型能力列表

示例：
- GPT/OpenAI 风格模型：在系列启发式方法支持的情况下，使用 `low`, `medium`, `high`, `xhigh`
- 通过当前 OpenCode 路径的 Claude 系列：在第 1 阶段将 `reasoningEffort` 视为不支持并将其删除

解析器应仅保留纯粹的模型/设置逻辑。传输限制仍然是请求构建路径的责任。

## 关注点分离

此设计故意分离：
- 模型选择（`resolveModel...`，回退链）
- 设置兼容性（此解析器）
- 请求传输兼容性（`chat.params`，提示词主体约束）

这保持了清晰的责任：
- 首先选择模型
- 其次规范化设置
- 第三构建请求

## 第一个集成点

第 1 阶段应首先集成到 `chat.params` 中。

原因：
- 它已经是请求时调整的集中路径
- 它可以影响面向提供商的选项，而不会将不支持的字段泄漏到提示词有效负载主体中
- 它避免了试图一次修补每个提示词构造函数

## 推出计划

### 第 1 阶段
- 添加解析器模块和测试
- 集成到 `chat.params`
- 迁移 `anthropic-effort` 以使用解析器，或成为围绕它的一层特定于 Claude 的薄薄补充

### 第 2 阶段
- 扩展到 `thinking`, `maxTokens`, `temperature`, `top_p`
- 如果需要，正式确定请求路径能力表

### 第 3 阶段
- 将所有 variant/reasoning 规范化集中起来，远离分散的钩子和临时的调用者

## 风险

- 使系列规则过度拟合当前的模型命名约定
- 意外更改目前依赖隐式行为的路径上的请求语义
- 将提供商传输限制与模型能力逻辑混合

## 缓解措施

- 在第 1 阶段保持解析器纯粹且范围狭窄
- 为保留/降级/丢弃决策添加显式的回归测试
- 首先在一个中心点集成 (`chat.params`)
- 在所需值已有效的情况下保留现有行为

## 建议

将中央解析器作为专用分支/工作树中新的、隔离的实现继续进行。
这是一条清晰的长期路径，并且比继续在钩子中添加特殊情况限制更具可审查性。
