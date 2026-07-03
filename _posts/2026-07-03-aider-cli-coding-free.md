---
layout: post
title: "Aider：一个命令让 Claude/GPT 直接在终端写代码（免费替代 Cursor）"
date: 2026-07-03 18:00:00 +0800
slug: aider-cli-coding-free
categories: [AI工具]
tags: [Aider, AI编程, CLI, 开源]
---

# Aider：一个命令让 Claude/GPT 直接在终端写代码（免费替代 Cursor）

看完 Cursor 涨价的糟心事，你可能在找退路。今天介绍一个你可能没试过的方案：**Aider**。

## 它是什么

Aider 是一个开源的命令行 AI 编程助手（Python 写的，`pip install` 就能用）。你不是在网页对话框里聊天，而是在终端里：

```bash
# 让你的项目文件夹变成 AI 工作区
cd your-project
aider --model claude-sonnet-4-20250514
```

然后 Aider 会：
- 读取你整个项目的文件结构
- 理解你当前的 git 状态
- 根据你的自然语言指令修改代码
- 自动 commit 每次修改

## 为什么值得一试

**真・看得见整个项目**：不像 Cursor 的上下文窗口有限，Aider 用 **map 文件** 维护项目结构的索引，大项目也不会迷路。

**自带 git 操作**：每次修改完自动 `git add` + `git commit`，可以 `git undo` 一步一步回退，不会改崩了找不到原始状态。

**模型自由**：支持 Claude、GPT、DeepSeek、本地 Ollama 模型，一个命令行切换。想省钱用本地模型，想质量用 Claude。

**价格**：本身免费开源（Apache 2.0），只需要付 API 费用。用 Claude Sonnet 的话，一天重度使用大概 $2-5，比 Cursor Pro $20/月 + 超量费便宜太多。

## 快速上手

```bash
# 安装
pip install aider-chat

# 设置 API Key（以 Claude 为例）
export ANTHROPIC_API_KEY=sk-ant-xxx

# 进入项目，开始干活
cd my-project
aider

# 然后像这样说话：
# "给这个 Flask app 加一个用户登录功能"
# "把这个函数拆成两个文件"
# "帮我写个 README"
```

Aider 会直接在终端里展示 diff，你可以 review 后再接受/拒绝。

## 缺点

- 纯终端，没有 IDE 内联补全。写代码时不能边写边补全。
- 不熟悉终端的开发者上手门槛高一点。
- 修改文件是整段替换，大段代码改动时 diff 看得有点累。

## 适用场景

- 需要 AI 帮你重构项目、写模块、加功能
- 想省钱避开 Cursor 的用量计费
- 本地有 GPU 想跑私有模型
- 想对改了什么有完全的控制和可见性

---

**一句话总结：** Cursor 涨价了确实恶心，但与其生气不如试试这个——Aider + Claude 的组合，写代码质量和 Cursor 同一级别，但花钱少得多，还不会背地里改你账单。
