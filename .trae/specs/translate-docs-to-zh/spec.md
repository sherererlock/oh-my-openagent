# Translate Documentation to Chinese Spec

## Why
用户阅读英文文档吃力，需要将仓库内所有的英文文档（如 agents、skills、docs 等目录下的文档）翻译成中文，以提升可读性和学习效率。

## What Changes
- 识别所有主要的英文文档目录和文件（如 `docs/`, `.opencode/skills/`, `src/` 内部的说明文档, `AGENTS.md` 等根目录文档）。
- 在项目根目录创建 `zh/` 文件夹。
- 将这些英文文档翻译为中文，并保留原始目录结构放入 `zh/` 目录下（例如 `docs/manifesto.md` 翻译后存放至 `zh/docs/manifesto.md`）。
- 使用多个智能体并行处理翻译任务以提高效率。

## Impact
- Affected specs: 无
- Affected code: 新增 `zh/` 目录及其子目录和翻译后的 Markdown 文件，不影响原有代码和英文文档。

## ADDED Requirements
### Requirement: Documentation Translation
The system SHALL provide Chinese translations for all English documentation.

#### Scenario: Success case
- **WHEN** user views the `zh/` directory
- **THEN** they will find the translated Chinese documentation mirroring the original English directory structure.