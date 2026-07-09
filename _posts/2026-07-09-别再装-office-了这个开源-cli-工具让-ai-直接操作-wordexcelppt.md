---
title: "别再装 Office 了！这个开源 CLI 工具让 AI 直接操作 Word/Excel/PPT"
date: 2026-07-09
tags: [OfficeCLI, AI, 开源工具, CLI, 效率]
description: "OfficeCLI 是一个开源的命令行工具，让 AI 代理无需安装 Office 就能读写 Word/Excel/PPT 文件。本文带你上手这个今天 GitHub 日增 1900+ stars 的热门项目。"
---

## 痛点：AI 再聪明，也打不开 .docx

你是不是也遇到过这种情况——

让 Claude 或 GPT 帮你写一份周报，它输出了一大段 Markdown。你说"帮我生成 .docx"，它说"我无法直接生成 Office 文件"。然后你得手动复制粘贴、排版、调格式。

或者更糟：你有一堆 Excel 需要批量处理，但公司的电脑没装 Office（或者装的还是 2007 版），你只能装个 python-pptx、openpyxl、python-docx，写 50 行胶水代码，调试半天，就为了改个字体颜色。

**AI 时代，这个痛点尤其扎心。** AI 代理能写代码、能部署、能调 API，但到了 Office 文档面前，就变成了瞎子。

今天要介绍的项目——**OfficeCLI**，就是来填这个坑的。

## OfficeCLI 是什么？

一句话：**让你和 AI 代理在命令行里直接读写 Word、Excel、PowerPoint 文件，不需要装 Office。**

从底层来看，它不是简单套壳 python-docx，而是一个从零开始实现 Office 文件格式引擎的单二进制工具。**C# 写的，12.9k stars，今天日增 1900+，热度还在涨。**

安装方式非常简单：

```bash
# macOS / Linux
curl -fsSL https://raw.githubusercontent.com/iOfficeAI/OfficeCLI/main/install.sh | bash

# Windows PowerShell
irm https://raw.githubusercontent.com/iOfficeAI/OfficeCLI/main/install.ps1 | iex

# 或者 Homebrew
brew install officecli

# 或者 npm
npm install -g @officecli/officecli
```

装完之后，你的 AI 代理（Claude Code、Cursor、Copilot、Windsurf 等）自动就能调用 `officecli` 命令来操作文档。

## 上手：3 分钟做一个 PPT

来看最直观的场景——自动生成演示文稿。

过去你要写 Python：

```python
from pptx import Presentation
from pptx.util import Inches, Pt

prs = Presentation()
slide = prs.slides.add_slide(prs.slide_layouts[0])
title = slide.shapes.title
title.text = "Q4 Report"
# ... 再写 45 行调字体、位置、颜色 ...
prs.save('deck.pptx')
```

用 OfficeCLI：

```bash
# 创建新 PPT
officecli create deck.pptx

# 添加一页幻灯片
officecli add deck.pptx / --type slide --prop title="Q4 业绩报告" --prop background=1A1A2E

# 在幻灯片上加文字
officecli add deck.pptx '/slide[1]' --type shape \
  --prop text="营收增长 25%" --prop x=2cm --prop y=5cm \
  --prop font=Arial --prop size=24 --prop color=FFFFFF

# 实时预览（打开浏览器即可看到效果）
officecli watch deck.pptx
```

注意那个 `watch` 命令——它会启动一个本地预览服务（端口 26315），你每改一次文档，浏览器自动刷新。

**"渲染 → 查看 → 修改"的闭环形成了，AI 终于有了「眼睛」。**

OfficeCLI 的核心能力就是把 .docx / .xlsx / .pptx 渲染成 HTML 或 PNG。AI 代理可以先 `officecli view deck.pptx html`，看看效果对不对，不对再 `officecli set` 微调。

## 实测：批量处理 Excel 报表

我测试了另一个真实场景——处理公司的月度报表。

表结构大概是：A 列是姓名，B-K 列是各项考核分数，最后要算总分和排名。

过去我写 Python 脚本至少半小时起步（装库、查文档、调试），现在：

```bash
# 读取数据（输出 JSON，AI 可以直接处理）
officecli get report.xlsx '/sheet[1]' --json

# 在 M 列插入公式：总分
officecli set report.xlsx '/sheet[1]/col[M]/row[2:31]' \
  --prop formula="=SUM(B2:K2)"

# 在 N 列写排名公式
officecli set report.xlsx '/sheet[1]/col[N]/row[2:31]' \
  --prop formula="=RANK(M2,$M$2:$M$31)"

# 把 N 列整成整数
officecli set report.xlsx '/sheet[1]/col[N]' \
  --prop numberFormat="0"

# 另存为新文件
officecli save report.xlsx report-ranked.xlsx
```

整个流程用 AI 代理来写，**不到 5 分钟。**

而且 OfficeCLI 处理的是原始 Office XML，不需要装 Excel 引擎。这意味着你可以在 Linux 服务器上、在 Docker 容器里、在 CI/CD 流水线中，直接操作 Office 文件。

## 让 AI 代理自己用 OfficeCLI

这是最有意思的部分。OfficeCLI 官方提供一个 skill 文件：

```
curl -fsSL https://officecli.ai/SKILL.md
```

把这个文件喂给 AI 代理（Claude Code、Cursor 等），代理就知道怎么用 OfficeCLI。你可以直接说：

> "读取上周的销售数据.xlsx，生成一份月度总结 PPT，风格要简洁专业，深色背景。"

代理会自动做：
1. 用 `officecli get` 读取 Excel 数据
2. 分析数据趋势
3. 用 `officecli create` 和 `officecli add` 逐页生成 PPT
4. 用 `officecli watch` 让你预览和微调

**全程不用打开 Office 软件。**

目前支持的功能列表非常完整：

| 功能 | Word | Excel | PPT |
|------|:----:|:-----:|:---:|
| 读取文本/结构 | ✅ | ✅ | ✅ |
| 修改内容 | ✅ | ✅ | ✅ |
| 创建新文件 | ✅ | ✅ | ✅ |
| 公式/图表 | ✅ | ✅ | ✅ |
| 排版/样式 | ✅ | ✅ | ✅ |
| 图片嵌入 | ✅ | ✅ | ✅ |
| 预览为 HTML/PNG | ✅ | ✅ | ✅ |

而且对 Word 的支持非常深——RTL 排版、中文/阿拉伯语混排、修订跟踪、批注、水印、目录、LaTeX 公式转方程、Mermaid 图转原生形状……基本覆盖了 95% 的日常需求。

## 需要注意的问题

当然，它也不是银弹：

1. **格式保真度**：复杂文档（嵌套表格、自定义样式链）渲染可能跟 Word 桌面版有细微差异。如果客户要求"完全对齐"，还是得用 Office 本地打开调一下。
2. **性能**：超大文件（1000+ 页的 docx）处理较慢，毕竟底层是完整的 XML 解析引擎。
3. **图形图表**：对高级图表（Excel 数据透视表、PPT SmartArt）支持还在完善中。
4. **还比较新**：当前 v0.x，API 可能还在迭代，升级时注意 changelog。

## 我的看法

**OfficeCLI 解决了 AI 落地的最后一个「非技术壁垒」。**

很多企业场景里，输出的交付物就是 .docx 报告、.pptx 演示、.xlsx 数据表。这些格式看起来简单，处理起来极其恶心——二进制格式、各家实现不兼容、文档标准比一般 API 复杂得多。

OfficeCLI 的做法我认为是对的方向：**不给 AI 装 Office 的 GUI，而是给它一个专门设计的 CLI 接口。** Office GUI 是为人类交互设计的，对机器来说太多冗余。CLI 接口才是 AI 代理该用的方式。

如果你是开发者、数据分析师、或者任何需要跟 Office 文件打交道的技术从业者，**这个工具值得现在就装一个试试**——今天日增 1900 stars 不是没道理的。

---

**相关链接：**
- GitHub: [github.com/iOfficeAI/OfficeCLI](https://github.com/iOfficeAI/OfficeCLI)
- 官网: [officecli.ai](https://officecli.ai)
- 安装: `curl -fsSL https://raw.githubusercontent.com/iOfficeAI/OfficeCLI/main/install.sh | bash`
