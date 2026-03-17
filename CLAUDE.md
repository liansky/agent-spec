# CLAUDE.md

此文件为 Claude Code (claude.ai/code) 在此仓库中工作时提供指导。

## 项目概述

Agent Spec 是一个前端规范开发通用约束项目，用于定义和管理前端开发标准、规则和技能。

## 目录结构

- **`rules/`** - 开发规则和约束。在此处放置规则定义和约束规范。
- **`skills/`** - 开发技能和最佳实践。在此处添加技能定义和实现模式。
- **`.specstory/`** - SpecStory 扩展配置目录，用于保存 AI 聊天历史。

## SpecStory 目录

`.specstory/` 目录由 SpecStory 扩展管理：

- `.specstory/history/` - AI 编程会话的自动保存 markdown 文件
- `.specstory/.project.json` - 工作区的持久化项目标识
- `.specstory/ai_rules_backups/` - 派生 AI 规则的备份（`.cursor/rules/derived-cursor-rules.mdc` 或 `.github/copilot-instructions.md`）

## 重要说明

- `.specstory/history/` 已从版本控制中排除（参见 `.specstory/.gitignore`）
- 重点关注 `rules/` 和 `skills/` 目录的更改和添加，因为它们包含核心规范
