---
layout: post
title: "3 分钟给 VS Code 配一个 AI 侧边栏（免费）"
date: 2026-07-01 18:00:00 +0800
slug: vscode-ai-sidebar-free
categories: [教程]
tags: [VS Code, AI, 效率]
---

# 3 分钟给 VS Code 配一个 AI 侧边栏（免费）

不想用 Cursor，但想要编辑器内 AI 对话？VS Code 的 **Continue** 插件就能做到——而且完全免费。

## 安装

```bash
# 在 VS Code 扩展市场搜索 "Continue"
# 或者按 Ctrl+P 然后粘贴：
ext install continue.continue
```

## 配置 API Key

Continue 自带支持多种模型。最省钱的方案：

```bash
# 用 Anthropic API（按量付费，约 $0.01/次对话）
export ANTHROPIC_API_KEY=sk-ant-xxx

# 或者用 Ollama 本地模型（完全免费）
ollama pull codellama
```

在 VS Code 里按 `Ctrl+Shift+P` → "Continue: Open Config" → 添加模型配置。

## 能做什么

- 选中代码 → 按 `Ctrl+L` → 在侧边栏问问题
- 右键 → "Edit with AI" → 直接修改选中代码
- 自动补全：输入时给出建议（和 Copilot 类似的效果）

不需要换编辑器，不需要付费。VS Code + Continue = 你熟悉的编辑器 + AI 能力。想省事就装一个，比在网页和编辑器之间来回切快多了。
