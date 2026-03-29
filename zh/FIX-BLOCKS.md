# 发布前阻塞问题：发布前必须修复所有问题

两份独立的发布前审查报告（Opus 4.6 + GPT-5.4）均得出结论：**阻塞——请勿发布**。您必须使用 UltraBrain 并行智能体修复以下所有阻塞问题。采用 TDD（测试驱动开发）风格进行工作：先编写/更新测试，然后修复，最后验证测试是否通过。

## 策略

使用 ultrawork (ulw) 并行生成 UltraBrain 智能体。每个 UB 智能体获得一个不重叠的作用域。在所有智能体完成后，运行 bun test 验证一切是否通过。每个修复组进行原子提交。

---

## 关键阻塞项 (必须修复 -- 6项)

### C1: Hashline 向后兼容性
**问题:** hashline 中严格的空白哈希处理会改变缩进行 LINE#ID 的值。这会破坏缓存/持久化编辑操作中的现有锚点。
**修复:** 添加一个兼容层——当按新哈希查找失败时，回退到旧哈希（没有严格的空白处理）。或者对哈希格式进行版本控制。
**文件:** 在 src/tools/ 或 src/shared/ 中查找与 hashline 相关的文件

### C2: 仅 OpenAI 模型目录在 OpenCode-Go 下损坏
**问题:** isOpenAiOnlyAvailability() 没有排除 availability.opencodeGo。当存在 OpenCode-Go 时，仅 OpenAI 检测是错误的——模型会被错误路由。
**修复:** 在 isOpenAiOnlyAvailability() 中添加 !availability.opencodeGo 检查。
**文件:** 模型/提供商系统文件——搜索 isOpenAiOnlyAvailability

### C3: CLI/运行时模型表分歧
**问题:** CLI 安装时和运行时的模型表不一致：
- ultrabrain: CLI 中是 gpt-5.3-codex，而运行时是 gpt-5.4
- atlas: CLI 中是 claude-sonnet-4-5，而运行时是 claude-sonnet-4-6
- unspecified-high 也有分歧
**修复:** 协调所有模型表。为每个选择正确的模型，并使 CLI 和运行时匹配。
**文件:** 搜索模型表定义、智能体配置、CLI 模型引用

### C4: atlas/metis/sisyphus-junior 缺少 OpenAI 回退
**问题:** 在仅 OpenAI 环境中，这些智能体可能会解析为 opencode/glm-4.7-free 或 undefined。不存在有效的 OpenAI 回退路径。
**修复:** 为所有需要它们的智能体添加有效的 OpenAI 模型回退路径。
**文件:** 智能体配置/模型解析代码

### C5: model_fallback 默认值不匹配
**问题:** 模式和文档指出 model_fallback 默认为 false，但运行时将未设置视为 true。对所有用户来说，行为在未告知的情况下发生了改变。
**修复:** 对齐——要么更新模式/文档说明为 true，要么修复运行时默认为 false。从 git 历史中检查预期行为是什么。
**文件:** 模式定义，运行时配置加载

### C6: background_output 默认值更改
**问题:** background_output 现在默认为 full_session=true。旧的调用者在不修改代码的情况下获得了不同的输出格式。
**修复:** 要么清楚地记录此更改，要么恢复旧的默认值并使 full_session 成为可选。
**文件:** 后台输出处理代码

---

## 高优先级 (强烈建议 -- 4项)

### H1: 运行时回退 session-status-handler 竞争
**问题:** 当回退模型处于 pending（挂起）状态时，处理程序无法在后续的 cooldown 事件上推进链。
**修复:** 允许像 message-update-handler 那样的覆盖。
**文件:** 搜索 session-status-handler, message-update-handler

### H2: Atlas 最终阶段审批门控逻辑
**问题:** 审批门控逻辑与真实的 Prometheus 计划结构（嵌套的复选框，并行执行）不匹配。触发逻辑是错误的。
**修复:** 更新以处理真实的计划结构。
**文件:** Atlas 智能体代码，审批门控逻辑

### H3: delegate-task-english-directive 废弃代码
**问题:** 未从 tool-execute-before.ts 调度 + 钩子签名错误。要么正确连接，要么完全删除。
**修复:** 如果不需要则删除（更干净）。如果需要，修复调度 + 签名。
**文件:** src/hooks/, tool-execute-before.ts

### H4: 自动斜杠命令会话生命周期去重
**问题:** 去重使用会话生命周期，这抑制了合法的重复相同命令。
**修复:** 更改为较短的 TTL（例如，30秒），而不是会话生命周期。
**文件:** 斜杠命令处理代码

---

## 附加阻塞项（来自 GPT-5.4 审查）

### G1: 包身份标识分裂
**问题:** 安装程序写入 oh-my-openagent，但 doctor、自动更新、版本查找、发布工作流仍然引用 oh-my-opencode。处于半迁移状态。
**修复:** 审核对包名称的所有引用。要么在本次发布中一致地完成迁移，要么恢复为单个名称。
**文件:** Installer, doctor, auto-update, version lookup, publish workflow -- 使用 grep 搜索这两个包名

### G2: OpenCode-Go --opencode-go 值验证
**问题:** 没有对 --opencode-go CLI 值进行验证。没有检测现有的 OpenCode-Go 安装。
**修复:** 添加值验证 + 现有安装检测。
**文件:** CLI 选项处理代码

### G3: 技能/钩子引用错误
**问题:**
- work-with-pr 引用了不存在的 git 工具类别
- github-triage 引用了 TaskCreate/TaskUpdate，这些不是真正的工具名称
**修复:** 修复工具引用，使用实际的工具名称。
**文件:** .opencode/skills/ 中的技能定义文件

### G4: 上下文限制缓存过时
**问题:** 共享上下文限制解析器会缓存提供商配置。当配置更改时，过时的已删除限制会持续存在，并破坏压缩/截断决策。
**修复:** 当提供商配置更改时添加缓存失效，或者使解析器无状态。
**文件:** 上下文限制解析器，压缩代码

### G5: disabled_hooks 模式与运行时契约不匹配
**问题:** 模式是严格的（拒绝未知的钩子名称），但运行时是宽松的（忽略未知）。契约不一致。
**修复:** 对齐——要么使两者都严格，要么两者都宽松。
**文件:** 钩子模式定义，运行时钩子加载

---

## 执行指令

1. 生成 UltraBrain 智能体并行修复这些问题——按文件接近度分组：
   - UB-1: C1 (hashline) + H4 (slash-command dedup)
   - UB-2: C2 + C3 + C4 (model/provider system) + G2
   - UB-3: C5 + C6 (config defaults) + G5
   - UB-4: H1 + H2 (runtime handlers + Atlas gate)
   - UB-5: H3 + G3 (dead code + skill references)
   - UB-6: G1 (package identity -- full audit)
   - UB-7: G4 (context-limit cache)

2. 每个 UB 智能体必须:
   - 首先编写或更新测试 (TDD)
   - 实现修复
   - 在受影响的测试文件上运行 bun test
   - 带有描述性消息的提交 (Commit)

3. 在所有 UB 智能体完成后，运行完整的 bun test 验证没有回退。

ulw
