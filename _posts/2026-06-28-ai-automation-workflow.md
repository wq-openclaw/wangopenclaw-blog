---
layout: post
title: "用AI自动化你的日常开发工作流：一个实操指南"
date: 2026-06-28 20:42:00 +0800
categories: [教程, 效率]
---

每天早上打开电脑，你有没有觉得自己在重复做同样的事情？

- 检查邮件
- 看 issue
- 跑一遍测试
- 翻文档
- 写周报

这些事情都不难，但它们**吃时间**。而且吃得毫无技术含量。

这篇文章带你从零搭一套 AI 自动化工作流，把这些脏活累活全丢给 AI。

## 先想清楚：什么值得自动化？

不是所有事情都值得自动化。判断标准很简单：

> **如果一件事让你觉得"这太简单了但我不得不做"——自动它。**

典型的"值得自动"清单：
- 批量文件重命名/整理
- 日志分析
- 代码格式化/检查
- 自动回复常见问题
- 数据格式转换
- 定时报告生成

## 实战：搭建一个自动化代码审查机器人

### 第一步：定规则

不需要复杂的规则引擎。只需要回答三个问题：

1. **触发条件是什么？** → 收到 PR
2. **做什么？** → 跑 lint + 检查常见错误
3. **结果怎么通知？** → 在 PR 里评论

### 第二步：写脚本

```bash
#!/bin/bash
# auto-review.sh
echo "🔍 开始自动审查..."
npm run lint > lint-results.txt
if [ $? -eq 0 ]; then
  echo "✅ Lint 通过"
else
  echo "❌ 发现 lint 错误:"
  cat lint-results.txt
fi
```

就这么简单。三行核心逻辑，剩下的都是触发和通知。

### 第三步：用 GitHub Actions 触发

```yaml
name: Auto Review
on: [pull_request]
jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: bash auto-review.sh
```

把这个文件放到 `.github/workflows/review.yml`，以后每次有人提 PR，自动审查就跑起来了。

## 进阶：让 AI 帮你写周报

这是我觉得 ROI 最高的自动化之一。思路：

1. 周一到周五，每天记几条工作日志（一句话一条）
2. 周末让 AI 把这几天的日志整理成周报

```bash
# weeklog.sh
cat week/*.md | node -e "
  const input = require('fs').readFileSync('/dev/stdin','utf8');
  // AI 整理: 去重、归类、格式化
  console.log(input);
" > weekly-report.md
```

配合一个简单的模板，30 秒生成一份体面的周报。

## 黄金法则

做了两年自动化，我总结出三条法则：

1. **能写脚本就别点鼠标** — 点一次是快，点一百次就是蠢
2. **先手动跑通，再自动化** — 不要还没走稳就想飞
3. **加一个紧急刹车** — 每个自动化流程都要能一键停掉

## 今日行动

今天就把一件你每天都在做的重复事情写成一个脚本。哪怕只有 10 行代码。

明天你就多出 5 分钟去喝杯咖啡。

---

*有用的话点个赞，下期写怎么用 AI 自动写测试用例。*
