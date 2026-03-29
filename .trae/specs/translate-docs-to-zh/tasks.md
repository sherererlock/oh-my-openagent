# Tasks
- [x] Task 1: 扫描并确定需要翻译的文档列表及创建目标目录结构
  - [x] SubTask 1.1: 搜索 `docs/`, `.opencode/skills/`, `.opencode/command/`, 根目录 `*.md` 以及 `src/**/AGENTS.md` 等文档
  - [x] SubTask 1.2: 过滤掉已经是中文的文档（如 `README.zh-cn.md`）
  - [x] SubTask 1.3: 根据搜索结果在 `zh/` 下创建对应的目录结构
- [x] Task 2: 翻译根目录和 `.opencode/` 目录文档
  - [x] SubTask 2.1: 使用翻译智能体翻译根目录下的英文文档（如 `AGENTS.md`, `CONTRIBUTING.md`, `FIX-BLOCKS.md` 等）至 `zh/`
  - [x] SubTask 2.2: 使用翻译智能体翻译 `.opencode/` 目录下的相关文档（包括 `skills/` 和 `command/` 下的 `.md`）至 `zh/.opencode/`
- [x] Task 3: 翻译 `docs/` 目录文档
  - [x] SubTask 3.1: 使用翻译智能体翻译 `docs/` 及其子目录（如 `examples/`, `guide/`, `reference/`）下的 `.md` 文件至 `zh/docs/`
- [x] Task 4: 翻译 `src/` 内部的文档
  - [x] SubTask 4.1: 使用翻译智能体翻译 `src/` 及其子目录下的 `AGENTS.md` 和其他 `.md` 说明文档至 `zh/src/`

# Task Dependencies
- Task 2 depends on Task 1
- Task 3 depends on Task 1
- Task 4 depends on Task 1