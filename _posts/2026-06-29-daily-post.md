---
layout: post
title: "Daily Post"
date: 2026-06-30 00:12:22 +0800
categories: [教程]
---
# Cursor 1.0 正式发布，但谷歌 Gemini CLI 才是真正的降维打击

> 2025年6月，Cursor 1.0 带着 BugBot、Background Agent、Memories 等一堆新功能来了。但如果你还在纠结要不要续费 $20/月的 Cursor Pro，我建议你先看看谷歌刚开源的 Gemini CLI——**免费、开源、1000次/天，还能直接操作你的终端。**

---

## 一、Cursor 1.0：贵，但确实好用

6月4日，Cursor 1.0 正式发布。作为已经靠 AI 编程工具赚到 5 亿美元 ARR、估值 99 亿美元的超级独角兽，Cursor 这次更新没有让人失望：

### 1. BugBot：自动代码审查

这是我最期待的功能。BugBot 会自动审查你的 GitHub PR，发现潜在 bug 后直接在该 PR 下评论，你还能一键跳回 Cursor 修复。

**实际体验：** 我把它接到一个写了 3 个月的 Python 项目上，它居然发现了一个我手动 review 三次都没看出的竞态条件。虽然偶尔也会误报（大概 20% 的噪音），但省下的时间绝对值回票价。

### 2. Background Agent：后台智能体全面开放

之前内测的功能现在所有人都能用了。你可以给 Cursor 一个任务（比如"重构这个模块"），然后关掉电脑去喝咖啡，回来它已经改好了。

**但有个坑：** 这个功能只在非隐私模式下可用。如果你在公司用 Cursor 处理敏感代码，大概率开不了隐私模式——然后 Background Agent 就用不了。这设计挺矛盾的。

### 3. Memories：上下文记忆

Cursor 现在能记住你项目里的约定和偏好。比如我告诉它"我们项目里所有 API 返回都用 Result 类型包装"，之后它生成的代码就自动遵循这个规则。

**实测效果：** 比手动写 `.cursorrules` 文件方便，但记忆有时候会被"污染"——它会把一次性的临时需求也记进去，导致后续生成奇怪的代码。

### 4. MCP 一键安装

MCP（Model Context Protocol）现在可以一键安装了。这意味着你可以把数据库、API 文档、设计稿直接接到 Cursor 里，让它基于真实上下文写代码。

**Cursor 1.0 的问题也很明显：**

- **贵**：Pro 版 $20/月，Team 版 $40/人/月。对于个人开发者来说，一年就是 240 美元。
- **闭源**：你所有的代码习惯、项目结构、甚至 BugBot 的审查记录，都存在 Cursor 的服务器上。
- **内存杀手**：大项目下经常吃掉 4-8GB 内存，低配笔记本直接卡死。

---

## 二、Gemini CLI：谷歌的"免费大招"

6月25日，谷歌突然扔了个炸弹——**Gemini CLI 开源了**（Apache 2.0 协议）。

这不是一个 IDE 插件，而是一个可以直接装在你终端里的 AI Agent。听起来平平无奇？看看它的实际能力：

### 1. 免费额度 generous 到离谱

```bash
# 用个人 Google 账号登录，免费额度：
# - 60 请求/分钟
# - 1000 请求/天
# - 1M token 上下文窗口
# - 包含 Gemini 2.5 Pro

gemini
```

对比 Cursor 的 $20/月（而且用完 fast requests 还要额外买），Gemini CLI 的免费额度足够一个全职开发者用一整天。

### 2. 直接操作终端，不是聊天框

Cursor 是"在编辑器里聊天"，Gemini CLI 是"在终端里执行"。区别很大：

```bash
# 让 Gemini 分析整个代码库架构
gemini -p "Explain the architecture of this codebase"

# 让它直接运行测试并修复失败的
gemini -p "Run tests and fix any failures"

# 基于 Google Search 实时信息回答问题
gemini -p "What's the latest stable version of React and what changed?"

# 非交互式脚本化使用（CI/CD 场景）
gemini -p "Review this PR for security issues" --output-format json
```

**关键区别：** Gemini CLI 可以直接执行 shell 命令。你让它"部署这个服务"，它真的会运行 `docker build` 和 `kubectl apply`。Cursor 的 Agent 模式也能执行命令，但 Gemini CLI 是原生的终端工具，更轻量、更快。

### 3. 多模态能力：图片/PDF 直接转代码

```bash
# 丢一张设计稿截图，让它生成前端代码
gemini -p "Convert this design mockup to React components" --image ./mockup.png

# 丢一个 PDF 需求文档，让它直接实现
gemini -p "Implement the features described in this PDF" --file ./requirements.pdf
```

这个功能 Cursor 也有（设计图转代码），但 Cursor 是在编辑器里操作，Gemini CLI 是在终端里一行命令搞定。

### 4. MCP 支持 + 开源生态

Gemini CLI 同样支持 MCP，而且因为它是开源的，社区已经在疯狂造轮子：

```bash
# 接入 Imagen 生成图片
# 接入 Veo 生成视频
# 接入你的内部 API
# 任何 MCP server 都能接
```

### 5. Checkpoint 功能：随时保存和恢复会话

```bash
# 保存当前会话
gemini /checkpoint save "before-refactor"

# 搞砸了，恢复
gemini /checkpoint load "before-refactor"
```

这比 Cursor 的聊天历史好用多了——你可以给 checkpoint 起名字，像 git commit 一样管理对话状态。

---

## 三、实际对比：我用同一个需求测试了两者

**需求：** "分析这个 Python 项目的依赖漏洞，并给出修复方案"

### Cursor 1.0 的做法：

1. 打开项目
2. 在聊天框输入需求
3. Cursor 分析 `requirements.txt`
4. 给出漏洞列表和修复建议
5. 我手动点击"Apply"应用修改

**耗时：** 3 分钟
**体验：** 流畅，但需要鼠标操作

### Gemini CLI 的做法：

```bash
cd my-project
gemini -p "Scan for dependency vulnerabilities in requirements.txt and generate a patched version"
```

**耗时：** 45 秒
**体验：** 终端直接输出结果，包括修复后的 `requirements.txt` 内容

---

## 四、我的结论：两者不是替代关系，但 Gemini CLI 改变了游戏规则

| 维度 | Cursor 1.0 | Gemini CLI |
|:---|:---|:---|
| **价格** | $20-40/月 | 免费（1000次/天） |
| **定位** | AI 原生 IDE | 终端 AI Agent |
| **代码编辑** | ✅ 可视化 diff，一键应用 | ⚠️ 文本输出，需手动复制 |
| **终端操作** | ⚠️ 需要 Agent 模式 | ✅ 原生支持 |
| **多模态** | ✅ 设计图转代码 | ✅ 图片/PDF 转代码 |
| **开源** | ❌ 闭源 | ✅ Apache 2.0 |
| **内存占用** | 高（4-8GB） | 低（Node.js 进程） |
| **适用场景** | 大型项目开发 | 快速脚本、运维、CI/CD |

**我的建议：**

1. **如果你主要写代码、做大型项目开发**——Cursor 1.0 仍然是最好的选择。可视化 diff、代码补全、IDE 集成这些体验是 Gemini CLI 给不了的。

2. **如果你经常写脚本、做运维、需要快速查询和处理**——Gemini CLI 是神器。免费、快、不占用内存，还能直接执行命令。

3. **如果你预算有限**——先用 Gemini CLI。1000 次/天的额度，对个人开发者来说基本够用。等收入上来了再考虑 Cursor。

4. **最佳实践：两者一起用**——日常开发用 Cursor，快速任务和脚本用 Gemini CLI。它们甚至可以互补：Gemini CLI 生成代码框架，Cursor 做精细化编辑。

---

## 五、Gemini CLI 快速上手

```bash
# 安装（需要 Node.js 20+）
npm install -g @google/gemini-cli

# 登录（用 Google 账号）
gemini

# 基础使用
gemini -p "Your prompt here"

# 指定模型
gemini -m gemini-2.5-pro -p "Complex task"

# 包含其他目录的上下文
gemini --include-directories ../lib,../docs -p "Refactor the API"

# 非交互式/脚本化
gemini -p "Deploy to staging" --output-format json
```

**项目级配置：** 在项目根目录创建 `GEMINI.md`，写入项目背景和规范，Gemini CLI 会自动读取：

```markdown
# GEMINI.md

## 项目背景
这是一个基于 FastAPI 的微服务，使用 PostgreSQL 和 Redis。

## 代码规范
- 所有函数必须有类型注解
- 异步函数统一用 async/await
- 错误处理使用自定义的 AppException
```

---

## 写在最后

Cursor 1.0 的发布证明了 AI 编程工具正在从"代码补全"进化到"自主代理"。但谷歌用 Gemini CLI 告诉我们另一件事：**这个赛道不一定非要花 20 美元/月才能玩。**

开源、免费、1000 次/天的额度——这对个人开发者、学生、初创团队来说，是一个巨大的福音。

当然，Gemini CLI 目前还不够成熟。没有可视化 diff、没有 IDE 集成、偶尔会出现"幻觉"执行危险命令。但它是开源的，社区正在快速迭代。

**2025 年的 AI 编程工具市场，已经从"Cursor 一家独大"变成了"百花齐放"。作为开发者，我们只需要做一件事：都试试，然后选最适合自己的。**

---

*本文基于 Cursor 1.0（2025-06-04）和 Gemini CLI（2025-06-25）实测。工具版本更新快，建议以官方文档为准。*